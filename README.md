# AI 学习作品集（ai-learning-portfolio）

本仓库用于整理 AI 项目导航、实操记录与复盘笔记。

这里不存放完整业务代码，重点沉淀三类内容：

- 外部项目入口
- 本地实践记录
- 二开时可复用的提示词、流程和经验总结

## 项目 / 实践

| 名称 | 简介 | 链接 | 关联记录 | 更新 |
|---|---|---|---|---|
| aitestlab | 从 0 到 1 搭建 AI 测试平台 | https://github.com/ljxpython/aitestlab | 暂无本地记录 | ![](https://img.shields.io/github/last-commit/ljxpython/aitestlab?label=last%20commit) |
| ai-agent-test-platform | AI Agent 测试平台实践项目 | https://github.com/ljxpython/ai-agent-test-platform | [`my_work_record/README.md`](./my_work_record/README.md) | 手动维护 |

## 学习笔记

| 名称 | 主题 | 链接 | 更新 |
|---|---|---|---|
| langgraph_teach | langgraph / langchain 学习笔记 | https://github.com/ljxpython/langgraph_teach | ![](https://img.shields.io/github/last-commit/ljxpython/langgraph_teach?label=last%20commit) |

## 本地实操记录

当前新增的本地记录围绕 [`ai-agent-test-platform`](https://github.com/ljxpython/ai-agent-test-platform) 展开，主要沉淀三类内容：智能体开发、复杂场景设计、AI 辅助部署。

索引入口见 [`my_work_record/README.md`](./my_work_record/README.md)。

## 仓库关系

- 外部项目仓库负责承载实际业务代码与完整工程结构。
- 当前作品集仓库负责承载导航、实操记录、提示词和复盘总结。
- 如果你想先看方法论和实践过程，从 [`my_work_record/README.md`](./my_work_record/README.md) 进入最快。

| 日期 | 主题 | 内容简介 | 文档 |
|---|---|---|---|
| 2026-03-23 | AI 辅助部署环境 | 用两轮对话完成环境部署，并整理验证入口 | [`20260323_deployment_environment.md`](./my_work_record/20260323_deployment_environment.md) |
| 2026-03-14 | 需求分析多智能体 | 从需求文档解析、评审到落库，梳理复杂场景的设计思路 | [`20260314_requirement_agent_rd.md`](./my_work_record/20260314_requirement_agent_rd.md) |
| 2026-03-12 | Text-to-SQL 智能体 | 记录从阅读代码、设计方案到前后端接入的完整实践 | [`20260312_texttosql_rd.md`](./my_work_record/20260312_texttosql_rd.md) |

## 推荐阅读顺序

1. 先看部署文档，快速把环境拉起来。
2. 再看 Text-to-SQL 案例，理解最小可行智能体如何接入平台。
3. 最后看需求分析智能体，理解复杂场景下的多智能体设计与后续优化方向。
