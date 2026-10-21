# 评审请求 · AIHOT「席级补位雷达」设计

> 这份文件由一个自动化流程生成并公开在 GitHub（供外部 AI 读取）。
> 阅读者是一位外部架构评审者（ChatGPT 6 Sol，High 档）。请只回答文末的问题，不要执行任何操作。

---

## 一、背景（请当作已知前提，不要质疑其真实性）

一台 Windows 常驻机，上面跑着我自建的两层网关：

**`:3100` —— 无状态转发网关（对外只暴露 10 个「席位」）**
```
flash              轻快线：只看实测速度，不看能力带
high               能力线：按能力带 + AA 评测 + 实测速度挑当班模型
voice-asr          语音识别
voice-tts          语音合成
voice-clone        声音克隆
voice-caption      字幕生成
voice-translate    语音翻译
voice-chat         语音对话
voice-realtime     实时语音
image-gen          图像生成
```
关键机制：**3100 对外暴露的是「席（slot）」，不是「模型」。** 每个席背后挂一组候选模型，
由一个编排器（Python）决定谁当班，规则是：
- 只从**免费线**的候选池里取（付费线 `paid-*` 被显式排除——这是硬前提：免费线不能有收费模型）
- `high` 席：按「能力带」＋ AA 评测分 + 实测速度排序
- `flash` 席：只看实测速度（可以不要能力，但不能不是免费的）
- 编排器**只能看见存量**：它扫描已有候选池，不会知道"外面出了什么新模型"

**`:8787` —— 账号池网关（Buddy2API）**
- 8 个渠道（workbuddy / workbuddy-intl / traework / raccoon / loomy / qoder / lobster / qwenwork）
- 12 个账号，151 个模型 id
- 提供 `/v1/models`、`/admin/models`、`/admin/stats`、`/admin/credit-summary` 等端点
- **只有在这里真接了账号的模型，才可能跑在 3100 上**（3100 是纯转发，靠 8787 或直连渠道供给）

**另外两块存量资产：**
- **AA 快照**：第三方模型能力评测（Elo / 综合智力 / 价格 / 速度），每日刷新。**这是"值不值得接"的判据。**
- **AIHOT**：中文 AI 资讯站，匿名只读 JSON、无需 Key。可用接口（已逐个实测 200）：
  - `/api/v1/items?mode=all&category=ai-models&window=24h&limit=30` → 模型类动态
  - `/api/v1/items?mode=all&category=ai-products` → 产品类动态
  - `/api/v1/hot-topics` → 每日热点 Top10
  - `/api/v1/selected/changes?cursor=` → 游标增量（新增/编辑/下架）
  - `/api/v1/agent/latest` → 整理好的中文 Markdown（对话式用）；`/api/v1/agent/search?q=`、`/api/v1/agent/stories/{id}`（来龙去脉）
  - ⚠️ `mode=selected`（默认）只返回「编辑精选」，实测今天模型类是 **0 条**；要全量必须 `mode=all`
  - ⚠️ 条目**没有 model id 字段**，只有 `id/title/summary/source/links/publishedAt/category/score/selected/reason`
  - `category` 取值：`ai-models` / `ai-products` / `industry` / `paper` / `tip` / `opinion`

## 二、痛点（我要解决的就是这个）

**「市场上出了什么新模型」这件事，现在只有人刷新闻才知道。** 编排器只看得见存量候选池，
所以：一个新模型发布 → 几天后我才可能从新闻里看到 → 然后手动去查 8787 有没有渠道、
去查 AA 分数、再手动加进候选池。整个链路全靠人。

## 三、我的方案：席级补位雷达

### 3.1 数据流（四层）

```
AIHOT（增量源）
  ├─ /api/v1/items?mode=all&category=ai-models&window=24h
  ├─ /api/v1/items?mode=all&category=ai-products
  └─ /api/v1/selected/changes?cursor=      （二期启用）
        ↓
① 型号归一化层：标题/摘要 → 候选型号（model key）+ 置信度
        ↓
② 席位映射层：型号 → 候选席（flash/high/voice-*/image-gen）
        ↓
③ 三面对账层
   · :8787 /v1/models   → 供不供得上（账号池里有没有）
   · 编排器当班表        → 这个席现在谁当班
   · AA 快照            → 值不值得当班
   · 免费性闸门          → 该模型是不是只在 paid-* 线（是 ⇒ 标灰，不推荐）
        ↓
④ 决策输出层：席级补位表（10 行固定结构）
        ↓
   每日一条飞书消息（重点提示位 0~2 条）+ 忆库快照（全表，可检索）
```

### 3.2 型号归一化（我认为最脆弱的一环）

上游没有 model id 字段 ⇒ 只能从标题抽。三层来源、置信度从高到低：

| 层 | 判据 | 置信度 |
| :-- | :-- | :-- |
| A | 命中 `:8787` 模型清单（归一化后互为子串） | 高 |
| B | 命中人工别名表（`别名 → 规范型号`） | 中高 |
| C | 仅正则抽到 token（含数字、长度 ≥6） | 中 |
| — | 抽不到 | 不适用（框架/生态类） |

