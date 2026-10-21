# 第二轮 · 评审追问：AIHOT「席级补位雷达」

> 承接第一轮评审（`reviews/aihot-slot-radar-review-prompt.md` 是背景与方案原文）。
> 本文件是**我对你第一轮意见的逐条核实结果 ＋ 我已做的修复 ＋ 新的六个追问**。
> 请只回答第五节的六个追问。仍然是**只做方案讨论，不要输出可执行脚本**。

---

## 一、你的第一轮判断核实结果：你抓到了一个真 bug，而且是我的

你指出「DeepSeek 和 MiniMax 两条记录都标注了网关模型 `qwenwork/flash`，但文件列出的 8787 相关模型 ID 中没有这个 ID」。
**你是对的，而且这不是材料笔误，是我的实现 bug。** 已核实并已修。

### 根因（两条，互相叠加）

1. **短 tail 参与子串匹配**：我的判定条件是「网关 id 的 tail 归一化后 ⊂ 候选型号归一化」。
   `qwenwork/flash` 的 tail 归一化 = `flash`，而 `flash` ⊂ `deepseekv41flash20261017`、
   也 ⊂ `m31flash` ⇒ **DeepSeek V4.1-Flash 和 MiniMax M3.1 Flash 双双被误判成"已接入 qwenwork/flash"**。
   实际上 `qwenwork/flash` 是 qwenwork 渠道的一个**通用 flash 模型**，跟 DeepSeek/MiniMax 毫无关系，
   它们只是共享了 "flash" 这个词。
2. **遍历用 set ⇒ 结果不可复现**：网关模型清单我存成了 `set`，遍历顺序不固定 ⇒
   **同一份输入，不同时间跑可能命中不同的 id**（实测：干跑时命中 `traework/deepseek-v4.1-flash`，
   真跑时命中 `qwenwork/flash`）。这类"每次不一样"的输出比稳定误判更糟，因为它无法被复核。

### 我已做的修复（① 型号归一化层）

- **只有归一化后长度 ≥6 且含数字的 tail 才有资格参与匹配**（`flash`、`glm` 这类短词直接出局）；
- **遍历改为有序列表**（`sorted(gw_ids)`）⇒ 结果可复现；
- **引入中间态「家族级」**：区分"型号一致"与"只是同一个家族"：
  - `已接入`＝归一化后**完全相等**，或网关 tail 是候选的**后缀**且只差厂商前缀
    （`m31flash` ↔ `minimax-m3.1-flash`），**且版本/日期标记一致**；
  - `家族级`＝只对上家族（版本号不同、变体不同）⇒ **不下"已接入"结论**，只标"同家族，变体待核"；
  - `未接入` / `不适用` / `未判定` 保持原义。
- 修复后实测（同一份输入）：

| 标题 | 修复前 | 修复后 |
| :-- | :-- | :-- |
| `DeepSeek-V4.1-Flash-20261017 (Max) 登顶 Arena` | ✅ 已接入 `qwenwork/flash` | 🔶 **同家族，变体待核**（`deepseek-v4.1-flash`） |
| `MiniMax M3.1 Flash 登顶搜索效率榜` | ✅ 已接入 `qwenwork/flash` | ✅ **已接入（型号级）** `minimax-m3.1-flash` |
| `Mistral 发布 Voxtral 3 …` | 🆕 未接入 | 🆕 未接入（不变） |
| `NVIDIA 发布 Cosmos3-H-Dreams …` | 🆕 未接入 | 🆕 未接入（不变） |
| `DeepSeek 开源内部推理框架 DeepInfer` | · 不适用 | · 不适用（不变） |

**请你评估：这个修法够不够？特别是两点——
(a) 我用"版本/日期标记不一致 ⇒ 降级为家族级"来治版本混淆。
今天这条 `DeepSeek-V4.1-Flash-20261017` 实际对应网关里的 `raccoon/sn-deepseek-v4-1-flash-1017`
（`1017` 与 `20261017` 是同一个发布日的两种写法），所以我的规则**保守地把一条真·已接入判成了家族级**。
这种"宁可漏判不可错判"的取舍你认可吗？还是你建议用别的方式对齐版本标记？
(b) 我把"已接入"收紧成"型号级"之后，语义就不再等于"这个型号的 API 一定能调"（见第三节的字段）。
你觉得这一层的输出该怎么措辞，才不会让人误读？

---

## 二、你其余意见的处置（我先说我的态度，不同意的请直接驳）

