结论：做成一个小目录，放进现有 workspace-index 仓，也就是 D:\Work\项目索引\concepts\。不要新建第五个 Work 子库，不要单独开 Git 仓，也不要塞进 handbook。 projects.yaml 继续独占“项目路由真源”；concepts/ 独占“概念语义与边界真源”。

下面这版可以直接作为 concepts/README.md 的初稿落档。

Edit
概念解释器 · Workspace Concepts

本目录是整个工作空间的概念语义唯一真源（Semantic SSOT）。

它只回答三个问题：

这是什么？

它负责什么、不负责什么？

关于它的事实最终去哪查？

它不保存项目正文、不复制项目状态、不代替项目路由、不代替 Memory 运行数据。

0. 总结论
0.1 放在哪里

采用小目录，位置固定为：

D:\Work\项目索引\                 ← GitHub: workspace-index
├─ README.md
├─ projects.yaml                 ← WHERE：有哪些一级工作单元、去哪找
└─ concepts\                     ← WHAT：这些东西到底是什么
   ├─ README.md                  ← 概念解释器总入口
   ├─ 00-体系与层级.md
   ├─ 10-项目与仓库.md
   ├─ 20-AI平台.md
   ├─ 30-Memory.md
   └─ TEMPLATE.md

不建议：

D:\Work\CONCEPTS.md：Work 根本身不是独立 Git 版本边界，容易成为未版本化孤岛。

D:\Work\解释器\：会在现有四个 Work 子库之外制造第五个一级目录，而且仍然没有自己的版本归属。

handbook/：handbook 的职责是“所有项目默认应该怎么做”，不是描述“郭老师这个 Workspace 现在有哪些实体以及它们是什么意思”。现有 handbook 也明确规定：workspace-index 管“有哪些、去哪找”，项目仓管项目事实，handbook 管跨项目默认规范。

所以最合理的归属是：

workspace-index = 工作空间元数据控制层

projects.yaml = 路由真源

concepts/ = 语义真源

二者都不保存项目正文

这需要把 workspace-index README 当前“本仓只回答两件事：有哪些项目、去哪找”稍微扩成：

本仓只保存工作空间级元数据：projects.yaml 回答 WHERE，concepts/ 回答 WHAT / NOT WHAT；项目正文和运行状态仍留在各自 Owner。

当前 README 已明确 projects.yaml 是项目清单唯一事实源，并禁止 workspace-index 保存项目正文副本。

1. 六字段标准模板

所有概念一律使用同一结构：

## <Concept ID> · <名称>

- **名称 Name**：
- **层级 Level**：Vault / 项目 / 仓库 / 子库 / 子模块 / 支撑目录 / 文件 / 外部数据层
- **是什么（一锤定音）**：
- **干什么**：
- **边界（不干什么）**：
- **真源在哪**：

其中“真源在哪”必须区分两种真源：

定义真源：本 concepts/ 中的对应条目。

事实真源：真正保存项目状态、配置、运行数据或实现的 Owner。

原则：

解释器拥有“这个词是什么意思”的最终解释权，但不拥有“这个东西当前运行成什么样”的事实。

2. 冲突裁决顺序

发生信息冲突时，按“问题类型”裁决，而不是简单规定某个文件永远最大：

问题	最终真源
“这个概念是什么意思/边界在哪？”	workspace-index/concepts/
“有哪些一级项目/仓？去哪找？”	workspace-index/projects.yaml
“这个项目现在实际是什么状态？”	对应项目仓的 README / Home / docs / Owner
“这个模块具体怎么运行？”	模块 README + 实现/配置
“所有项目默认应该遵守什么？”	handbook，项目 override 优先
“Agent 当前记忆、决策、状态是什么？”	Memory / ai-hub-memory 运行仓
“飞书里现在有什么结构化记录？”	对应飞书 Base / 导出数据层

这与现有体系一致：workspace-index 只做跨仓定位；项目仓保存本项目事实；handbook 保存跨项目默认规范。

3. 外部 AI 与郭老师的读取顺序
3.1 外部 AI / 陌生 Agent

