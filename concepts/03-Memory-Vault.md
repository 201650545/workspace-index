# 03 · Memory Vault

> Memory 库的定义。负责整个体系跨 Agent、跨会话的"持续记忆"。

## 六字段

- **名称 Name**：Memory / 记忆库
- **层级 Level**：Vault
- **是什么（一锤定音）**：跨 Agent、跨会话保存持续状态、决策和协作记忆的独立生命周期边界。
- **干什么**：让不同 Agent 在换模型、换窗口、换执行者之后能够恢复项目当前状态与既有决策。
- **边界（不干什么）**：不保存业务项目完整正文；不代替项目仓；不承担 Workspace 路由；不把飞书当记忆正文源。
- **真源在哪**：
  - 本地运行边界：`D:\记忆`
  - Git 运行仓：`ai-hub-memory`（公开仓库，明确自定义为多 Agent 共享记忆唯一真源，实行项目隔离与 STATE/DECISIONS/CHANGELOG 分层）
  - 概念定义：`concepts/03-Memory-Vault.md`（本页）

> 关于 `D:\Work\` 与 `D:\记忆` 的分工：**Work 存干活内容，Memory 存 Agent 的持续记忆**，两个 Vault 平行，不是从属。

## 分层结构（内部细节见 `modules/memory-structure.md`）

Memory 内部按 `global/`（全局层）+ `projects/<id>/`（项目记忆线）+ `inbox/`（暂存）+ `archive/`（历史）+ 工具链分层。每个项目有 STATE / DECISIONS / CHANGELOG 三种记忆类型。

## 边界澄清

- **agent/memory 协议快照 ≠ 运行记忆**：`ai-platform/agent/memory/` 只是"记住怎么上报"的协议文档快照；真正运行状态、决策、正式记忆写入归 ai-hub-memory。在 ai-platform 的 memory 目录写运行状态无效。
- **.workbuddy/memory/**：暂定为工具私有适配/运行产物区，不升格为 Memory 核心语义层（主 README 未把它列入正式分层协议）。