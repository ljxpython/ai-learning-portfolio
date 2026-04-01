# ai-agent-test-platform 实操记录

这部分内容围绕 [`ai-agent-test-platform`](https://github.com/ljxpython/ai-agent-test-platform) 的实操、验证和复盘展开，重点不是重复贴源码，而是把一套可以反复复用的工作方式沉淀下来。

和根目录 README 的区别是：

- 根 README 负责说明项目版图、仓库关系和演进脉络
- 这里负责说明具体怎么做、怎么验证、怎么复盘

如果说根 README 讲的是“项目地图”，这里讲的就是“落地路径”。

## 这组记录主要解决什么问题

当前这批文档主要围绕三个核心问题展开：

- 怎么让 AI 先读懂现有代码，再开始开发和接入
- 怎么把需求拆成可执行的提示词、阶段目标和交付路径
- 怎么在开发完成后继续做验证、修复和复盘，而不是只停留在“功能写完”

## 快速入口

- 返回仓库首页：[`../README.md`](../README.md)
- 主线项目源码：<https://github.com/ljxpython/ai-agent-test-platform>
- 开发范式总览：[`20260325_project_development_paradigm.md`](./20260325_project_development_paradigm.md)
- 部署与验证入口：[`20260323_deployment_environment.md`](./20260323_deployment_environment.md)
- 单智能体案例：[`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)
- 多智能体案例：[`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)
- 测试用例平台闭环：[`20260330_requirement_optimize.md`](./20260330_requirement_optimize.md)

## 实践主线

这组记录目前可以理解成一条从“先跑起来”到“做复杂场景”，再到“把复杂场景真正接成平台能力”的连续主线：

| 阶段 | 主题 | 你能得到什么 | 文档 |
|---|---|---|---|
| 第一步 | 部署与验证基线 | 一套用 AI 辅助完成环境部署、问题排查和结果验证的路径 | [`20260323_deployment_environment.md`](./20260323_deployment_environment.md) |
| 第二步 | 单智能体最小闭环 | 一个从阅读代码、讨论方案到接入图表 MCP 的最小可运行案例 | [`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md) |
| 第三步 | 复杂业务多智能体 | 一个把上传文档、解析、评审、落库串起来的复杂场景拆解方案 | [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md) |
| 第四步 | 测试用例平台闭环优化 | 一条从 Skills 编排、interaction-data-service 落库，到 platform-api / platform-web 工作区接入的完整收口记录 | [`20260330_requirement_optimize.md`](./20260330_requirement_optimize.md) |

## 按目标阅读

### 如果你只想先把项目跑起来

1. 先看 [`20260323_deployment_environment.md`](./20260323_deployment_environment.md)。
2. 把环境、依赖和验证入口先跑通。
3. 跑通之后再决定要继续看单智能体，还是直接看复杂场景。

### 如果你想先搞懂这个项目为什么这么设计

1. 先看主仓库 README，知道系统版图。
2. 再看 [`20260325_project_development_paradigm.md`](./20260325_project_development_paradigm.md)。
3. 这篇主要补“平台侧为什么浅封装、为什么要功能解耦、后面应该按什么节奏继续开发”。

### 如果你想看一个最小可行 Agent 是怎么接进平台的

1. 先补一眼部署文档，至少知道环境怎么起。
2. 再看 [`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)。
3. 这篇最适合用来理解“AI 先读代码，再讨论方案，再落到实现”的基本节奏。

### 如果你想看复杂业务怎么拆成多智能体

1. 先确认自己已经知道项目的基础运行方式。
2. 然后直接看 [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)。
3. 这篇更关注复杂业务流转、职责边界和后续演进，不是入门案例。

### 如果你想看测试用例链路怎么真正落到平台里

1. 先补 [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)，知道复杂场景最初是怎么拆的。
2. 再看 [`20260330_requirement_optimize.md`](./20260330_requirement_optimize.md)。
3. 这篇更关注 Skills 编排、结果域服务、平台工作区接入和真实联调，不只是设计草图。

### 如果你想做完整复盘

按下面顺序看最顺：

1. [`20260323_deployment_environment.md`](./20260323_deployment_environment.md)
2. [`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)
3. [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)
4. [`20260330_requirement_optimize.md`](./20260330_requirement_optimize.md)

这样能看到从部署准备、单点能力接入，到复杂场景设计的完整脉络。

## 这组文档和主线项目的关系

- 主线项目 [`ai-agent-test-platform`](https://github.com/ljxpython/ai-agent-test-platform) 承载真实工程代码。
- 这里不重复贴源码，而是补源码仓库里不适合展开写的上下文。
- 这些记录更偏“为什么这样做、过程中踩了什么坑、后面还能怎么优化”。

如果你是第一次进这个作品集仓库，建议先看根目录 [`../README.md`](../README.md) 了解整体项目关系，再回到这里按阅读目标进入具体文档。

## 记录约定

- 图片统一放在 `assets/` 目录，按文档内引用使用。
- 文档内容以实操记录和复盘为主，不是完整源码镜像。
- 后续如果继续新增案例，建议沿用 `YYYYMMDD_主题.md` 的命名方式。
- 新增文档时，优先补齐它在“实践主线”中的位置，避免索引页重新变成一堆散链接。