固定使用：

1. workspace-index/README.md
       ↓
2. projects.yaml
   先回答：我要去哪个一级工作单元？
       ↓
3. concepts/README.md → 对应 Concept
   再回答：这个东西是什么？边界在哪？
       ↓
4. 目标仓 README / Home / docs/00
   获取项目当前事实
       ↓
5. 对应模块 README / Owner
   获取执行细节
       ↓
6. handbook
   只有需要通用规范时才读取
       ↓
7. Memory
   只有需要跨会话状态/历史决策时才进入

口诀：

先 WHERE，后 WHAT，再 FACT。

现有 workspace-index 已规定外部 AI 先读 projects.yaml，再进目标项目 README / docs/00，最后按需查 handbook；解释器只需要插在“路由完成”和“进入项目事实”之间即可。

3.2 郭老师本人

郭老师不需要每次先扫 projects.yaml。

推荐：

Work Vault 首页
    ↓
概念不清楚 → concepts/
    ↓
要做事 → 直接进入对应项目 Home / docs
    ↓
需要历史连续性 → Memory

即：

人是“概念 → 工作”；陌生 Agent 是“路由 → 概念 → 工作”。

在 Work 首页固定放一个入口：

- [[项目索引/concepts/README|概念解释器]]
4. projects.yaml 与概念解释器如何分责
projects.yaml

唯一职责：一级工作单元路由表。

回答：

有哪些一级工作单元？

ID 是什么？

GitHub 仓在哪？

active / archived 等路由状态是什么？

从哪个入口文件开始读？

当前文件已经登记 nitian-theme、handbook、ai-platform、ai-hub-memory，并明确 ai-hub-memory 是记忆运行系统唯一真源，而 ai-platform 只保存协议快照。

concepts/

唯一职责：解释名词。

回答：

“项目”是什么意思？

“仓库”和“Vault”是不是一回事？

“AI平台”和“resource-ops”谁管什么？

“飞书项目”和 workspace 项目是不是一回事？

“agent/memory”是不是运行记忆？

“逆天主题”为什么既是子库、又是仓、又是项目？

禁止双写

concepts/ 不要复制完整项目列表、状态、URL 清单。

例如正确写法：

- 真源在哪：
  - 定义：concepts/10-项目与仓库.md#nitian-theme
  - 路由：projects.yaml 中 id=nitian-theme
  - 项目事实：nitian-theme/docs/00-项目总览.md

而不要在 concepts 再维护一份：

status: active
repo: ...
task_board: ...

这些字段属于 projects.yaml。

5. Q1 · 「项目索引」到底索引什么
一锤定音定义

“项目索引”不是所有文件、资源和数据源的目录；它是“工作空间一级、可独立路由的 Git 版本单元”的注册表。

这里的“项目”是宽义一级工作单元，不等于狭义产品项目。

所以 handbook 虽然不是产品，也可以进；Memory 虽然不是产品，也可以进。关键不是“是不是产品”，而是它是不是一级、独立、稳定、可路由的版本边界。

进入 projects.yaml 的判定规则

某样东西同时满足以下条件，才进入：

是 Workspace 的一级责任单元，不是另一个项目内部的模块或资源。

有独立 Git 仓库/版本生命周期。

有稳定入口，例如 README / project.yaml。

外部 Agent 有合理场景需要直接路由进入它。

一句机械判定：

能否作为一个独立 Git 仓被陌生 Agent 直接进入，并且进入后存在自己的一套事实/规范/生命周期？能，则进；不能，则不进。

飞书多维表格

不进。

飞书是：

结构化数据层 / 数据源，不是 Workspace 一级项目路由单元。

它可以被某个项目引用，也可以在 integrations/feishu/ 内拥有自己的数据 catalog，但那个 catalog 中出现的“project”只是数据导出 namespace，不是 workspace-index 所说的一级项目。

现有 Feishu Data Hub 的定位就是把 Bitable 数据导出为静态 JSON 给 AI 消费。

因此：

