# 02 · Work Vault

> Work 库的定义。负责管理整个"干活"侧的内容。

## 六字段

- **名称 Name**：Work Vault / Work 库
- **层级 Level**：Vault
- **是什么（一锤定音）**：郭老师整个工作体系的内容容器，承载业务/项目/规范/索引的 Obsidian 浏览工作空间；由本地 `D:\Work` 合并库承载。
- **干什么**：把 通用规范(handbook) / 项目索引(workspace-index) / 逆天主题(nitian-theme) / AI平台(ai-platform) 四个子库组织在一个可浏览的工作空间里。
- **边界（不干什么）**：不保存 Agent 持续记忆（那是 Memory Vault）；本身不是 Git 项目（容器）；不承担跨子库的版本边界（各子库各自是 .git 仓）。
- **真源在哪**：
  - 概念定义：`concepts/02-Work-Vault.md`（本页）
  - 位置：本地 `D:\Work\`

## 四个子库（内容在各自独立 Git 仓）

| 子库 | Git 仓 | dominant 职责 |
|---|---|---|
| 通用规范 | handbook | 所有项目默认规范 |
| 项目索引 | workspace-index | 路由(WHERE) + 概念(WHAT) 双真源 |
| 逆天主题 | nitian-theme | 《仙逆》游戏业务项目 |
| AI平台 | ai-platform | AI 基础设施与能力运营 |

每个子库 `.git` 完好，各自独立 remote/版本生命周期；Work 根不做独立的 git 仓。

## 细节

- **Obsidian 身份**：Work Vault 内的每个子库是一个"子库文件夹"。
- **子库 ≠ Git 仓 ≠ 项目**：三者是维度不同，见 `common-concepts.md`。
- **路径漂移注意**：各仓 `README/Home` 须自报各自当前本地路径（`D:\Work\<子库>\`），解释器不代它们纠正。