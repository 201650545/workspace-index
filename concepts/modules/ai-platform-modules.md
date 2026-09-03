# modules · ai-platform 模块地图

> ai-platform 的内部模块展开（正式模块 + 平台主干内部目录）。只写"是什么/边界/真源"，不复制实现正文。

## 正式四模块（project.yaml 声明，一级模块以此为准）

| 名称 | 层级 | 是什么（一锤定音） | 干什么 | 边界（不干什么） | 真源 |
|---|---|---|---|---|---|
| 平台主干 `./` | 核心模块 | ai-platform 的核心控制与能力主干 | 搜索/API 接入、中央管理、GitHub 管理、编排及平台公共能力 | 不负责独立业务项目；不承担 Memory 正式运行数据 | project.yaml + docs/03-规格与规范.md |
| resource-ops/ | 核心模块 | AI 自助资源运营体系 | 收集、分类、授权 API/账号/权益/工具/网关等数字资源，让本地 AI 自主选择使用 | 不等于网关运行时；不保存项目知识正文；不负责最终模型 routing | resource-ops/README.md |
| integrations/feishu/ | 核心模块 | 飞书多维表格的数据导出桥 | 把 Bitable 转成静态 JSON/schema/manifest 并发布到 GitHub Pages，供 AI 读取 | 不做 Workspace 项目路由；不做记忆主库；不保存知识正文 | integrations/feishu/README.md |
| agent/memory/ | 核心模块 | Agent Memory 协议文档快照 | 让 ai-platform 使用者知道记忆协议、上报格式和同步方法 | 绝不承担正式运行记忆读写；不是 Memory Vault 替身 | agent/memory/README.md；运行事实转 ai-hub-memory |

## 平台主干内部目录（GitHub 根树实际存在，不凭想象补目录）

| 名称 | 层级 | 是什么（一锤定音） | 干什么 | 边界（不干什么） | 真源 |
|---|---|---|---|---|---|
| `00_中央平台/` | 核心模块 | ai-platform 的中央控制面 | FastAPI 服务、导航面板、网关注册、GitHub 管理、飞书同步、统计等 | 不是每个网关自身的业务实现；不保存项目正文 | docs/03 + 00_中央平台/ 实现 |
| `04_任务卡/` | 核心模块 | 机器执行任务的正式任务源 | 保存 task_XXX 机器可消费任务卡 | 不是人类最终验收看板；不是历史随手记录 | 04_任务卡/README.md |
| `05_执行指令/` | 支撑目录 | 给不同执行 Agent 的执行提示/任务说明集合 | 约束 DeepSeek、Gemini、OpenCode 等执行位怎么执行任务 | 不决定任务是否存在，不代替 04_任务卡 | 05_执行指令/；Owner 关系见 docs/03 |
| `06_组件编排器/` | 核心模块 | AI 自主选择能力组件完成目标的编排系统 | 组件注册、规则卡、资产槽位、画布观察、编排运行 | 不是 n8n/ComfyUI 式人工预连工作流引擎；画布也不是编辑器 | 06_组件编排器/架构设计.md + 组件规则卡 |
| `config/` | 支撑目录 | 平台运行配置边界 | 保存渠道、网关、仓库等配置/示例配置 | 不保存概念定义；真实凭据不得提交；不承担资源运营台账 | config/ + 网关实现 |
| `docs/` | 支撑目录 | ai-platform 的人类/Agent 项目知识入口 | 总览、任务看板、资产清单、规格、设计、运行手册、迁移说明 | 不复制各模块实现正文；不是机器任务执行源 | Home.md + docs/00 + docs/03 |
| `tests/` | 支撑目录 | 平台自动验证代码 | 验证中央平台、网关、搜索、额度、编排等行为 | 不定义产品职责，不保存运行状态 | tests/ 实际测试代码 |
| `.github/workflows/` | 支撑目录 | GitHub CI/自动化执行层 | push/PR/定时任务等自动化 | 不保存业务事实，不定义项目边界 | .github/workflows/ |
| `AI日报/` | 产物目录 | AI 行业日报的日期型输出目录 | 保存阶段性 AI 信息汇总 | 不是 ai-platform 当前状态真源，也不是项目任务板 | AI日报/<date>.md |
| `modelscope-daily/` | 运维子模块 | ModelScope 每日权益/额度守护自动化 | 保持登录态、核对每日额度、生成额度日报 | 不是通用网关，不负责所有 AI 资源分配 | modelscope-daily/部署交接文档.md |
| `agent/` | 命名空间 | Agent 相关能力的父路径 | 给 Agent 类模块提供稳定路径 | 本身不是独立业务模块；不要把 agent 与 Memory 运行仓等同 | 当前正式模块为 agent/memory/ |
| `integrations/` | 命名空间 | 外部系统集成的父路径 | 容纳飞书等外部系统适配 | 本身不是飞书数据，也不是独立项目 | 当前正式模块为 integrations/feishu/ |

> 说明：当前 GitHub 根树没有 `tools/`，**不要凭想象建 ai-platform/tools 概念**。顶层散文件（ARCHITECTURE/TOPOLOGY/启动AIHub.bat 等）见 `projects/ai-platform.md`。