飞书 Base / 多维表格       ❌ projects.yaml
integrations/feishu        ❌ projects.yaml
ai-platform                ✅ projects.yaml
本地补项目 / 临时项目，没有 GitHub 仓

暂时不进。

它仍然可以存在于 Work 中，但在正式“升格”为一级工作单元之前，不进入公共路由真源。

升格条件：

临时目录
→ 明确长期职责
→ 建独立仓
→ README / project.yaml
→ 才登记 projects.yaml

否则公共 GitHub 索引会出现一个外部 AI 根本无法访问的 D:\... 路由，失去意义。

本地随便笔记

绝对不进。

普通笔记是知识内容，不是路由节点。

6. Q2 · 「逆天主题」要不要进项目索引

结论：要，而且现在这一行是正确的。不得删除。

原因不是因为它“位于 D:\Work 下”，而是因为它同时满足：

独立业务目标：一个《仙逆》深度联动修仙养成网页游戏。

独立 Git 仓：nitian-theme。

独立项目事实：docs/00-项目总览.md。

独立任务、资产、规格体系。

独立生命周期。

外部 Agent 有直接进入该项目工作的合理需求。

所以它有三个同时成立、但不互相等价的身份：

D:\Work\逆天主题
│
├─ Obsidian 身份：Work Vault 内的子库
├─ Git 身份：独立仓 nitian-theme
└─ 业务身份：逆天主题游戏项目

“子库”“仓库”“项目”是三个维度，不是三个同义词。

7. 六字段完整样板 · 逆天主题
nitian-theme · 逆天主题（仙逆）游戏

名称 Name：逆天主题（nitian-theme）

层级 Level：项目 / 仓库；在 Work Vault 中同时表现为一个子库文件夹。

是什么（一锤定音）：一个独立的《仙逆》深度联动修仙养成网页游戏项目；nitian-theme 是它的 Git 版本边界。

干什么：保存该游戏自己的设计、任务、规格、资产台账以及生产/校验工具，形成这个游戏“当前真实是什么”的项目事实源。其 README 当前明确指向 docs/00、任务看板、资产清单和项目规格作为 Source of Truth。

边界（不干什么）：不保存跨项目通用规范；不承担 Workspace 路由；不管理 AI 基础设施；不承担多 Agent 运行记忆；成品图片、视频、GLB 也不因为属于游戏就进入 Git 仓。

真源在哪：

概念定义：workspace-index/concepts/10-项目与仓库.md#nitian-theme

项目路由：workspace-index/projects.yaml → id: nitian-theme

项目事实：nitian-theme/README.md → docs/00-项目总览.md

任务：docs/01-任务看板.md

资产：docs/02-资产清单.md

项目规格：docs/03-规格与规范.md

通用规范：引用 handbook

8. Q3 · AI平台到底干什么
一句话职责

AI平台（ai-platform）是整个体系的 AI 基础设施与能力运营主仓：负责把搜索/API 网关、中央管理、GitHub 管理、组件编排、飞书数据桥、AI 资源运营以及 Agent 记忆协议入口组织成可被人和 Agent 使用的基础能力。

当前 README 和项目元数据把它定义为 AI 基建主仓，并明确列出平台主干、资源运营、飞书同步、记忆协议四个正式模块。

最大边界

AI平台 不是：

逆天主题等业务项目正文仓。

handbook 通用规范仓。

Workspace 一级路由表。

Agent 运行记忆库。

飞书知识正文库。

所有 API Key / Token 的明文存储库。

尤其：

agent/memory/ 只是协议快照；运行状态、决策、工具链和正式记忆写入仍归独立 Memory / ai-hub-memory。在 ai-platform 的 memory 目录写运行状态无效。

9. AI平台 · 正式模块层

ai-platform/project.yaml 当前正式只声明四个模块，因此解释器必须首先以这四个为一级模块，不能因为根目录文件夹很多就把所有目录都提升成同一级“模块”。

