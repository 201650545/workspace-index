# workspace-index · 工作空间索引

> AI 统一入口。本仓**只保存工作空间级元数据**：`projects.yaml` 回答 **WHERE**（有哪些项目、去哪找），`concepts/` 回答 **WHAT / NOT WHAT**（这些东西到底是什么）。项目正文和运行状态仍留在各自 Owner，本仓不保存任何项目的正文副本。

## 结构

```
workspace-index/
├─ projects.yaml       ← 路由真源（WHERE）
├─ concepts/           ← 概念语义真源（WHAT / NOT WHAT）：概念解释器
│  ├─ README.md        ← 解释器总入口
│  ├─ TEMPLATE.md      ← 六字段 + 文件/目录用途 + 冲突裁决 + 粒度宪法
│  ├─ 00-体系总览.md   Vault/仓/项目/模块/数据层 关系
│  ├─ 01-项目索引与路由.md   projects.yaml 边界说明
│  ├─ 02-Work-Vault.md     03-Memory-Vault.md
│  ├─ common-concepts.md   跨域概念（项目≠文件夹≠Vault 等）
│  ├─ projects/        ← 每个一级项目的解释条目
│  ├─ modules/         ← 项目内部模块/分类细节
│  └─ CONSULT-2026-09-03-gpt-extended.md  ← GPT 原始问诊回复（定稿依据）
└─ STATUS.md
```

## 读取顺序（外部 AI 必读）

1. 读 `projects.yaml` 定位项目（WHERE：去哪个一级工作单元）
2. 概念不清时读 `concepts/` 对应条目（WHAT：这是什么、边界在哪）——先 WHERE，后 WHAT，再 FACT
3. 进入目标项目仓：读它的 README → docs/00 总览（当前事实）
4. 需要全局规范时查 handbook（注意项目声明的版本号）
5. 状态时效：见 `STATUS.md` 顶部时间戳；GitHub 内容可能落后于本地

## 仓库清单

见 [projects.yaml](projects.yaml)（唯一事实源，本 README 不重复罗列）。

## 维护规则

- 新项目接入：project.yaml + README 按 `handbook/templates/` 生成，然后在本仓 projects.yaml 登记一行
- 本仓不复制项目状态细节（那是各项目仓任务看板的事）
- 项目归档/冻结：改 projects.yaml 的 status，并在 STATUS.md 留一行快照

---
最后更新：2026-09-03（来源：本地 vault，同步方式：手动 push）
