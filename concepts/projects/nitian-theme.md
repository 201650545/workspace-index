# nitian-theme · 逆天主题（仙逆）

## 基本定义（六字段）

- **名称 Name**：逆天主题（nitian-theme）
- **层级 Level**：项目 / 仓库；在 Work Vault 中同时表现为一个子库文件夹。
- **是什么（一锤定音）**：一个独立的《仙逆》深度联动修仙养成网页游戏项目；nitian-theme 是它的 Git 版本边界。
- **干什么**：保存该游戏自己的设计、任务、规格、资产台账以及生产/校验工具，形成这个游戏"当前真实是什么"的项目事实源。
- **边界（不干什么）**：不保存跨项目通用规范；不承担 Workspace 路由；不管理 AI 基础设施；不承担多 Agent 运行记忆；成品图片、视频、GLB 不因属于游戏就进入 Git 仓。
- **真源在哪**：
  - 概念定义：`concepts/projects/nitian-theme.md#nitian-theme`
  - 路由：`projects.yaml → id: nitian-theme`
  - 项目事实：`nitian-theme/README.md → docs/00-项目总览.md`
  - 通用规范：引用 handbook

## 路由入口

| 入口 | 位置 |
|---|---|
| 路由登记 | projects.yaml → id: nitian-theme |
| Agent 第一阅读 | README → docs/00-项目总览.md |
| 任务 | docs/01-任务看板.md |
| 资产 | docs/02-资产清单.md |
| 项目规格 | docs/03-规格与规范.md |

## 三身份（见 common-concepts 概念3）

Obsidian 子库 + Git 独立仓 + 业务项目，三者同时成立、互不等价。

## 关键路径文件用途

| 路径 | 一句话用途 |
|---|---|
| `docs/00-项目总览.md` | 项目事实源入口（第一阅读，回答"这游戏现在真实是什么"） |
| `docs/01-任务看板.md` | 任务状态真源 |
| `docs/02-资产清单.md` | 资产台账真源 |
| `docs/03-规格与规范.md` | 项目自己定规格/规范的地方（覆盖 handbook 默认） |

> 成品产出（图片/视频/GLB）不纳入 Git，由资产清单管理；此处仅指向 docs，不复制正文。