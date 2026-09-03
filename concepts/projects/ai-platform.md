# ai-platform · AI平台

## 基本定义（六字段）

- **名称 Name**：AI平台（ai-platform）
- **层级 Level**：项目 / 仓库；在 Work Vault 中表现为一个子库文件夹。
- **是什么（一锤定音）**：整个体系的 **AI 基础设施与能力运营主仓**：把搜索/API 网关、中央管理、GitHub 管理、组件编排、飞书数据桥、AI 资源运营以及 Agent 记忆协议入口组织成可被人和 Agent 使用的基础能力。
- **干什么**：承载平台主干 + 资源运营 + 飞书同步 + 记忆协议四个正式模块（见下方模块地图 / `modules/ai-platform-modules.md`）。
- **边界（不干什么）**：不是业务项目正文仓（不含 nitian-theme）；不是 handbook 通用规范仓；不是 Workspace 一级路由表；不是 Agent 运行记忆库；不是飞书知识正文库；不是所有 API Key/Token 的明文存储库。尤其 **agent/memory/ 只是协议快照**，运行状态/决策/工具链/正式记忆写入归 ai-hub-memory。
- **真源在哪**：
  - 概念定义：`concepts/projects/ai-platform.md`
  - 路由：`projects.yaml → id: ai-platform`
  - 项目事实：`ai-platform/README → docs/00`
  - 规格：`docs/03-规格与规范.md`
  - 模块细节：`modules/ai-platform-modules.md`

## 路由入口

| 入口 | 位置 |
|---|---|
| 路由登记 | projects.yaml → id: ai-platform |
| Agent 第一阅读 | README → docs/00 |
| 规范入口 | docs/03-规格与规范.md |
| 模块解析 | concepts/modules/ai-platform-modules.md |

## 核心模块地图（正式四模块 = project.yaml 声明，不因根目录文件夹多就全提升）

| 模块 | 是什么 | 干什么 | 真源 |
|---|---|---|---|
| 平台主干 `./` | 核心控制与能力主干 | 搜索/API 接入、中央管理、GitHub 管理、编排及平台公共能力 | project.yaml + docs/03 |
| resource-ops/ | AI 自助资源运营体系 | 收集/分类/授权 API/账号/权益/工具/网关等数字资源，供本地 AI 自主选择 | resource-ops/README.md |
| integrations/feishu/ | 飞书多维表格数据导出桥 | Bitable → 静态 JSON/schema/manifest → GitHub Pages，供 AI 读取 | integrations/feishu/README.md |
| agent/memory/ | Agent Memory 协议文档快照 | 让使用者知道记忆协议、上报格式和同步方法 | agent/memory/README.md；运行事实转 ai-hub-memory |

## 不展开内容（进对应 docs/）

API 实现、代码逻辑、配置字段说明、架构演进历史、设计理由 → ai-platform/docs/ 与各模块 README；不在 concepts 复制正文。

## 关键路径文件用途（顶层散文件，遵守范围闸门）

| 路径 | 一句话用途 |
|---|---|
| `ARCHITECTURE.md` | 平台架构高层说明（在哪：README/提供架构导读，不使用为运行状态） |
| `TOPOLOGY.md` | 组件/网关拓扑关系说明 |
| `启动AIHub.bat` | 本地一键启动中央平台/网关的入口脚本 |
| `docs/00-*.md` | 项目第一阅读总览 |
| `docs/03-规格与规范.md` | 平台规格/规范真源（网关入口、路由归属） |

> 反馈记录/问题分诊/免费资源权益清单/项目简述 等为过程性/辅助文档，不逐一建档；具体目录展开见 `modules/ai-platform-modules.md`。