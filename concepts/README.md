# 概念解释器 · Workspace Concepts

> **本目录是整个工作空间的概念语义唯一真源（Semantic SSOT）。**
> 版本：**v2 定稿**（2026-09-03，GPT Extended 镜像问诊实读定稿，落于 workspace-index/concepts/）。

它只回答三个问题：
- 这是什么？
- 它负责什么、不负责什么？
- 关于它的事实最终去哪查？

它**不**保存项目正文、不复制项目状态、不代替项目路由、不代替 Memory 运行数据。

---

## 0. 总结论 · 放在哪里

采用小目录，位置固定为：

```
D:\Work\项目索引\                 ← GitHub: workspace-index
├─ README.md
├─ projects.yaml                 ← WHERE：有哪些一级工作单元、去哪找
└─ concepts\                     ← WHAT：这些东西到底是什么
   ├─ README.md                  ← 概念解释器总入口（本文件）
   ├─ 00-体系与层级.md
   ├─ 10-项目与仓库.md
   ├─ 20-AI平台.md
   ├─ 30-Memory.md
   └─ TEMPLATE.md
```

**不建议**：
- `D:\Work\CONCEPTS.md`：Work 根本身不是独立 Git 版本边界，容易成为未版本化孤岛。
- `D:\Work\解释器\`：会在现有四个 Work 子库之外制造第五个一级目录，仍没有自己的版本归属。
- `handbook/`：handbook 职责是"所有项目默认应该怎么做"，不是描述"这个 Workspace 现在有哪些实体以及它们是什么意思"。

**归属定死**：
```
workspace-index        = 工作空间元数据控制层
projects.yaml          = 路由真源（WHERE：有哪些一级工作单元、去哪找）
concepts/              = 语义真源（WHAT / NOT WHAT：这些东西到底是什么）
```
二者都不保存项目正文。

---

## 01. 一句话总宪法（§18）

> **projects.yaml 决定"去哪"；concepts 决定"它是什么、它不是什么"；项目仓决定"它现在真实是什么"；handbook 决定"默认应该怎么做"；Memory 决定"我们之前做过什么、现在接着哪里做"；飞书只提供结构化数据。**
>
> 这六句话互不越权，整个体系就不会再靠猜。

**最关键的架构决策**：把"解释器"并入 workspace-index 的元数据职责，但**不给它项目路由权**。这样既有 Git 版本、外部 Agent 可读，也不会制造第五个库或第五种真源。

---

## 02. 从哪读起

| 角色 | 读取顺序 |
|---|---|
| 外部 AI / 陌生 Agent | ① workspace-index/README → ② projects.yaml（我要去哪个一级工作单元？）→ ③ concepts/README→对应条目（这是什么？边界？）→ ④ 目标仓 README/Home/docs/00（当前事实）→ ⑤ 对应模块 README/Owner（执行细节）→ ⑥ handbook（需要通用规范时）→ ⑦ Memory（需要跨会话状态/历史决策时）。**口诀：先 WHERE，后 WHAT，再 FACT。** |
| 郭老师本人 | 概念不清楚 → concepts/ ←；要做事 → 直接进对应项目 Home/docs；需要历史连续性 → Memory。**人是"概念→工作"，陌生 Agent 是"路由→概念→工作"。** |

Work 首页固定放一个入口：
```
[[项目索引/concepts/README|概念解释器]]
```

---

## 03. 子文件导航

| 文件 | 内容 |
|---|---|
| `00-体系与层级.md` | 三个不等号（Vault≠仓库≠项目）、Vault 层（Work/Memory）、进 projects.yaml 的判定规则 |
| `10-项目与仓库.md` | Q1 项目索引索引什么、Q2 逆天主题、各项目条目样板、最终判定表 |
| `20-AI平台.md` | Q3 AI平台到底干什么、正式模块层、平台主干内部目录、网关最终边界 |
| `30-Memory.md` | Memory、Memory 分类、飞书结构化数据层 |
| `TEMPLATE.md` | 六字段模板 + 两种真源 + 冲突裁决表 |

---

## 04. 解释器 vs 其它"真源"的分责（不双写）

concepts/ **禁止**复制完整项目列表、状态、URL。一个概念条目的"真源在哪"正确写法：
```
- 真源在哪：
  - 定义：concepts/10-项目与仓库.md#nitian-theme
  - 路由：projects.yaml 中 id=nitian-theme
  - 项目事实：nitian-theme/docs/00-项目总览.md
```
而 **不要在 concepts 再维护一份** `status/ repo/ task_board/` ——这些字段属于 projects.yaml。