名称 Name	层级 Level	是什么（一锤定音）	干什么	边界（不干什么）	真源在哪
平台主干 ./	子模块	ai-platform 的核心控制与能力主干	搜索/API 接入、中央管理、GitHub 管理、编排及平台公共能力	不负责独立业务项目；不承担 Memory 正式运行数据	project.yaml + docs/03-规格与规范.md
resource-ops/	子模块	AI 自助资源运营体系	收集、分类、授权 API/账号/权益/工具/网关等数字资源，让本地 AI 自主选择使用	不等于网关运行时；不保存项目知识正文；不负责最终模型 routing	resource-ops/README.md
integrations/feishu/	子模块	飞书多维表格的数据导出桥	把 Bitable 转成静态 JSON / schema / manifest 并发布到 GitHub Pages，供 AI 读取	不做 Workspace 项目路由；不做记忆主库；不保存知识正文	integrations/feishu/README.md
agent/memory/	子模块	Agent Memory 协议文档快照	让 ai-platform 使用者知道记忆协议、上报格式和同步方法	绝不承担正式运行记忆读写；不是 Memory Vault 替身	agent/memory/README.md；运行事实转 ai-hub-memory
10. AI平台 · 平台主干内部目录

当前 GitHub 根树实际存在：

00_中央平台、04_任务卡、05_执行指令、06_组件编排器、AI日报、agent/memory、config、docs、integrations/feishu、modelscope-daily、resource-ops、tests、.github/workflows 等。当前根目录没有 tools/，所以不要凭想象创建一个 ai-platform/tools 概念。

名称 Name	层级 Level	是什么（一锤定音）	干什么	边界（不干什么）	真源在哪
00_中央平台/	子模块	ai-platform 的中央控制面	FastAPI 服务、导航面板、网关注册、GitHub 管理、飞书同步、统计等	不是每个网关自身的业务实现；不保存项目正文	docs/03 + 00_中央平台/ 实现
04_任务卡/	子模块	机器执行任务的正式任务源	保存 task_XXX 机器可消费任务卡	不是人类最终验收看板；不是历史随手记录	04_任务卡/README.md；现行规范也指定这里为机器任务真源
05_执行指令/	支撑目录	给不同执行 Agent 的执行提示/任务说明集合	约束 DeepSeek、Gemini、OpenCode 等执行位怎么执行任务	不决定任务是否存在，不代替 04_任务卡	05_执行指令/；Owner 关系见 docs/03
06_组件编排器/	子模块	AI 自主选择能力组件完成目标的编排系统	组件注册、规则卡、资产槽位、画布观察、编排运行	不是 n8n/ComfyUI 式人工预连工作流引擎；画布也不是编辑器	06_组件编排器/组件编排器架构设计.md + 组件规则卡
config/	支撑目录	平台运行配置边界	保存渠道、网关、仓库等配置/示例配置	不保存概念定义；真实凭据不得提交；不承担资源运营台账	config/ + 网关实现；当前含 channels/gateways 示例和 repos 配置
docs/	知识入口	ai-platform 的人类/Agent 项目知识入口	总览、任务看板、资产清单、规格、设计、运行手册、迁移说明	不复制各模块实现正文；不是机器任务执行源	Home.md + docs/00 + docs/03
tests/	支撑目录	平台自动验证代码	验证中央平台、网关、搜索、额度、编排等行为	不定义产品职责，不保存运行状态	tests/ 实际测试代码
.github/workflows/	支撑目录	GitHub CI/自动化执行层	push/PR/定时任务等自动化	不保存业务事实，不定义项目边界	.github/workflows/
AI日报/	产物目录	AI 行业日报的日期型输出目录	保存阶段性 AI 信息汇总	不是 ai-platform 当前状态真源，也不是项目任务板	AI日报/<date>.md；当前仓中存在日期日报
modelscope-daily/	运维子模块	ModelScope 每日权益/额度守护自动化	保持登录态、核对每日额度、生成额度日报	不是通用网关，不负责所有 AI 资源分配；属于具体资源运维自动化	modelscope-daily/部署交接文档.md
agent/	命名空间	Agent 相关能力的父路径	给 Agent 类模块提供稳定路径	本身目前不是独立业务模块；不要把 agent 与 Memory 运行仓等同	当前正式模块为 agent/memory/
integrations/	命名空间	外部系统集成的父路径	容纳飞书等外部系统适配	本身不是飞书数据，也不是一个独立项目	当前正式模块为 integrations/feishu/
11. 网关的最终边界

