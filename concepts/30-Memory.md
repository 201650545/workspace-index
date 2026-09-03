# 30 · Memory & 飞书

## Memory Vault

### Memory / 记忆库
- **名称 Name**：Memory / 记忆库
- **层级 Level**：Vault
- **是什么（一锤定音）**：跨 Agent、跨会话保存持续状态、决策和协作记忆的独立生命周期边界。
- **干什么**：让不同 Agent 在换模型、换窗口、换执行者之后能够恢复项目当前状态与既有决策。
- **边界（不干什么）**：不保存业务项目完整正文；不代替项目仓；不承担 Workspace 路由；不把飞书当记忆正文源。
- **真源在哪**：本地运行边界 `D:\记忆` ｜ Git 运行仓 `ai-hub-memory` ｜ 概念定义 `concepts/30-Memory.md`。

> 公开 ai-hub-memory 当前明确把自己定义为多 Agent 共享记忆唯一真源，并实行项目隔离与 STATE / DECISIONS / CHANGELOG 分层。

## Memory 分类

| 名称 Name | 层级 Level | 是什么（一锤定音） | 干什么 | 边界（不干什么） | 真源在哪 |
|---|---|---|---|---|---|
| global/ | 子模块 | 所有 Agent 启动时共享的全局记忆层 | 保存全局规则、项目地图、全局决策、资源说明、工具说明 | 不保存某一个项目当前 STATE | global/；README 明确列出 RULES / PROJECTS / DECISIONS / RESOURCES / TOOLS |
| projects/\<id\>/ | 子模块 | 某个项目自己的正式记忆线 | 保存该项目 STATE / DECISIONS / CHANGELOG | 不保存项目完整代码/正文；不同项目不得默认互读 | projects/\<id\>/ |
| STATE | 文件/记忆类型 | "现在是什么状态"的短上下文 | Agent 接手任务前快速恢复当前事实 | 不记录长历史，不替代 CHANGELOG | 每项目 STATE.md |
| DECISIONS | 文件/记忆类型 | 已经拍板、后续不得随意重新猜的决策 | 防止 Agent 反复推翻既有决定 | 不承担当前任务流水 | 每项目 DECISIONS.md |
| CHANGELOG | 文件/记忆类型 | 已完成动作的时间流水 | 保留过程追踪和交接历史 | 不应作为启动时主要上下文 | 每项目 CHANGELOG |
| inbox/ | 子模块 | 尚未确定正式归属的隔离暂存区 | 先 capture 候选事实，再 settle 到具体项目 | 不属于正式记忆；未 settle 前不得当作事实引用 | inbox/ + memory.py settle 流程 |
| archive/projects/ | 子模块 | 已从热记忆移出的历史材料 | 防止 STATE / DECISIONS 无限膨胀，同时保留追溯能力 | 不是当前状态，Agent 不应默认加载 | archive/projects/ |
| coordination/ | 支撑目录 | 多 Agent 并发协作协调区 | 保存 claims / locks，避免多个 Agent 同时抢同一写入 | 不属于业务知识或正式项目状态 | coordination/LOCKS.md + claims/ |
| scripts/ | 工具目录 | Memory 的机械读写与校验工具链 | route/read/write/capture/settle/validate/rotate 等 | 不属于记忆正文；Agent 不应绕过工具直接乱改运行文件 | scripts/memory.py 等 |
| skills/memory-router/ | 协议/工具目录 | Agent 怎样正确访问 Memory 的行为协议 | 告诉 Agent 什么时候读、读哪层、如何路由 | 不是记忆内容本身 | skills/memory-router/SKILL.md |
| docs/ | 文档目录 | Memory 系统的说明、同步和上报文档 | 教人/Agent 使用 Memory | 不是运行状态 | docs/ |

> 关于 `.workbuddy/memory/`：暂定为**工具私有适配/运行产物区**，不要升格成 Memory 核心语义层。当前主 README 没把它列入正式分层协议，仓内实际表现为 WorkBuddy 日期记录及 automation 子目录。

## 飞书结构化数据层

### Feishu Structured Data Layer
- **名称 Name**：飞书结构化数据层
- **层级 Level**：外部数据层
- **是什么（一锤定音）**：保存可计算、可查询、可同步的结构化记录的外部数据库层。
- **干什么**：存 API、网关、配额、资源清单等表格型数据；通过 ai-platform 集成/导出供 Agent 消费。
- **边界（不干什么）**：不是 Work Vault；不是 Memory；不是知识正文主库；每一张表也不是一个 workspace 项目。
- **真源在哪**：数据本身 = 对应飞书 Base ｜ 导出机制 = ai-platform/integrations/feishu/ ｜ 概念定义 = 本解释器。