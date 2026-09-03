# 概念解释器 · Workspace Concepts

> **本目录是整个工作空间的概念语义唯一真源（Semantic SSOT）。**
> 版本：**v2 定稿**（2026-09-03，GPT Extended 镜像问诊实读定稿，结构为「系统导航骨架 + 概念条目」双层）。

它只回答三个问题：
- 这是什么？
- 它负责什么、不负责什么？
- 关于它的事实最终去哪查？

它**不**保存项目正文、不复制项目状态、不代替项目路由、不代替 Memory 运行数据。

---

## 一句话地图

```
Workspace = Work Vault + Memory Vault

Work Vault:  通用规范(handbook) · 项目索引(workspace-index) · 逆天主题(nitian-theme) · AI平台(ai-platform)
Memory Vault: Agent 持续记忆（运行仓 ai-hub-memory）
```

## 三个真源（互不越权）

| 真源 | 管什么 |
|---|---|
| `projects.yaml` | **WHERE** —— 有哪些一级项目、去哪找 |
| `concepts/`（本目录） | **WHAT / NOT WHAT** —— 它是什么、它不是什么 |
| 各项目 Owner 仓 README/docs | **FACT** —— 它现在真实是什么 |

一句话总宪法：
> **projects.yaml 决定"去哪"；concepts 决定"它是什么、它不是什么"；项目仓决定"它现在真实是什么"；handbook 决定"默认应该怎么做"；Memory 决定"我们之前做过什么、现在接着哪里做"；飞书只提供结构化数据。**
>
> 这六句话互不越权，整个体系就不会再靠猜。

---

## 结构导航

```
concepts/
├── README.md                  ← 本文件（总入口）
├── TEMPLATE.md                六字段 + 文件/目录用途 + 冲突裁决 + 粒度宪法
├── 00-体系总览.md             Vault/仓/项目/模块/数据层 关系
├── 01-项目索引与路由.md       projects.yaml 边界说明（索引什么/不索引什么）
├── 02-Work-Vault.md           Work 库定义
├── 03-Memory-Vault.md         Memory 库定义
├── common-concepts.md         跨域概念（项目≠文件夹≠Vault、镜像/投影、逆天三身份、飞书层）
├── projects/                  每个一级项目的解释条目
│   ├── nitian-theme.md
│   ├── ai-platform.md
│   ├── handbook.md
│   └── ai-hub-memory.md
├── modules/                   项目内部模块/分类细节
│   ├── ai-platform-modules.md
│   └── memory-structure.md
└── CONSULT-2026-09-03-gpt-extended.md   ← GPT 原始问诊回复（定稿依据，只读）
```

---

## 读取顺序

**外部 Agent**（口诀：先 WHERE，后 WHAT，再 FACT）：
```
projects.yaml → concepts → 目标项目 README → docs/00 →（规范则）handbook →（跨会话则）Memory
```

**郭老师本人**：
```
概念不清楚 → concepts/
要做事     → 直接进对应项目 Home / docs
历史恢复   → Memory
```

---

## 不展开内容（别塞进 concepts）

以下**不**在本解释器展开，指向各项目 docs/：API 实现、代码逻辑、配置字段说明、架构演进历史、设计理由。