这是最容易混淆的一条，固定成下面这句话：

ai-platform 管网关的“平台机制、登记、控制面、配置接口和调用整合”；某次请求最终落到哪个具体渠道/模型，则由网关运行实现及其 channels / unified_models / routing 配置负责。

当前 docs/03 已把：

:3000 定义为聚合搜索入口；

:3100 定义为 API 转发入口；

渠道集合归 channels 配置；

统一模型组和实际选择归 unified_models / routing；

Owner 指向“网关实现与配置”。

因此：

resource-ops
    ↓ 提供“有什么资源可用”
网关配置 / routing
    ↓ 决定“这一请求实际走谁”
00_中央平台
    ↓ 管登记、发现、控制、管理

飞书不负责 routing。

concepts 不负责 routing。

resource-ops 不拥有最终请求 routing。

12. Memory · 一锤定音
Memory Vault

名称 Name：Memory / 记忆库

层级 Level：Vault

是什么（一锤定音）：跨 Agent、跨会话保存持续状态、决策和协作记忆的独立生命周期边界。

干什么：让不同 Agent 在换模型、换窗口、换执行者之后能够恢复项目当前状态与既有决策。

边界（不干什么）：不保存业务项目完整正文；不代替项目仓；不承担 Workspace 路由；不把飞书当记忆正文源。

真源在哪：

本地运行边界：D:\记忆

Git 运行仓：ai-hub-memory

概念定义：concepts/30-Memory.md

公开 ai-hub-memory 当前也明确把自己定义为多 Agent 共享记忆唯一真源，并实行项目隔离与 STATE / DECISIONS / CHANGELOG 分层。

13. Memory 分类
名称 Name	层级 Level	是什么（一锤定音）	干什么	边界（不干什么）	真源在哪
global/	子模块	所有 Agent 启动时共享的全局记忆层	保存全局规则、项目地图、全局决策、资源说明、工具说明	不保存某一个项目当前 STATE	global/；README 明确列出 RULES / PROJECTS / DECISIONS / RESOURCES / TOOLS
projects/<id>/	子模块	某个项目自己的正式记忆线	保存该项目 STATE / DECISIONS / CHANGELOG	不保存项目完整代码/正文；不同项目不得默认互读	projects/<id>/
STATE	文件/记忆类型	“现在是什么状态”的短上下文	Agent 接手任务前快速恢复当前事实	不记录长历史，不替代 CHANGELOG	每项目 STATE.md
DECISIONS	文件/记忆类型	已经拍板、后续不得随意重新猜的决策	防止 Agent 反复推翻既有决定	不承担当前任务流水	每项目 DECISIONS.md
CHANGELOG	文件/记忆类型	已完成动作的时间流水	保留过程追踪和交接历史	不应作为启动时主要上下文	每项目 CHANGELOG
inbox/	子模块	尚未确定正式归属的隔离暂存区	先 capture 候选事实，再 settle 到具体项目	不属于正式记忆；未 settle 前不得当作事实引用	inbox/ + memory.py settle 流程
archive/projects/	子模块	已从热记忆移出的历史材料	防止 STATE / DECISIONS 无限膨胀，同时保留追溯能力	不是当前状态，Agent 不应默认加载	archive/projects/
coordination/	支撑目录	多 Agent 并发协作协调区	保存 claims / locks，避免多个 Agent 同时抢同一写入	不属于业务知识或正式项目状态	coordination/LOCKS.md + claims/
scripts/	工具目录	Memory 的机械读写与校验工具链	route/read/write/capture/settle/validate/rotate 等	不属于记忆正文；Agent 不应绕过工具直接乱改运行文件	scripts/memory.py 等
skills/memory-router/	协议/工具目录	Agent 怎样正确访问 Memory 的行为协议	告诉 Agent 什么时候读、读哪层、如何路由	不是记忆内容本身	skills/memory-router/SKILL.md
docs/	文档目录	Memory 系统的说明、同步和上报文档	教人/Agent 使用 Memory	不是运行状态	docs/

