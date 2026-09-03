# workspace-index · 工作空间索引

> AI 统一入口。本仓只回答两件事：**有哪些项目、去哪找**。不保存任何项目的正文副本。

## 读取顺序（外部 AI 必读）

1. 读 `projects.yaml` 定位项目
2. 进入目标项目仓：读它的 README → docs/00 总览
3. 需要全局规范时查 handbook（注意项目声明的版本号）
4. 状态时效：见 `STATUS.md` 顶部时间戳；GitHub 内容可能落后于本地

## 仓库清单

见 [projects.yaml](projects.yaml)（唯一事实源，本 README 不重复罗列）。

## 维护规则

- 新项目接入：project.yaml + README 按 `handbook/templates/` 生成，然后在本仓 projects.yaml 登记一行
- 本仓不复制项目状态细节（那是各项目仓任务看板的事）
- 项目归档/冻结：改 projects.yaml 的 status，并在 STATUS.md 留一行快照

---
最后更新：2026-09-03（来源：本地 vault，同步方式：手动 push）
