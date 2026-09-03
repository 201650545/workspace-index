# handbook · 通用规范

## 基本定义（六字段）

- **名称 Name**：通用规范（handbook）
- **层级 Level**：项目 / 仓库；在 Work Vault 中表现为一个子库文件夹。
- **是什么（一锤定音）**：**"所有项目默认应该怎么做"** 的跨项目通用规范仓；是行为/流程的默认真源，不是某个项目的正文。
- **干什么**：沉淀跨项目统一规范（课件生产规范、知识管理规范、命名规范、格式规范等），供所有人/Agent 按默认方式做事。
- **边界（不干什么）**：不描述 Workspace 有哪些实体（那是 workspace-index）；不保存单个业务项目事实（那是项目仓）；不承担 Agent 运行记忆；不复制项目正文。
- **真源在哪**：
  - 概念定义：`concepts/projects/handbook.md`
  - 路由：`projects.yaml → id: handbook`
  - 项目事实：`handbook/README → docs/00`（规范本体的最新入口）

## 路由入口

| 入口 | 位置 |
|---|---|
| 路由登记 | projects.yaml → id: handbook |
| Agent 第一阅读 | README → docs/00 |
| 规范入口 | docs/（各章） |

## 关键路径文件用途

| 路径 | 一句话用途 |
|---|---|
| `docs/00-*.md` | 规范总览入口（先读这判断用哪章） |
| 各章规范（01/02/…/格式规范等） | 某类事务的默认规范正文，项目可 override |

## 项目 override 关系

项目仓可以在自己的 `docs/03-规格与规范` 里声明覆盖 handbook 默认（项目 override 优先，见 TEMPLATE 冲突裁决）。handbook 是"默认"，不是"强制唯一"。

> 本页只解释 handbook 是谁、管什么；具体章节内容在 handbook 仓内，不在 concepts 复制。