关于当前存在的 .workbuddy/memory/：

暂定为工具私有适配/运行产物区，不要把它升格成 Memory 核心语义层。

原因是当前主 README 没有把它列入正式分层协议，而仓内实际内容表现为 WorkBuddy 日期记录及 automation 子目录。

14. 飞书概念
Feishu Structured Data Layer

名称 Name：飞书结构化数据层

层级 Level：外部数据层

是什么（一锤定音）：保存可计算、可查询、可同步的结构化记录的外部数据库层。

干什么：存 API、网关、配额、资源清单等表格型数据；通过 ai-platform 集成/导出供 Agent 消费。

边界（不干什么）：不是 Work Vault；不是 Memory；不是知识正文主库；每一张表也不是一个 workspace 项目。

真源在哪：

数据本身：对应飞书 Base

导出机制：ai-platform/integrations/feishu/

概念定义：本解释器

15. 最终项目索引判定表
对象	进 projects.yaml？	原因
nitian-theme	✅	一级项目 + 独立仓 + 独立事实/生命周期
ai-platform	✅	一级 AI 基建工作单元 + 独立仓
handbook	✅	一级规范工作单元 + 独立仓，虽不是产品项目
ai-hub-memory	✅	一级运行记忆工作单元 + 独立生命周期
飞书 Base	❌	数据层，不是版本/路由单元
飞书某张多维表	❌	数据对象
ai-platform/resource-ops	❌	ai-platform 内部模块
ai-platform/integrations/feishu	❌	ai-platform 内部模块
00_中央平台	❌	ai-platform 内部模块
Work 内临时无仓项目	❌	尚未升格为稳定一级路由单元
随手笔记	❌	知识内容
Work Vault	❌	容器，不是 Git 项目
Memory Vault	❌	容器/运行边界；真正路由对象是其运行仓
16. 三个最重要的“不等号”

以后所有 Agent 强制记住：

Vault ≠ Git 仓库

Vault 是 Obsidian 的浏览/知识工作空间；Git 仓库是版本、权限、历史和发布边界。

项目 ≠ 文件夹

“项目”是责任和生命周期概念；文件夹只是物理承载方式。

数据源 ≠ 项目

飞书 Base、JSON、API、资产清单等可以服务项目，但不会因此自动成为 workspace-index 的项目。

17. 当前 GitHub 需要顺手消除的一处漂移

新的两 Vault 结构已经确定后，旧本地路径不要继续成为隐形第二真源。

当前公开仓仍残留旧路径，例如：

ai-platform/Home.md 仍写本地 D:\AI平台；

ai-platform/docs/00 仍写 D:\AI平台、D:\项目索引；

nitian-theme/README 仍记录项目根 D:\逆天主题。

新体系既然已经是：

D:\Work\
├─ 通用规范
├─ 项目索引
├─ 逆天主题
└─ AI平台

D:\记忆\

那么各仓下一次文档维护时应统一修改自己的“当前本地路径”。

不要让 concepts 长期承担“覆盖旧 README 错路径”的责任。

解释器负责解释；

项目 README/Home 仍必须对自己的实际位置负责。

18. 一句话总宪法

projects.yaml 决定“去哪”；concepts 决定“它是什么、它不是什么”；项目仓决定“它现在真实是什么”；handbook 决定“默认应该怎么做”；Memory 决定“我们之前做过什么、现在接着哪里做”；飞书只提供结构化数据。

这六句话互不越权，整个体系就不会再靠猜。

这版最关键的结构决策就是：把“解释器”并入 workspace-index 的元数据职责，但不给它项目路由权。 这样既有 Git 版本、外部 Agent 可读，也不会制造第五个库或第五种真源。