| 你的意见 | 我的态度 | 理由 |
| :-- | :-- | :-- |
| `category=ai-models` ≠ 可接入新品，要分实体类型和事件类型 | **接受**，进 ① 层 | 今天的样本里 Vyb 两条、Voxtral 多条确实重复 |
| 匹配要分级（精确/别名/家族/模糊/未识别） | **接受并已落地一半**（我加了"家族级"，还没做"已审核别名"与"模糊候选"） | 别名表是二期的事，因为要人工核 |
| "待人工归席"应是**阻断状态**，不是低优先级候选 | **接受**，这是硬规则 | 塞错席会污染编排 |
| 免费性要从三态扩到五态，**失败关闭** | **接受**，但需要你用第三节的真实字段给出判据 | 见第五节追问 2 |
| ① 层要区分"模型身份"与"事件类型"（首发/开放权重/上线 API/评测） | **接受**，但我担心在只有 title/summary 的情况下可自动判定的程度有限 | 见第五节追问 3 |
| 原文链接里抠型号"值得利用" | **保留意见**（我倾向只对官方发布信源做） | 见第五节追问 4 |
| **只做①层最简版本**（每天一份可审计的"新型号候选清单"） | **接受，这会改我的分期** | 见第五节追问 5 |
| 一期成功指标是"每条建议都能解释为什么"，不是"每天发现多少新品" | **完全接受** | 这句我会写进验收标准 |
| ④ 层必须区分"本次成功"与"沿用旧快照" | **接受** | 现在旧快照和当日结果长得一样，是个真缺陷 |

---

## 三、给你的真实字段（免得你只能按原则空谈）

### 1) `:8787` 的 `accounts` 表实际列（共 26 列）

```
id, name, uid, nickname, phone, account_type, access_token, refresh_token,
expires_at, refresh_expires_at, domain, enterprise_id, session_state, status,
weight, priority, credit_limit, credit_baseline, last_used_at, total_requests,
total_tokens, total_credits, created_at, updated_at, provider, extra
```

当前 12 个账号按 `provider` 分布：
```
workbuddy 6 ｜ traework 1 ｜ raccoon 1 ｜ loomy 1 ｜ qoder 1 ｜ lobster 1 ｜ qwenwork 1
```
（`workbuddy-intl` 在账号表里没有独立账号；它在 `/admin/credit-summary` 的渠道清单里，状态 `unsupported=true`、
`unit=unknown` ⇒ 网关自己也说"这个渠道没有额度概念"。）

### 2) 渠道维度（`/admin/credit-summary` 的 `channels[]` 实测）

| channel | unit | remaining | ok | accounts | ok_accounts | unsupported |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| workbuddy | credit | 8082.0 | true | 6 | 6 | false |
| workbuddy-intl | unknown | – | true | 0 | 0 | **true** |
| traework | credit | 2530.15 | true | 1 | 1 | false |
| raccoon | unknown | – | true | 1 | 0 | **true** |
| loomy | unknown | – | true | 1 | 0 | **true** |
| qoder | unknown | – | true | 1 | 0 | **true** |
| lobster | credit | 1443.22 | true | 1 | 1 | false |
| qwenwork | credit | 2100.0 | true | 1 | 1 | false |

⇒ **注意**：`remaining` 有值的只有 4 个渠道（workbuddy / traework / lobster / qwenwork），
其余 4 个 `unit=unknown` 且 `unsupported=true`。
**这就是我卡住的地方**：模型挂在 `raccoon` / `loomy` / `qoder` / `workbuddy-intl` 渠道下，
但网关自己标 `unsupported` ⇒ **它们算不算"免费可用"？** 我的编排器里 `paid-*` 被显式排除，
但这些渠道**既不是 `paid-*`、也没有额度数据**。

### 3) 编排器（`auto_orchestrate.py`）里与"免费"相关的硬规则

```
LIVE_CHAINS   = ("high", "flash")
POOL_CHANNELS = ("high", "flash") + (SPEED_CHAIN if SPEED_LINE_ENABLED else ())
# 池子取「所有免费线现有的席」，**刻意排除 paid-***
# 另有一条已知失效的坑：某些模型静默躺在 paid 线里，一旦编排扫到就可能被塞进免费线
```

### 4) `:8787` 的模型 id 形态（151 条，节选）

