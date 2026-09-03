# common-concepts · 跨域共用概念

> 会被多个项目反复引用的通用概念。避免每个项目条目里重复解释。

## 概念 1 · 项目 ≠ 仓库 ≠ Vault（三个不等号）

- **Vault ≠ Git 仓库**：Vault 是 Obsidian 的浏览/知识工作空间；Git 仓库是版本、权限、历史和发布边界。
- **项目 ≠ 文件夹**："项目"是责任和生命周期概念；文件夹只是物理承载方式。
- **数据源 ≠ 项目**：飞书 Base、JSON、API、资产清单等可以服务项目，但不会因此自动成为一级项目。

**推论**：一个事物可以同时有三个互不等价的身份（详见"身份叠加"）。

## 概念 2 · 镜像 / 投影（脱敏副本）

- **定义**：镜像是对某个 canonical 事实源做脱敏、降权、可公开的副本，用于 GitHub/外部 AI 消费，不以镜像为真源。
- **例**：`D:\记忆` = local canonical；GitHub `yongtai-memory` = 脱敏镜像；`_private_bak/` 本地保存脱敏前内容。
- **真源原则**：local canonical 为真，镜像只读引用；安全红线——密钥/API Key 不入库不入文档不入镜像。

## 概念 3 · 逆天主题的三身份（O2 结论，正确保留）

逆天主题 = `D:\Work\逆天主题`，三个身份同时成立但不互相等价：

```
D:\Work\逆天主题
├─ Obsidian 身份：Work Vault 内的子库
├─ Git 身份：独立仓 nitian-theme
└─ 业务身份：逆天主题游戏项目（《仙逆》深度联动修仙养成网页游戏）
```

**"子库""仓库""项目"是三个维度，不是三个同义词。** 因此它必须进 projects.yaml（判定见 01）。

## 概念 4 · 飞书结构化数据层（外部数据层）

- **是什么**：保存可计算、可查询、可同步的结构化记录的外部数据库层。
- **干什么**：存 API、网关、配额、资源清单等表格型数据；通过 ai-platform 集成/导出（Bitable → 静态 JSON/schema/manifest → GitHub Pages）供 Agent 消费。
- **边界（不干什么）**：不是 Work Vault；不是 Memory；不是知识正文主库；每一张表也不是一个 workspace 项目。
- **真源**：数据本身 → 对应飞书 Base；导出机制 → ai-platform/integrations/feishu/；概念定义 → 本解释器。

## 概念 5 · 网关的最终边界（最容易混淆的一条）

> **ai-platform 管网关的"平台机制、登记、控制面、配置接口和调用整合"；某次请求最终落到哪个具体渠道/模型，则由网关运行实现及其 channels / unified_models / routing 配置负责。**

三段流：
```
resource-ops        ↓ 提供"有什么资源可用"
网关配置 / routing  ↓ 决定"这一请求实际走谁"
00_中央平台         ↓ 管登记、发现、控制、管理
```
**飞书不负责 routing；concepts 不负责 routing；resource-ops 不拥有最终请求 routing。**

## 概念 6 · 禁止双写

`concepts/` 不复制项目列表、状态、URL 清单、status/repo/task_board 等字段——这些属于 projects.yaml。concepts 只解释"是什么、边界、去哪查"，事实字段指向各 Owner。

---
- 概念条目统一用 `TEMPLATE.md` 六字段结构维护；本页收录的是跨域共用项。