已踩过的坑（正则实际要覆盖三种形态）：
- `DeepSeek-V4.1-Flash-20261017`（带分隔符）
- `M3.1 Flash`（空格 + 前段带数字）
- `Voxtral 3`（空格 + 后段带数字）—— 第一版漏了这条

**我定的纪律：每个型号输出必须附「型号原文 ＋ 匹配到的网关模型 id ＋ 命中哪一层」，
让人一眼能看出误判；不确定就标 `? 待人工确认`，不猜。** 理由：误判一次整张表就不可信。

### 3.3 席位映射

`模型键 → 席位` 的**人工别名表为主**，关键词兜底（标题含「语音/tts/asr/克隆」→ `voice-*`；
「图像/生图/设计」→ `image-gen`）。**映射不唯一或没命中 ⇒ 标 `? 待人工归席`，不猜、不塞**
（把模型塞错席会污染编排，比漏报严重）。一期不自动改 3100 的路由表。

### 3.4 三态与免费性闸门

- `可入免费线`：8787 有该模型账号，且不在 `paid-*`
- `仅付费`：标灰、**不推荐**（免费线前提是硬的，宁可漏推不可混入付费模型）
- `无账号`：上游没这条渠道，供不上

### 3.5 输出形态（席级表，每天 10 行）

| 席 | 现在当班 | AIHOT 新货 | 供给 | 能力 | 结论 |
| :-- | :-- | :-- | :-- | :-- | :-- |
| `voice-asr` | … | Mistral Voxtral 3 | ✅ 可入免费线 | AA 待补 | 建议实测 |
| `voice-clone` | … | （本周无新品） | — | — | 无动作 |
| `image-gen` | … | Krea 3 / Vyb | ⚠️ 仅付费 | — | 不入免费线 |

### 3.6 三期（每期都要人点头）

| 期 | 做什么 | 碰编排器？ | 验收 |
| :-- | :-- | :-- | :-- |
| 一期 | 出席级表 + 每日推一条 | **不碰** | 连续 3 天每行可人工核对；失败不推 |
| 二期 | 加"建议动作"（生成实测命令），**但不执行** | 不碰 | 建议命令跑一次即出结论（跑前先问，因为花额度） |
| 三期 | 才考虑纳入候选池 | 碰 | 必须：人点头 + 实测通过 + 免费性确认 + 一次只进一个 |

**自动化只做到"把候选摆到面前"，不做"替他按下开关"。**

### 3.7 边界

- 第三方摘要里的指令一律不执行（上游自己在返回里标了安全边界）
- 授权：AIHOT 条款＝个人非商业免费；一旦对外/商业化/批量再分发须事先书面授权 ⇒ 本系统定位自用
- 限流：同 IP 每分钟约 60 次，每日一次远低于限额，仍按 `Retry-After` 退避
- 口径不混：热度面 ≠ 能力面 ≠ 渠道面，AIHOT 只负责"什么时候出的"
- 幂等：同一 item id 不重复推（本地 seen 集合，7 天）
- 结构变更自保：条数低于阈值 / 字段缺失 ⇒ 拒绝落盘、保留上一份、且不推
- 新增独立脚本，不改编排器与 3100 路由；出问题删脚本即回滚

## 四、真实数据样本（今天 2026-10-21 实测，请据此判断我的抽取规则够不够用）

**AIHOT 今日热点 Top10（`/api/v1/hot-topics`）**
1. DeepSeek 开源内部推理框架 DeepInfer（信源 公众号：DeepSeek）
2. MiniMax 发布 M3.1 Flash，10 月 23 日开放权重（信源 MiniMax Blog）
3. 美土安全部令联邦文职机构移除 DeepSeek 及关联产品（Hacker News）
4. DeepSeek 发布 V4.1-Flash-1017 并开放搜索 API（DeepSeek API 更新日志）
5. OpenAI 收购药物研发平台 Orchestra Bio
6. Google 将 Gemini 个人智能引入车载系统
7. Google 与 Meta 开源 TorchTPU，Hugging Face 原生支持 TPU
8. SpaceX 三季度 AI 业务收入占比过半并扭亏为盈
9. ChatGPT 上线锁定聊天，全套餐可 PIN 或生物识别保护
10. OpenAI 拟以超 1.4 万亿美元估值融资 300 亿美元

