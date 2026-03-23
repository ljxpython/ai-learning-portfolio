# AI 辅助部署 ai-agent-test-platform：两轮对话完成环境搭建

> 主题：借助 AI 助手快速完成项目部署
> 日期：2026-03-23

## 快速导航

- 返回实操索引：[`README.md`](./README.md)
- 项目源码：<https://github.com/ljxpython/ai-agent-test-platform>
- 部署完成后看单智能体案例：[`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)
- 想看复杂场景设计：[`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)

## 这篇文章在讲什么

这篇记录的核心不是“手动部署步骤大全”，而是如何把重复、繁琐、但规则相对稳定的环境搭建工作，交给 AI 助手去执行和编排。

对第一次接触项目的人来说，从数据库、后端、前端到模型配置，全流程走下来很容易耗掉大半天；如果边看文档边摸索，甚至一天都不一定够。这里记录的是一种更高效的方式：先给 AI 明确的部署说明，再通过两轮对话把环境拉起来。

## 项目与工具

- 项目地址：<https://github.com/ljxpython/ai-agent-test-platform>
- 使用工具：Codex

你也可以换成别的 AI 工具，关键不在具体产品，而在于你有没有把部署步骤整理成 AI 能理解、能执行、能验证的工作流。

## 开始前的准备

先拉取项目：

```bash
git clone git@github.com:ljxpython/ai-agent-test-platform.git
cd ai-agent-test-platform
```

![image-20260323110610441](./assets/image-20260323110610441.png)

启动 Codex：

```bash
codex --search --dangerously-bypass-approvals-and-sandbox
```

## 第一次对话：让 AI 先根据部署说明跑一遍

第一轮我给 AI 的话非常直接：

```text
阅读 `docs/ai-deployment-assistant-instruction.md` 帮我部署环境。
```

AI 理解后，会先检查本地环境、读取项目里的部署说明，再开始执行部署流程。

![image-20260323110835964](./assets/image-20260323110835964.png)

第一轮结束后，通常已经能把大部分基础工作做掉，但它会告诉你还缺什么。

![image-20260323111103553](./assets/image-20260323111103553.png)

这里暴露出来的关键点是：还需要补真实模型配置，也就是推理模型和视觉模型的相关信息。真实密钥不要提交到仓库里，这种东西只应该在本地配置。

## 第二次对话：补模型配置

第二轮我直接把本地可用的模型配置给它。这里的重点不是模型品牌，而是你至少要提供一套推理模型和一套视觉模型。

示例格式如下：

```yaml
# 心流视觉模型
iflow_qwen3-vl-plus:
  alias: 心流 Qwen3-VL-Plus
  model_provider: openai
  model: qwen3-vl-plus
  base_url: https://apis.iflow.cn/v1/chat/completions
  api_key: sk-xxxxxxxxxxxxxxxxxxxx

# 心流 deepseek-v3 模型
iflow_deepseek-v3:
  alias: 心流 deepseek-v3
  model_provider: openai
  model: deepseek-v3
  base_url: https://apis.iflow.cn/v1/chat/completions
  api_key: sk-xxxxxxxxxxxxxxxxx
```

![image-20260323111301691](./assets/image-20260323111301691.png)

补完这部分以后，AI 会继续完成剩余部署动作。

![image-20260323111328401](./assets/image-20260323111328401.png)

## 部署完成后的验证入口

部署完成后，当时的访问地址如下：

```text
runtime-service: http://127.0.0.1:8123
platform-api:    http://127.0.0.1:2024
runtime-web:     http://127.0.0.1:3001
platform-web:    http://127.0.0.1:3002
```

## 验证 1：直连 runtime-web

先走最直接的验证路径：

```text
runtime-web: http://127.0.0.1:3001
```

打开页面后直接对话验证：

![image-20260323111452697](./assets/image-20260323111452697.png)

## 验证 2：验证平台前端

再验证平台层：

```text
platform-web: http://127.0.0.1:3002
账号：admin / admin123456
```

![image-20260323111858136](./assets/image-20260323111858136.png)

进入后：

![image-20260323111915334](./assets/image-20260323111915334.png)

对话验证：

![image-20260323112004280](./assets/image-20260323112004280.png)

## 这次实践可以复用什么

这次最大的收获不是某一条命令，而是一种方法：

- 把部署步骤沉淀成项目文档或 AI 指令。
- 让 AI 先按说明执行，再根据缺口补配置。
- 把验证入口提前整理好，避免部署完不知道怎么验。

如果这套方法稳定下来，后面很多重复性的环境搭建、初始化配置、基础验证，都可以交给 AI 先做一遍，人只需要做关键确认和结果验收。

## 注意点

- 模型配置、密钥等敏感信息不要提交到仓库。
- 文中的地址和端口是这次记录里的结果，后续以项目实际配置为准。
- 这篇文章主要记录“AI 辅助部署”的方法论，平台其他功能后面还可以继续单独展开。

## 相关阅读

- 环境起来后，可以直接继续看 [`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)。
- 如果你更关心复杂业务设计，再看 [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)。
- 总索引入口在 [`README.md`](./README.md)。


