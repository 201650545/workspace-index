# modules · Memory 分类

> ai-hub-memory 运行仓的内部结构分解。只写"是什么/边界/真源"，不复制记忆正文。

## 一锤定音

Memory 运行仓 `ai-hub-memory` = 多 Agent 共享记忆唯一真源，实行**项目隔离 + 分层检索（Routing-before-Retrieval）+ Fail-Closed**。

## 分类表

| 名称 | 层级 | 是什么（一锤定音） | 干什么 | 边界（不干什么） | 真源 |
|---|---|---|---|---|---|
| `global/` | 核心模块 | 所有 Agent 启动时共享的全局记忆层 | 保存全局规则、项目地图、全局决策、资源说明、工具说明 | 不保存某一个项目当前 STATE | global/（README 明确列出 RULES/PROJECTS/DECISIONS/RESOURCES/TOOLS） |
| `projects/<id>/` | 核心模块 | 某个项目自己的正式记忆线 | 保存该项目 STATE/DECISIONS/CHANGELOG | 不保存项目完整代码/正文；不同项目不得默认互读 | projects/<id>/ |
| STATE | 文件/记忆类型 | "现在是什么状态"的短上下文 | Agent 接手任务前快速恢复当前事实 | 不记录长历史，不替代 CHANGELOG | 每项目 STATE.md |
| DECISIONS | 文件/记忆类型 | 已拍板、后续不得随意重新猜的决策 | 防止 Agent 反复推翻既有决定 | 不承担当前任务流水 | 每项目 DECISIONS.md |
| CHANGELOG | 文件/记忆类型 | 已完成动作的时间流水 | 保留过程追踪和交接历史 | 不应作为启动时主要上下文 | 每项目 CHANGELOG |
| `inbox/` | 核心模块 | 尚未确定正式归属的隔离暂存区 | 先 capture 候选事实，再 settle 到具体项目 | 不属于正式记忆；未 settle 前不得当作事实引用 | inbox/ + memory.py settle 流程 |
| `archive/projects/` | 核心模块 | 已从热记忆移出的历史材料 | 防止 STATE/DECISIONS 无限膨胀，保留追溯 | 不是当前状态，Agent 不应默认加载 | archive/projects/ |
| `coordination/` | 支撑目录 | 多 Agent 并发协作协调区 | 保存 claims/locks，避免多 Agent 抢同一写入 | 不属于业务知识或正式项目状态 | coordination/LOCKS.md + claims/ |
| `scripts/` | 工具目录 | Memory 的机械读写与校验工具链 | route/read/write/capture/settle/validate/rotate | 不属于记忆正文；Agent 不应绕过工具直接乱改运行文件 | scripts/memory.py 等 |
| `skills/memory-router/` | 协议/工具目录 | Agent 怎样正确访问 Memory 的行为协议 | 告诉 Agent 什么时候读、读哪层、如何路由 | 不是记忆内容本身 | skills/memory-router/SKILL.md |
| `docs/` | 文档目录 | Memory 系统的说明、同步和上报文档 | 教人/Agent 使用 Memory | 不是运行状态 | docs/ |

## 读取协议（Routing-before-Retrieval）

1. 先按"是全局还是某项目"路由到正确层（`global/` 或 `projects/<id>/`）。
2. 再按记忆类型取对应文件（STATE 恢复状态 → DECISIONS 知已拍板 → CHANGELOG 查流水）。
3. 未 settle 的 inbox 内容不得当事实引用（Fail-Closed）。

## 明确不在 Memory 语义层的内容

- **`.workbuddy/memory/`**：工具私有适配/运行产物区，不升格为 Memory 核心语义层（主 README 未列入正式分层协议）。
- **ai-platform/agent/memory/**：协议快照，不是运行真源（运行在 `ai-hub-memory`）。