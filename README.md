# AI 学习作品集（ai-learning-portfolio）

本仓库用于统一整理我的 AI 项目导航、架构演进脉络和本地实操记录。

这里不存放完整业务代码，主要承担三件事：

- 汇总各个外部项目仓库的入口
- 说明项目之间的继承、拆分和整合关系
- 沉淀本地实践记录、提示词、复盘和方法总结

截至 2026-03-23，当前项目主线已经收敛到 `ai-agent-test-platform`，其他多个仓库分别承担过工作流验证、平台层封装、LangGraph 封装、部署研究、历史归档和配套基础设施等角色。

## 项目总览

### 1. 当前主线项目

| 仓库 | 定位 | 说明 |
|---|---|---|
| [ai-agent-test-platform](https://github.com/ljxpython/ai-agent-test-platform) | AI 测试平台主项目 | 当前核心项目，承接平台能力、Agent 场景验证和工程集成 |
| [codex_opencode](https://github.com/ljxpython/codex_opencode) | 工作流实验项目 | 用于模仿 opencode 工作流，沉淀 AI Coding / Agent 协作方式 |

### 2. 已并入主线的拆分项目

这些仓库本身都做过独立探索，后续能力逐步并入 [`ai-agent-test-platform`](https://github.com/ljxpython/ai-agent-test-platform)：

| 仓库 | 角色 | 当前状态 |
|---|---|---|
| [langgraph-agent-studio](https://github.com/ljxpython/langgraph-agent-studio) | LangGraph 架构封装 | 已集成到 `ai-agent-test-platform` |
| [agent_server_try](https://github.com/ljxpython/agent_server_try) | AI 测试平台的平台层封装 | 已集成到 `ai-agent-test-platform` |
| [my_research_langgraph](https://github.com/ljxpython/my_research_langgraph) | LangGraph 企业级部署方案探索 | 已集成到 `ai-agent-test-platform` |
| [AITestLab-archive](https://github.com/ljxpython/AITestLab-archive) | 历史版本归档 | 用于保留早期平台演进记录 |

### 3. 配套基础设施 / 工具项目

| 仓库 | 定位 | 说明 |
|---|---|---|
| [project-file-platform](https://github.com/ljxpython/project-file-platform) | 小型文件存储服务 | 为平台或实验场景提供文件管理能力 |
| [infra-provisioning-hub](https://github.com/ljxpython/infra-provisioning-hub) | AI 辅助环境配置仓库 | 用于部署、环境初始化和基础设施配置实践 |

### 4. 学习笔记与技术沉淀

| 仓库 | 主题 | 说明 |
|---|---|---|
| [langgraph-sdk-teach](https://github.com/ljxpython/langgraph-sdk-teach) | LangGraph SDK 学习笔记 | 聚焦 SDK 侧能力理解和使用方式整理 |
| [langchain_teach](https://github.com/ljxpython/langchain_teach) | LangChain / LangGraph 学习笔记 | 偏基础学习和概念梳理 |

### 5. 其他非 AI 项目

这些项目不直接属于当前 AI 主线，但能反映测试开发、平台建设和工程能力的基础积累：

| 仓库 | 定位 |
|---|---|
| [locust_framework](https://github.com/ljxpython/locust_framework) | 基础压测框架 |
| [pytest_framework](https://github.com/ljxpython/pytest_framework) | 自动化测试框架 |
| [test_platform](https://github.com/ljxpython/test_platform) | 测试开发平台（前端 TS） |
| [flask_platform_srv](https://github.com/ljxpython/flask_platform_srv) | 测试平台后端 |
| [test_engineer](https://github.com/ljxpython/test_engineer) | 测试开发知识总结 |

## 项目关系图

可以把这批仓库理解成一条逐步收敛的演进链路：

1. 先有平台和测试开发方向的长期积累，形成非 AI 项目的工程基础。
2. 进入 AI 方向后，围绕平台能力、LangGraph 封装、部署方案、文件服务等做了多仓库探索。
3. 再把验证过的能力逐步收敛回 `ai-agent-test-platform`，形成当前主线项目。
4. 同时保留 `AITestLab-archive` 作为历史归档，保留 `codex_opencode` 作为 AI Coding 工作流实验分支。

如果只想快速理解当前版图，可以按下面这个顺序看：

- 主线平台：`ai-agent-test-platform`
- 架构来源：`langgraph-agent-studio`、`agent_server_try`、`my_research_langgraph`
- 配套能力：`project-file-platform`、`infra-provisioning-hub`
- 学习沉淀：`langgraph-sdk-teach`、`langchain_teach`
- 历史参考：`AITestLab-archive`

## 本地实操记录

除了外部仓库导航，这个作品集仓库还保存了围绕 `ai-agent-test-platform` 的本地实操记录，重点不是重复贴源码，而是沉淀可复用的方法和过程。

索引入口见 [`my_work_record/README.md`](./my_work_record/README.md)。

| 日期 | 主题 | 内容简介 | 文档 |
|---|---|---|---|
| 2026-03-23 | AI 辅助部署环境 | 用两轮对话完成环境部署，并整理验证入口 | [`20260323_deployment_environment.md`](./my_work_record/20260323_deployment_environment.md) |
| 2026-03-14 | 需求分析多智能体 | 从需求文档解析、评审到落库，梳理复杂场景的设计思路 | [`20260314_requirement_agent_rd.md`](./my_work_record/20260314_requirement_agent_rd.md) |
| 2026-03-12 | Text-to-SQL 智能体 | 记录从阅读代码、设计方案到前后端接入的完整实践 | [`20260312_texttosql_rd.md`](./my_work_record/20260312_texttosql_rd.md) |

## 推荐阅读顺序

### 如果你想看当前主线项目

1. 先看 `ai-agent-test-platform`，这是当前能力最集中的主仓库。
2. 再回看 `langgraph-agent-studio`、`agent_server_try`、`my_research_langgraph`，理解这些能力原本是怎么拆出来的。
3. 最后补 `AITestLab-archive`，看历史版本和演进路径。

### 如果你想看方法论和实操过程

1. 先从 [`my_work_record/README.md`](./my_work_record/README.md) 进入。
2. 再看部署文档，先把环境和验证入口跑通。
3. 然后看 Text-to-SQL 和需求分析多智能体两个案例，理解从单智能体到复杂场景的设计方法。

## 仓库定位说明

- 外部项目仓库负责承载实际业务代码与完整工程结构。
- 当前作品集仓库负责承载项目地图、演进说明、实操记录和复盘总结。
- 如果后续还有新的实验仓库，建议继续按“是否并入主线”这个维度补充到本 README，避免项目列表越堆越散。
