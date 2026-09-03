# ai-hub-memory · 运行记忆仓

## 基本定义（六字段）

- **名称 Name**：运行记忆仓（ai-hub-memory）
- **层级 Level**：仓库 / 项目；是 Memory Vault 的可路由 Git 运行边界。
- **是什么（一锤定音）**：多 Agent 共享记忆的**唯一运行真源**：跨 Agent、跨会话保存持续状态、决策和协作历史。
- **干什么**：实行项目隔离 + global 层 + STATE/DECISIONS/CHANGELOG 分层；让换模型/换窗口/换执行者后能恢复项目当前状态与既有决策。
- **边界（不干什么）**：不存业务项目完整正文/代码；不代替项目仓；不承担 Workspace 路由；不把飞书当记忆正文源。
- **真源在哪**：
  - 概念定义：`concepts/projects/ai-hub-memory.md`（+ `modules/memory-structure.md`）
  - 路由：`projects.yaml → id: ai-hub-memory`
  - 运行事实：`ai-hub-memory` 各 `projects/<id>/STATE.md` + CHANGELOG
  - 本地 canonical：`D:\记忆`

## 路由入口

| 入口 | 位置 |
|---|---|
| 路由登记 | projects.yaml → id: ai-hub-memory（已明确 = 记忆运行系统唯一真源） |
| Agent 第一阅读 | Memory 的 README / global/ + 目标项目 STATE |
| 分层细节 | concepts/modules/memory-structure.md |

## 关键路径文件用途（memory 层，内部细节见 modules/memory-structure.md）

| 路径 | 一句话用途 |
|---|---|
| `global/` | 所有 Agent 启动时共享的全局记忆层（RULES/PROJECTS/DECISIONS/RESOURCES/TOOLS） |
| `projects/<id>/STATE.md` | "现在是什么状态"——接手任务前快速恢复当前事实 |
| `projects/<id>/DECISIONS.md` | 已拍板、后续不得随意重新猜的决策 |
| `projects/<id>/CHANGELOG` | 已完成动作的时间流水（过程追踪/交接） |
| `inbox/` | 尚未确定正式归属的隔离暂存区（先 capture 再 settle） |
| `scripts/memory.py` | Memory 机械读写与校验工具链（route/read/write/capture/settle/validate/rotate） |

> 与 ai-platform 的边界：`ai-platform/agent/memory/` 只是**协议快照**，这里才是**运行真源**。运行状态写入若放错到 ai-platform 的 memory 目录则无效。