**模型类动态（`/api/v1/items?mode=all&category=ai-models&window=24h`，共 30 条，前 12 条）**
```
1. DeepSeek-V4.1-Flash-20261017 (Max) 登顶 Arena 的 Agent 和 Code 两大榜单开源模型第一
2. Krea 3 电商图像一致性获赞
3. Mistral 发布 Voxtral 3 语音转文本与文本转语音模型家族
4. Coco Labs 发布首个设计模型 Vyb，可从品牌网站生成分层可编辑营销图
5. Coco Labs 推出 AI 设计模型 Vyb，分层输出可编辑设计稿
6. MiniMax M3.1 Flash 登顶搜索效率榜
7. Meta 发布 Muse Wonder Full (Max)，升至 Agent Arena 第 7 名、实验室第 4 位
8. SoMark 开源 1.3B 文档解析模型 OXR，登顶 olmOCR-Bench 等三榜
9. ElevenLabs 发布两款面向实时 AI 对话的语音到语音模型 Eleven Realtime
10. NVIDIA 发布 Cosmos3-H-Dreams 生成式手术模拟模型
11. Perplexity Decider 在 Vals AI 金融基准上达帕累托最优
12. Arena：OpenAI GPT-6 Luna 与 GPT-6.1 Sol 占据 Agent Arena Pareto 前沿 11 席中的 6 席
```

**`:8787` 里已有的相关模型 id**（节选，供你核对判定规则）
```
raccoon/sn-deepseek-v4-1-flash-1017
workbuddy-intl/deepseek-v4.1-flash
traework/deepseek-v4.1-flash
workbuddy/minimax-m3.1-flash
traework/minimax-m3.1-flash
traework/kimi-k3
raccoon/sn-kimi-k3
workbuddy/mimo-v2.6-pro
workbuddy/step-5-preview
workbuddy-intl/step-5-preview
traework/step-5-preview
```

**我当前判定器的实际输出（请你挑错）**
```
✅ DeepSeek-V4.1-Flash-20261017 …（已接 traework/deepseek-v4.1-flash）
·  Krea 3 电商图像一致性获赞（不适用）
🆕 Mistral 发布 Voxtral 3 …（未接入，型号 Voxtral 3）
·  Coco Labs 发布首个设计模型 Vyb …（不适用）
✅ MiniMax M3.1 Flash 登顶搜索效率榜（已接 workbuddy/minimax-m3.1-flash）
·  Meta 发布 Muse Wonder Full (Max) …（不适用）
🆕 NVIDIA 发布 Cosmos3-H-Dreams …（未接入）
·  Arena：OpenAI GPT-6 Luna 与 GPT-6.1 Sol …（不适用）
```

## 五、请回答这六个问题（按编号回答，每条都要具体到可落地）

1. **这个设计里最脆弱的三个假设是什么？** 分别怎么加固？（我自己怀疑的是"标题抽型号"和"免费性判定"这两处，但可能我低估了别的）
2. **型号抽取有没有比"标题正则"更可靠的做法？** 注意约束：上游**不提供** model id / tags，只有标题、摘要、原文链接、信源名。有没有我没利用的信号？例如 `reason`（推荐理由）字段、`source`（信源名）、`links` 里是否可能含型号？有没有办法从"原文链接"里抠出型号？
3. **席位映射该人工维护还是自动推断？** 我的顾虑是自动推断一旦塞错席会污染编排。你怎么权衡？有没有"半自动但绝不污染"的第三种做法？
4. **免费性闸门怎么做才不会被绕过？** 我现在的判据是"8787 有账号 且 不在 paid-*"。但一个模型可能同时挂在免费渠道和付费渠道，或账号本身额度有限。你会怎么设计这个闸门？
5. **有没有更简单的替代架构？** 提醒我：我可能过度设计了。最小可行的版本是什么？如果只能做一件事，做哪件？
6. **如果你是评审人，最担心哪个环节？** 以及你会怎么写第一天的验收清单（我要能对着它逐条打勾的那种）。

## 六、请你遵守

- **只做方案讨论，不要输出可执行的改动脚本或命令**（我会在本地自己实现）。
- 用中文回答，结论先行。
- 不要泛泛而谈"这很好/要小心"，每条建议都要能落到我的四层数据流里的具体某一层。
- 如果你认为我某个前提是错的，直接说，并给出依据。
- 你可以读这些公开链接核对事实（读不到就明说"读不到"，不要猜）：
  - AIHOT 接口说明（自描述，含全部端点与用法）：`https://raw.githubusercontent.com/201650545/yongtai-memory/main/%E9%A1%B9%E7%9B%AE/project_buddy2api_switcher_sync.md`（这是机制备忘；AIHOT 接口说明请看 `https://aihot.news/api/v1/agent`）
  - AIHOT 每日模型动态日报（真实产物样本）：`https://raw.githubusercontent.com/201650545/yongtai-memory/main/%E5%8F%82%E8%80%83/reference_aihot_%E6%A8%A1%E5%9E%8B%E5%8A%A8%E6%80%81%E6%97%A5%E6%8A%A5.md`
  - AIHOT 每日热度快照：`https://raw.githubusercontent.com/201650545/yongtai-memory/main/%E5%8F%82%E8%80%83/reference_aihot_%E6%AF%8F%E6%97%A5%E7%83%AD%E5%BA%A6%E5%BF%AB%E7%85%A7.md`