```
deepseek-v4.1-flash            workbuddy/deepseek-v4.1-flash
workbuddy-intl/deepseek-v4.1-flash   traework/deepseek-v4.1-flash
raccoon/sn-deepseek-v4-1-flash  raccoon/sn-deepseek-v4-1-flash-1017
minimax-m3.1-flash             workbuddy/minimax-m3.1-flash   traework/minimax-m3.1-flash
loomy/DeepSeek-V4.1-Flash      qwenwork/flash    webimage/agnes-image-2.5-flash
```
⇒ **同一个模型在不同渠道下 id 形态不同**（有 `sn-` 前缀、有驼峰、有的带日期后缀），
这也是"归一化后互为子串"容易误配的另一个来源。

---

## 四、你的验收清单我先收下，并加两条

你给的 14 条（①②③④ 四层各若干条）我照单收下。**我再加两条，都是这轮暴露的**：

- **① 层**：同一输入重复运行，**判定结果必须完全一致**（这轮就是 set 无序导致每次不同）。
- **③ 层**：每个"已接入"结论必须能指到一个**具体的网关模型 id**，
  且该 id 的 tail 归一化后长度 ≥6 且含数字（否则只算"家族级"或"模糊"，不下"已接入"）。

---

## 五、请你回答的六个追问

1. **第一节 (a) 的版本标记对齐**：用"日期串不同就降级家族级"来治版本混淆，
   今天已经把一条真·已接入（`20261017` ↔ `1017`）降级了。你建议怎么对齐版本标记？
   是维护一张"日期别名表"，还是改用别的判据（比如"厂商+基础型号+主版本号相同即视为型号级，
   版本后缀另行标注"）？两种做法各自的误判方向分别是什么？

2. **免费性闸门的具体判据**（用第三节的真实字段表达）：
   `accounts.provider` × `status` × `credit_limit` × `/admin/credit-summary` 的
   `unit/remaining/unsupported` × 编排器的 `paid-*` 排除规则。
   请给出一个**失败关闭**的判据顺序：第几步用什么字段判断什么、不满足时输出哪个状态。
   特别回答：`unit=unknown` 且 `unsupported=true` 的渠道（raccoon/loomy/qoder/workbuddy-intl），
   承载的模型**算不算可入免费线**？如果要算，证据是什么；如果不算，缺什么证据？

3. **实体类型 / 事件类型的可自动判定规则**：你要求区分"具体模型/家族/框架/应用产品/评测新闻/行业事件"
   和"首发/版本更新/开放权重/上线 API/评测报道/重复转载"。
   在只有 `title/summary/source/category/links` 的前提下，**哪些能靠规则可靠判、哪些只能靠语义理解**？
   请给一份**可实现的规则集**（关键词/后缀/句式），并标注每条的把握程度。

4. **原文链接抠型号：你的坚持程度**。我的担心是：今天的链接以媒体页/社媒帖为主
   （`alignment.openai.com`、`techcrunch.com`、X 帖），官方发布页占比不高；
   为此加一层 URL 解析，成本可能高于收益。你怎么看？
   如果要做，**限定在哪一类信源**（例如只对"官方发布"信源）才划算？
   还是干脆不做，改为"用摘要里的官方措辞（如"开放权重""上线 API"）代替链接证据"？

5. **① 层候选清单的最小输出**：你把它列为唯一优先项。请给出**具体字段集与顺序**
   （我想要 6~8 个字段，能直接对应"人工核对一次要看的全部信息"），
   并说明哪几个字段是**必须**的、哪几个是"有了更好"。
   另外：按你说的"别名表靠人工核验积累"，**建立一张可用的初始别名表，大概要多少人天**？
   有没有比"人工核"更省的做法（例如只核"近两周出现的、且被判为未接入的"）？

6. **你的验收清单的自动化拆分**：请把第十四节那 14 条（加上我加的 2 条）分成
   **"可以自动检查"** 与 **"必须人工看"** 两类，并说明自动检查各自需要读哪个数据源
   （我这边可读的有：AIHOT JSON 原始条目、8787 的 `/v1/models` 与 `/admin/*` 端点、
   8787 的 SQLite 库、编排器当前当班表、忆库里已落盘的日报）。

> 提醒：同目录第一轮回复的原文在 `reviews/aihot-slot-radar-review-prompt.md` 同目录的
> `reviews/` 仓内；上一轮你引用的 `qwenwork/flash` 矛盾，出处是我生成的
> `D:\记忆\参考\reference_aihot_模型动态日报.md`（已用修正后的判定重写）。
