# 测试用例生成闭环优化：从 Skills 编排到平台工作区落地

> 主题：测试用例生成链路从运行时设计、落库，到平台工作区接入的完整优化记录
> 日期：2026-03-30 ~ 2026-04-01

## 快速导航

- 返回实操索引：[`README.md`](./README.md)
- 返回仓库首页：[`../README.md`](../README.md)
- 项目源码：<https://github.com/ljxpython/ai-agent-test-platform>
- 先看开发范式：[`20260325_project_development_paradigm.md`](./20260325_project_development_paradigm.md)
- 先看复杂场景起点：[`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)
- 先看最小智能体案例：[`20260312_texttosql_rd.md`](./20260312_texttosql_rd.md)

## 这篇文章在讲什么

前一篇 [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md) 主要解决的是“需求分析多智能体怎么拆”。这篇记录继续往前推，核心不是重新讲一遍多智能体概念，而是把测试用例生成这条链路继续收口：

1. 从上一版多 Agent 协作，逐步调整成更容易维护的 Skills 编排。
2. 把 `runtime-service` 里的 `test_case_service` 做成更清晰的最小闭环。
3. 把结果通过 `interaction-data-service` 做持久化承接。
4. 再把 `platform-api` 和 `platform-web` 一起补齐，真正形成可用的平台入口。
5. 用真实 PDF、真实前后端链路去验证，而不是停留在 mock 和对话演示。

如果把前几篇文档理解成“先把项目跑起来、先把单点能力做出来”，那这篇更像“怎么把一个复杂场景真正接成平台能力”。

## 这次优化最终落了什么

这轮讨论和开发，最后收敛出来的是一条比较清楚的分层边界：

- `runtime-service` 负责 `test_case_agent` 的执行、多模态解析、用例生成和结果持久化编排。
- `interaction-data-service` 负责承接 `documents` 和 `test_cases` 这类结果域数据。
- `platform-api` 负责对平台前端暴露 testcase 专属 management facade，而不是让前端直连底层服务。
- `platform-web` 负责提供 `Testcase` 工作区入口，包括 AI 对话生成、用例管理、PDF 解析三个页面。

对应的一期目标，也从一开始的“先做管理页”不断收敛成下面这套更稳的闭环：

- 对话页复用现有 chat runtime，固定接 `test_case_agent`。
- 管理页一期先以“读”为主，但接口 contract 可以提前把 CRUD 定死。
- PDF 页面重点展示解析结果本身，而不是把它藏在对话过程里。
- 验证时必须走真实文件、真实服务、真实页面，不走 mock。

## 这篇文档怎么读

这次我把原始对话保留了，因为里面不只是聊天记录，还包含了不少关键决策是怎么一步步收敛出来的。为了不让整篇文档重新变成纯流水账，阅读方式建议这样：

1. 先看上面的总结，先知道这篇到底解决什么问题。
2. 再看下面的“关键结论”和“效果展示”，先抓住这轮优化真正落地了什么。
3. 最后再看“原始对话记录”，那里保留了从运行时、落库、平台侧、前端走查到二期方向的完整讨论脉络。

## 关键结论

### 1. 运行时侧的优化方向更明确了

- 测试用例生成不再继续沿着历史 `usecase_workflow_agent` 的复杂思路扩，而是收敛到 `test_case_service`。
- 图片和 PDF 解析仍然通过中间件承接，避免把多模态处理硬塞进主流程代码。
- 持久化通过专属 skill 和 tool 去调用 `interaction-data-service`，而不是在平台层散落业务逻辑。

### 2. 结果域服务边界被单独拎清了

- `interaction-data-service` 继续承担 `documents` 和 `test_cases` 的存储。
- `project_id` 等上下文后续由 `runtime.config` / `runtime.state` 透传；本地阶段没有拿到时按 `default` 兜底。
- 平台读链路通过聚合接口和 facade 做收口，避免前端直接感知底层字段和服务拆分。

### 3. 平台入口不是简单加个页面，而是做成独立工作区

- `platform-web` 最终收敛为新的 `Testcase` 一级入口。
- 页面结构继续遵守当前项目前端的工作区模式，而不是为了这个模块单独发明一套管理后台壳子。
- 模块内重点分成三个页面：AI 对话生成、用例管理、PDF 解析。

### 4. 二期方向也提前定下来了

当前文档解析结果的可见性，最初是“正式保存 testcase 批次后一起落库可见”。后续讨论里又往前推进了一步，二期要改成：

- 文档解析完成后，在结果传给主 Agent 前就先做确定性落库。
- 去重逻辑优先放在服务内控制，不让中间件耦合数据库写入。
- 平台侧文档说明要同步更新，避免文档和真实行为打架。

## 效果展示

下面这几组图，建议先快速扫一遍，能更直观看到这轮优化从“生成”到“平台接入”的变化。

### 1. Skills 编排下的生成与评审

使用 deepagent 自动规划任务并查找可用 skills：

![image-20260330164511997](./assets/image-20260330164511997.png)

完成任务后会自动勾选执行进度：

![image-20260330164430530](./assets/image-20260330164430530.png)

生成完用例后，再使用评审 skill 继续检查输出质量：

![image-20260330164849333](./assets/image-20260330164849333.png)

![image-20260330171722380](./assets/image-20260330171722380.png)

### 2. 落库与真实验证

围绕持久化链路继续补设计、补验证，并坚持用真实业务场景联调：

![image-20260330174714573](./assets/image-20260330174714573.png)

![image-20260330175710298](./assets/image-20260330175710298.png)

![image-20260330194018788](./assets/image-20260330194018788.png)

![image-20260330194455995](./assets/image-20260330194455995.png)

### 3. 平台工作区效果

前端走查阶段，可以看到 Testcase 工作区已经形成完整入口：

![image-20260330205023819](./assets/image-20260330205023819.png)

发现展示问题后继续修正：

![image-20260330211949965](./assets/image-20260330211949965.png)

修复后继续推进页面和交互收口：

![image-20260330215417402](./assets/image-20260330215417402.png)

### 4. 最终联调效果

前端入口：

![image-20260331201450514](./assets/image-20260331201450514.png)

真实对话过程：

![image-20260401111713115](./assets/image-20260401111713115.png)

生成后的用例与文档展示：

![image-20260401115008123](./assets/image-20260401115008123.png)

![image-20260401115048237](./assets/image-20260401115048237.png)

![image-20260401115508409](./assets/image-20260401115508409.png)

![image-20260401115141424](./assets/image-20260401115141424.png)

## 这次实践沉淀了什么

- 复杂场景要先把职责边界收敛清楚，再谈“页面怎么做”和“接口怎么补”。
- Skills 编排不等于随便堆流程，还是要先把运行时、落库、平台三层边界想明白。
- 平台侧不要直接透出底层服务细节，前端越少感知底层拆分，后面越稳。
- 真正能暴露问题的，还是用真实 PDF、真实模型、真实服务链路联调，而不是 mock。
- 原始对话本身是有价值的，因为很多关键决策不是一下子想出来的，而是一轮轮讨论、质疑、收敛出来的。

## 相关阅读

- 如果你想先搞懂当前项目为什么这么拆，先看 [`20260325_project_development_paradigm.md`](./20260325_project_development_paradigm.md)。
- 如果你想看复杂场景最初怎么拆多智能体，先看 [`20260314_requirement_agent_rd.md`](./20260314_requirement_agent_rd.md)。
- 如果你想回到记录总索引，入口在 [`README.md`](./README.md)。

## 原始对话记录

下面完整保留这轮过程中的原始对话和中间记录，方便后续回看每个决策是怎么收敛出来的。

### 原始记录：用例生成优化及落库



上一版本的用例生成是基于多agent协作的方式开发的，今天我们使用skills编排的方式，文档可以阅读`apps/runtime-service/runtime_service/services/test_case_service`下的`design.md`

开发的范式还是之前和大家说的，了解好我的框架每一部分是什么，然后让AI来阅读代码





## 效果展示



使用deepagent，会自动将任务规划，然后查找可用的skills

![image-20260330164511997](./assets/image-20260330164511997.png)





完成相关任务后会勾选完成

![image-20260330164430530](./assets/image-20260330164430530.png)







生成完用例后，使用评审skill评审用例

![image-20260330164849333](./assets/image-20260330164849333.png)







![image-20260330171722380](./assets/image-20260330171722380.png)





## 落库开发



```
你不用参考usecase_workflow_agent 中的思路, 我们做的简单一点,也会设计tools,然后调用
  apps/interaction-data-service ,你也了解下apps/interaction-data-service的开发范式,看
  下是否有问题,我感觉增加一个持久化的skill,他会调用tools,这个tools会调用interaction-
  data-service 暴露的接口服务即可  我们所有的开发也都在apps/runtime-service/
  runtime_service/services/test_case_service进行
```





<img src="./assets/image-20260330174714573.png" alt="image-20260330174714573" style="zoom:50%;" />

和AI进一步探讨

```
我认可,  apps/interaction-data-service 你删除原来已有的业务接口,未来apps/runtime-service/runtime_service/services/usecase_workflow_agent 下线处理, interaction-data-service的设计范式是我记得是每个服务一个专属的接口后缀,不知道是不是这样
 
  project_id 这些参数上线后肯定由runtime.config / runtime.state 读取透传过来,现在我们接受不到我们都按照default处理
  
 设计的越简单越稳定越好
```



![image-20260330175710298](./assets/image-20260330175710298.png)

```
你的方案我赞同
另外，一个本地 HTTP client 这部分看下要不要抽象出来？未来我们其他服务设计也会用到
上面的设计方案也写入到文档中

最后验证的时候，基于真实的业务场景和测试方式来验证，不要进行mock

```



<img src="./assets/image-20260330194018788.png" alt="image-20260330194018788" style="zoom:50%;" />

手动验证这部分没有问题

<img src="./assets/image-20260330194455995.png" alt="image-20260330194455995" style="zoom:50%;" />



## 平台侧开发

```
我们接下来应该完成apps/platform-api 和apps/platform-web ,我们先确认要怎么做然后在开始写代码, 前端我想增加一个testcase生成的一级标题栏,下面有AI对话生成用例的界面和AI管理的界面,对话的界面参考SQL_agent的那个界面，你认为这块应该如何开发
```





继续对话

```
Testcase 做成一个新的一级工作区入口 同意
我还想把PDF解析的内容也加一个页面展示，这部分也规划一下
SQL Agent 那样直接可用，但目标固定到 test_case_agent 这一点也同意
再加一个轻量 header slot，放三块信息 同意
AI 管理页 ->应该是用例管理界面
后端加一组 testcase 专属管理接口

你在回答我一个问题，前端没有像管理平台那样的一级标题栏二级标题栏吗？那是怎么样的一个形式，我认为正常的平台应该都有这些能力
上面是我的一些想法，我们继续明确前后端方案
```









agent的回复

```
方案我建议走“最小闭环、先跑通再扩展”。

  推荐方案
  把 Testcase 做成一个新的一级工作区入口，下面只有两个二级页：

  1. AI 对话生成
  2. AI 管理

  不要一上来做“测试用例列表、结果检索、统计大盘”这些杂东西。先把这两条链打通。

  ———

  一、整体信息架构

  前端路由建议：

  - /workspace/testcase
      - 重定向到 /workspace/testcase/generate
  - /workspace/testcase/generate
      - AI 对话生成用例页
  - /workspace/testcase/manage
      - AI 管理页

  导航结构建议：

  - 顶部一级导航新增 Testcase
  - Testcase 页面内部再做一层二级 tabs，风格直接复用 Runtime 那套

  参考现有实现：

  - SQL Agent 页面基座：apps/platform-web/src/app/workspace/sql-agent/page.tsx
  - Chat 模板基座：apps/platform-web/src/components/chat-template/base-chat-
    template.tsx
  - 二级导航模式：apps/platform-web/src/app/workspace/runtime/layout.tsx
  - 二级导航组件：apps/platform-web/src/components/platform/runtime-sections-nav.tsx
  - 一级导航入口：apps/platform-web/src/components/platform/workspace-shell.tsx

  ———

  二、前端怎么做

  1. AI 对话生成页
  目标：像 SQL Agent 那样直接可用，但目标固定到 test_case_agent。

  建议实现：

  - 页面直接复用 BaseChatTemplate
  - 默认 target 逻辑：
      - 优先使用“当前项目下已配置的 testcase assistant”
      - 如果项目还没配 assistant，就 fallback 到 graph：test_case_agent
  - feature flags：
      - allowAssistantSwitch=false
      - allowApiUrlEdit=false
      - allowRunOptions=true
      - showHistory=true
      - showArtifacts=true
      - showContextBar=true

  我建议再加一个轻量 header slot，放三块信息：

  - 当前项目
  - 当前运行模式
      - Managed Assistant
      - 或 Graph Fallback
  - 快捷提示
      - “支持上传 PDF / 图片 / 文本需求”
      - “默认最小交付并持久化”

  这页不需要自己重写 chat runtime。继续吃现有 thread/stream/provider 体系，别作死复制一
  套。

  2. AI 管理页
  这里我不建议做成“多 assistant 列表管理页”。

  最简单稳定的做法是：

  - 每个 project 只维护一个 testcase assistant
  - 页面是“单对象管理页”，不是列表页

  页面内容建议：

  - 当前 assistant 状态卡片
      - 是否已配置
      - name
      - langgraph_assistant_id
      - graph_id 固定为 test_case_agent
      - sync 状态
      - 最近同步时间
  - 配置表单
      - 名称
      - 描述
      - runtime model
      - enable tools
      - config
      - context
      - metadata
  - 操作按钮
      - 创建/保存
      - 重新同步
      - 删除
      - 打开对话页

  这样前端交互最稳，不会变成一个 assistant CMS。

  ———

  三、后端怎么做

  platform-api 我建议加一组 testcase 专属管理接口，但底层继续复用现有 assistant 管理模
  型，不新建表。

  现有 assistant 管理范式参考：

  - apps/platform-api/app/api/management/assistants.py
  - apps/platform-web/src/lib/management-api/assistants.ts

  推荐新增接口

  - GET /_management/projects/{project_id}/testcase-assistant
  - PUT /_management/projects/{project_id}/testcase-assistant
  - POST /_management/projects/{project_id}/testcase-assistant/resync
  - DELETE /_management/projects/{project_id}/testcase-assistant

  设计原则：

  - graph_id 后端固定为 test_case_agent
  - 前端不传 graph_id
  - 继续复用现有 agents + assistant_profile 模型
  - 只是在 management 层包一层 testcase 专属 facade

  为什么不直接复用 /assistants 通用页
  因为前端如果直接拿 /assistants?graph_id=test_case_agent 去拼，会把业务规则散到页面
  层，后面很快烂掉。

  让 platform-api 提供 testcase 专属 management facade，前端就只关心 testcase 业务，不
  关心底层 assistant 通用协议。

  ———

  四、数据边界

  这期我建议明确边界：

  - platform-web/platform-api 负责：
      - testcase 专属入口
      - testcase chat
      - testcase assistant 管理
  - runtime-service 负责：
      - test_case_agent 执行
      - PDF 解析
      - 持久化到 interaction-data-service
  - interaction-data-service 负责：
      - documents
      - test-cases

  也就是说，这一阶段先不在 platform-web 做“测试用例结果管理页”。
  那是下一期，不要现在一起搅。

  ———

  五、页面行为建议

  Generate 页

  - 进入页面时：
      - 读取 projectId
      - 请求 /_management/projects/{projectId}/testcase-assistant
      - 如果存在：
          - 用 assistant 模式打开 chat
      - 不存在：
          - fallback 到 graph=test_case_agent
  - 页面提示：
      - 如果处于 fallback 模式，顶部给一个提示：
          - “当前未配置项目级 Testcase AI，正在使用默认 graph 运行”

  Manage 页

  - 如果项目未选中：
      - 直接空状态，提示先选 project
  - 如果项目已选中：
      - 加载当前 testcase assistant
      - 不存在就显示创建表单
      - 存在就显示详情 + 编辑表单

  ———

  六、开发顺序

  我建议按这个顺序干：

  1. platform-api
      - 先补 testcase 专属 management 接口
      - 复用 assistants 现有逻辑
  2. platform-web
      - 增加一级导航 Testcase
      - 增加 /workspace/testcase layout + 二级 tabs
  3. platform-web
      - 先做 generate 页
      - 直接复用 BaseChatTemplate
  4. platform-web
      - 再做 manage 页
      - 单 assistant 管理，不做列表
  5. 联调
      - project 切换
      - fallback graph
      - managed assistant
      - open chat / save / resync / delete

  ———

  七、为什么我推荐这个方案

  因为它最简单，也最稳：

  - 不重写 chat runtime
  - 不新造 assistant 数据模型
  - 不把 testcase 结果浏览一起塞进来
  - 不让前端散落 graph_id 业务规则
  - 先把“入口 + 对话 + 管理”闭环打通

  ———

  八、我建议的最终落地形态

  前端：

  - 一级导航：Testcase
  - 二级：
      - AI 对话生成
      - AI 管理

  后端：

  - testcase 专属 management facade
  - 底层继续复用 assistant 表和 runtime assistant 机制

  运行逻辑：

  - 管理页配置项目级 testcase assistant
  - 对话页优先绑定该 assistant
  - 没配置就 fallback 到 test_case_agent graph

  这个方案我认可，可以直接开始做。

  如果你确认，我下一步就先拆：

  1. platform-api 路由和 schema
  2. platform-web 的 Testcase layout + generate 页
  3. platform-web 的 manage 页


› Testcase 做成一个新的一级工作区入口 同意
  我还想把PDF解析的内容也加一个页面展示，这部分也规划一下
  SQL Agent 那样直接可用，但目标固定到 test_case_agent 这一点也同意
  再加一个轻量 header slot，放三块信息 同意
  AI 管理页 ->应该是用例管理界面
  后端加一组 testcase 专属管理接口

  你在回答我一个问题，前端没有像管理平台那样的一级标题栏二级标题栏吗？那是怎么样的一个
  形式，我认为正常的平台应该都有这些能力
  上面是我的一些想法，我们继续明确前后端方案


• 不是没有，是现在这套前端已经有“一级导航 + 二级导航”的能力，只是形态不是传统后台那种左
  侧树菜单。

  当前前端的真实形态

  - 一级导航：工作区顶部横向导航，在 apps/platform-web/src/components/platform/
    workspace-shell.tsx
  - 二级导航：某个模块自己的页签导航，比如 Runtime 在：
      - apps/platform-web/src/app/workspace/runtime/layout.tsx
      - apps/platform-web/src/components/platform/runtime-sections-nav.tsx

  所以它不是“没有能力”，而是当前采用的是：

  - 顶部一级工作区导航
  - 页面内容区再放二级 tabs

  这也是一种正常的平台形态。
  如果你想更像管理平台，可以后面再统一抽一个“模块 layout + 二级 tabs”基座，但我不建议现
  在为了 Testcase 去重做全站导航。

  ———

  我建议的 Testcase 最终形态

  一级入口：

  - Testcase

  二级页签：

  1. AI 对话生成
  2. 用例管理
  3. PDF 解析

  我建议直接做三页，不要把 PDF 解析塞进对话页右侧角落里糊弄。你既然要“页面展示”，那就正
  经给它一个二级页。

  ———

  一、前端方案

  ### 1. 一级入口

  在 apps/platform-web/src/components/platform/workspace-shell.tsx 增加：

  - /workspace/testcase

  ### 2. Testcase 模块 layout

  新增：

  - src/app/workspace/testcase/layout.tsx
  - src/components/platform/testcase-sections-nav.tsx

  结构直接复用 Runtime 那套：

  - 页面标题：Testcase
  - 副标题：说明这是“测试用例生成、结果管理、PDF 解析查看”模块
  - 二级 tabs：
      - /workspace/testcase/generate
      - /workspace/testcase/cases
      - /workspace/testcase/documents

  ### 3. AI 对话生成页

  新增：

  - src/app/workspace/testcase/generate/page.tsx

  实现方式：

  - 直接复用 apps/platform-web/src/components/chat-template/base-chat-template.tsx
  - 固定目标：
      - targetType: "graph"
      - graphId: "test_case_agent"
  - 这点完全照着 apps/platform-web/src/app/workspace/sql-agent/page.tsx 的范式做

  header slot 放三块信息：

  1. 当前项目
  2. 当前目标图：test_case_agent
  3. 数据落点说明：生成结果会写入 documents + test_cases

  这页不要自己重造 chat runtime，继续复用现有 provider 体系。

  ### 4. 用例管理页

  新增：

  - src/app/workspace/testcase/cases/page.tsx
  - src/lib/management-api/testcase.ts

  页面目标：

  - 查看项目下已经生成并落库的测试用例
  - 支持按 batch_id / title / module / priority / created_at 筛选
  - 列表 + 详情抽屉

  我建议第一期先做“读”为主：

  - 列表
  - 详情
  - 按批次筛选
  - 从某条用例跳到关联 PDF 文档

  先不要一上来做复杂编辑器。

  ### 5. PDF 解析页

  新增：

  - src/app/workspace/testcase/documents/page.tsx

  页面目标：

  - 展示上传 PDF 后保存下来的解析结果
  - 列表字段建议：
      - 文件名
      - batch_id
      - parse_status
      - source_kind
      - 创建时间
  - 详情页/抽屉展示：
      - summary_for_model
      - parsed_text
      - structured_data
      - provenance
      - error

  这页很关键，因为它直接回答“PDF 到底解析成了什么”。

  ———

  二、后端方案

  你现在既然把“AI 管理页”改成“用例管理界面”，那 platform-api 这轮就不要再做 testcase
  assistant 管理了。

  这轮 platform-api 应该做的是 testcase 专属管理接口，面向“数据查看与管理”，不是
  assistant 配置。

  我建议新增一组专属接口：

  前缀：

  - /_management/projects/{project_id}/testcase

  接口建议：

  1. GET /_management/projects/{project_id}/testcase/overview
     作用：

  - 返回总文档数、总用例数、最近批次、最近更新时间

  2. GET /_management/projects/{project_id}/testcase/batches
     作用：

  - 返回批次列表
  - 每个批次带：
      - batch_id
      - document_count
      - test_case_count
      - latest_created_at

  3. GET /_management/projects/{project_id}/testcase/cases
     作用：

  - 用例列表
  - 支持：
      - batch_id
      - query
      - priority
      - module_name
      - limit/offset

  4. GET /_management/projects/{project_id}/testcase/cases/{case_id}
     作用：

  - 单条用例详情

  5. GET /_management/projects/{project_id}/testcase/documents
     作用：

  - PDF/附件解析结果列表
  - 支持：
      - batch_id
      - query
      - parse_status
      - limit/offset

  6. GET /_management/projects/{project_id}/testcase/documents/{document_id}
     作用：

  - 单个 PDF 解析结果详情

  第一期我建议先到这里。
  PATCH/DELETE 可以第二期再加，不然你前端会一起膨胀。

  ———

  三、platform-api 内部实现建议

  不要让 platform-web 直接打 interaction-data-service。
  统一由 platform-api 包一层 facade。

  内部建议新增：

  - app/api/management/testcase.py
  - app/services/interaction_data_service/client.py
  - 对应 schema 文件

  调用链：

  - platform-web
    -> platform-api /_management/projects/{project_id}/testcase/*
    -> interaction-data-service /api/test-case-service/*

  这样好处很直接：

  - project scope、auth、审计都还在 platform-api
  - 前端不感知底层服务拆分
  - 后面如果 interaction-data-service 改字段，前端不用跟着炸

  ———

  四、页面与接口的对应关系

  AI 对话生成

  - 不走 management
  - 直接还是现有 chat/runtime 流
  - 固定 graph：test_case_agent

  用例管理

  - 走：
      - overview
      - batches
      - cases
      - case detail

  PDF 解析

  - 走：
      - batches
      - documents
      - document detail

  ———

  五、我对“管理平台感”的建议

  你说得对，正常平台应该有一级和二级结构。
  但这不等于一定要做左侧树菜单。

  我建议当前项目保持一致性：

  - 一级：顶部工作区导航
  - 二级：模块内部 tabs
  - 页面顶部：标题 + 描述 + 操作区

  这已经是标准平台结构了。
  现在真正缺的不是能力，而是“把这种结构在更多模块里统一用起来”。

  所以我建议顺手抽一个通用组件，类似：

  - WorkspaceSectionLayout
      - title
      - description
      - navItems
      - actions

  这样：

  - Runtime 可以复用
  - Testcase 直接用
  - 后面别的模块也能统一

  这比现在为了 testcase 去全站改导航靠谱得多。

  ———

  六、最终建议版本

  我建议这轮明确成下面这个版本：

  前端：

  - 一级入口：Testcase
  - 二级页：
      - AI 对话生成
      - 用例管理
      - PDF 解析

  后端：

  - platform-api 新增 testcase 专属 management router
  - 封装 interaction-data-service
  - 第一阶段只做读接口，不做复杂编辑

  聊天页：

  - 固定 test_case_agent
  - 复用 BaseChatTemplate
  - 增加轻量 header slot 三块信息

  这套方案简单、稳、边界清楚。

  如果你确认，我下一步就不写代码前再给你一版“精确到文件和接口名”的实施清单。
```



```
我想做成，一级导航下拉能出来二级导航的那种形式
一期就读为主吧，可以吧增删改也一起固定下来
PDF我是想展示对话中上传的文档解析后的内容，看下apps/runtime-service/runtime_service/services/test_case_service有没有实现这个功能，如果没有我们还需要实现一下，也通过apps/interaction-data-service 持久化存储一下

专属接口 的部分我也同意
```



AI回复

```
结论

  1. test_case_service 现在已经有 PDF 解析结果落库能力，但不是“上传即自动落库”，而是“最
     终调用 persist_test_case_results 后，连同正式测试用例一起落库”。
  2. platform-web 目前没有“一级导航下拉出二级导航”的全局能力，当前形态是一级横向导航 +
     某些模块页内二级导航。这个可以改，而且改动面可控。
  3. 一期可以按“读为主”推进，后端接口把 CRUD 一次性定死，前端先把读链路和页面结构做稳。

  现状判断

  - PDF 解析链路已经存在：
    apps/runtime-service/runtime_service/services/test_case_service/graph.py
    apps/runtime-service/runtime_service/middlewares/multimodal/parsing.py
    apps/runtime-service/runtime_service/services/test_case_service/tools.py
  - parsed_text、structured_data、summary_for_model 已经会写入 interaction-data-
    service：
    apps/interaction-data-service/app/api/test_case_service/documents.py
    apps/interaction-data-service/app/db/models.py
  - 但注意边界：
    现在文档持久化触发点在 persist_test_case_results，所以它是“已保存批次的 PDF 解析结
    果”，不是“聊天里刚上传完就一定能在管理页看到”。

  我建议的一期方案

  1. 一级导航做成下拉式。
  2. 新增一级入口 Testcase。
  3. Testcase 下拉三项：
     AI 对话生成
     用例管理
     PDF 解析
  4. Testcase 顶层点击默认进入 /workspace/testcase/generate。
  5. 页面内不再重复放一条明显的二级 tab 栏，避免全局下拉和页内 tab 重复打架。
  6. 页面顶部保留轻量 header slot，放三块信息：
     当前项目
     固定目标 graph = test_case_agent
     持久化范围 = documents + test_cases

  前端方案

  - 先改全局导航壳子：
    apps/platform-web/src/components/platform/workspace-shell.tsx
  - 新增模块路由：
    /workspace/testcase/generate
    /workspace/testcase/cases
    /workspace/testcase/documents
  - AI 对话生成 页直接复用：
    apps/platform-web/src/components/chat-template/base-chat-template.tsx
  - 固定目标：
    targetType = "graph"
    graphId = "test_case_agent"
  - 用例管理 页一期做：
    列表
    搜索
    分页
    详情侧栏或详情页
  - PDF 解析 页一期做：
    文档列表
    批次筛选
    文档详情
    展示 summary_for_model、parsed_text、structured_data

  后端方案

  - platform-api 新增 testcase 专属管理接口，前缀建议：
    /_management/projects/{project_id}/testcase
  - 一期接口建议固定为：
    GET /overview
    GET /batches
    GET /cases
    GET /cases/{case_id}
    POST /cases
    PATCH /cases/{case_id}
    DELETE /cases/{case_id}
    GET /documents
    GET /documents/{document_id}
  - 角色控制：
    读接口允许 admin/editor/executor
    写接口允许 admin/editor
  - platform-api 不直接让前端打 interaction-data-service，由 platform-api 包一层。
  - platform-api 现在还没有 interaction-data-service 配置，需要补：
    apps/platform-api/app/config.py
    建议新增：
    INTERACTION_DATA_SERVICE_URL
    INTERACTION_DATA_SERVICE_TOKEN
    INTERACTION_DATA_SERVICE_TIMEOUT_SECONDS

  interaction-data-service 方案

  - 现有原子接口已经够支撑 cases/documents。
  - 为了让页面更干净，建议再补两个聚合接口：
    GET /api/test-case-service/overview
    GET /api/test-case-service/batches
  - 这样 platform-api 不需要自己扫全量数据拼批次和统计，简单稳定。

  关于 PDF 页面这件事

  - 如果一期定义为“展示已保存 testcase 批次里的 PDF 解析结果”，当前后端能力已经基本够
    了，只需要把读链路接出来。
  - 如果你要的是“用户在对话里一上传 PDF，就算还没正式保存 testcase，也必须在 PDF 页面看
    到”，那当前实现还不够。
  - 这个增强我建议放二期，而且不要交给模型决定，应该做成服务内的确定性归档链路。

  CRUD 取舍

  - 用例：后端 contract 一次性固定 CRUD。
  - PDF 文档：一期保持只读，别做编辑删除。这个是系统解析产物，不是人工业务主数据。

  开发顺序

  1. 先补 interaction-data-service 的 overview/batches
  2. 再补 platform-api 的 testcase client + management router
  3. 然后改 platform-web 全局导航下拉
  4. 接着做 AI 对话生成
  5. 再做 用例管理
  6. 最后做 PDF 解析
  7. 用真实前端上传 PDF、真实模型、真实服务链路联调，不做 mock

  如果按这个口径推进，我下一步就按 interaction-data-service -> platform-api ->
  platform-web 的顺序开始写。
```



继续对话

```
你的方案我认同，先把这个方案写入到文档中，然后就可以开发了

真实前端上传 PDF 我们使用apps/runtime-service/runtime_service/test_data/接口文档.pdf

test_case_service 现在已经有 PDF 解析结果落库能力，但不是“上传即自动落库”，而是终调用 persist_test_case_results 后，连同正式测试用例一起落库”。  -> 这个我们二期优化，修改成，文档解析后，传给主agent时，就落库，二期看下这里如何实现



```



```
接下来我们将在apps/runtime-service中进行开发,你可以阅读apps/runtime-service/runtime_service/docs中的开发规范,但是你不可以阅读apps/runtime-service/runtime_service/services/usecase_workflow_agent下的代码,接下来我们要在apps/runtime-service/runtime_service/services下开发一个测试用例分析智能体服务,解析图片和PDF的方式使用中间件,位置在apps/runtime-service/runtime_service/middlewares ,先不要写代码,可以参考/Users/bytedance/Downloads/2026-03-28-testing-agent-system项目中实现,先告诉我你要怎么做,创建一个test_case_service目录,然后把我们要如何实现这个需求写入一个文档中,当我们把所有的细节讨论清楚后,规划如何实现,然后再开始编写代码,设计要符合当前项目的代码规范和范式
```



agent回复

```
我补做了最小 HTTP 探测。结果很直接：runtime-service 和 interaction-data-service 都
  在，platform-api 当前没起，platform-web 的 3000 服务还没把新 testcase 路由挂出来，返
  回的是 404，说明现有前端进程需要重启才能吃到这批新页面。

• 方案文档已经落库，Testcase 工作区一期代码也已经接上。

  已完成

  - 方案文档：
    apps/runtime-service/runtime_service/docs/11-testcase-platform-workspace-
    design.md:1
    里面写明了一期结构，以及你确认的二期“上传后立即保存 PDF 解析结果”的实现方向：apps/
    runtime-service/runtime_service/docs/11-testcase-platform-workspace-design.md:330
  - interaction-data-service 增加聚合读接口：
    apps/interaction-data-service/app/api/test_case_service/aggregates.py:15
  - interaction-data-service 增加筛选和聚合查询：
    apps/interaction-data-service/app/db/access.py:55
    apps/interaction-data-service/app/db/access.py:197
    apps/interaction-data-service/app/db/access.py:251
  - platform-api 增加 testcase 专属管理接口：
    apps/platform-api/app/api/management/testcase.py:35
  - platform-api 增加本地共享 HTTP client 和 interaction-data-service service：
    apps/platform-api/app/services/local_json_http_client.py
    apps/platform-api/app/services/interaction_data_service.py
  - platform-web 增加一级导航下拉式 Testcase 入口：
    apps/platform-web/src/components/platform/workspace-shell.tsx:33
  - platform-web 增加三个页面：
    apps/platform-web/src/app/workspace/testcase/generate/page.tsx
    apps/platform-web/src/app/workspace/testcase/cases/page.tsx:38
    apps/platform-web/src/app/workspace/testcase/documents/page.tsx:38
  - platform-web 增加 testcase 管理 API client：
    apps/platform-web/src/lib/management-api/testcase.ts

  当前实现边界

  - 一期已经支持：
      - AI 对话生成
      - 用例管理
      - PDF 解析
      - 后端 testcase 管理接口
  - 二期还没做：
      - “上传 PDF 后，在解析结果传给主 agent 前立即落库 document”
  - 二期方案已经写入文档，不会丢。

  验证结果

  - interaction-data-service 定向测试通过：
    uv run pytest tests/test_test_case_service_api.py -q
  - platform-api 编译通过：
    uv run python -m py_compile ...
  - platform-web 类型检查通过：
    pnpm exec tsc --noEmit
  - platform-web 定向 ESLint 通过：
    pnpm exec eslint ...

  本机实时探测

  - interaction-data-service 活着：8081 -> 200
  - runtime-service 活着：8123 -> 200
  - platform-api 当前没起：2024 -> connect fail
  - platform-web 当前 3000 上运行的实例对新 /workspace/testcase* 路由返回 404
    这是我基于现象的判断：当前前端进程还没吃到这批新路由，通常需要重启 platform-web

  补充说明

  - 你仓库里原有的两个未跟踪文件我没动：
      - apps/runtime-service/runtime_service/langgraph.json.bak
      - apps/runtime-service/runtime_service/test_data/接口文档.pdf

  下一步建议

  1. 启动或重启 platform-api，确认跑在 http://localhost:2024
  2. 重启 platform-web
  3. 用真实 PDF apps/runtime-service/runtime_service/test_data/接口文档.pdf 做一轮前端
     联调，我再继续帮你抓实际输出和报错定位
```





前端走查

![image-20260330205023819](./assets/image-20260330205023819.png)





```
一级导航的testcase样式和其他的不一样，修改为一致

AI对话页面的那三个小卡片信息应该放到和左侧的小卡里一起展示即可

用例管理 页面没有用例信息是因为当前项目没有生成用例导致的吗？ 

PDF解析内容这里也一样没有解析的内容

现在使用的projectId=5f419550-a3c7-49c6-9450-09154fd1bf7d，你要不直接改数据库，将已有的数据，迁移到这个project下面让我能看到


仓库里原有的两个未跟踪文件:apps/runtime-service/runtime_service/test_data/接口文档.pdf 下次一起提交，另一个我删除了
```





前端展示有问题的地方记录和AI进行对话

![image-20260330211949965](./assets/image-20260330211949965.png)



agent修复问题

![image-20260330215417402](./assets/image-20260330215417402.png)



```
增加将用例导出为Excel的功能，前后端应该如何开发
```



先保存一下代码

```
我们将开发的新功能记录到各个项目下的文档内容，之后提交代码，代码也符合docs/commit-and-changelog-guidelines.md中的开发规范
```





继续开发

```
中间件内不能有落库的耦合, 你上述的方案我认同,将方案写入到对应的文档中,就可以开始写代
  码了,测试要使用真实的数据进行测试,确保各个环节都没问题,不是平台层可以使用apps/
  runtime-service/runtime_service/tests/services_test_case_service_debug.py 等你之前写
  好的case方式 前端的如果你要使用playwright进行验证,需要我配合的地方可以停下来和我说
```





结果

```
当前状态
  apps/platform-api 和 apps/platform-web 的 testcase 工作区，其实不是“还没做”，而是“一
  期主体已经落地，剩下的是收口和二期增强”。

  已经做了什么

  - platform-api 的 testcase 管理接口已经有了：
      - apps/platform-api/app/api/management/testcase.py
  - platform-web 的一级导航和二级页面已经有了：
      - apps/platform-web/src/components/platform/workspace-shell.tsx
      - apps/platform-web/src/app/workspace/testcase/generate/page.tsx
      - apps/platform-web/src/app/workspace/testcase/cases/page.tsx
      - apps/platform-web/src/app/workspace/testcase/documents/page.tsx
  - 前端管理 API client 也已经有了：
      - apps/platform-web/src/lib/management-api/testcase.ts
  - 平台侧当前落地说明文档也有：
      - apps/platform-web/docs/testcase-workspace.md

  还没做的，主要有这几块

  1. platform-web 还缺“写操作 UI”

  - 现在 用例管理 页面是读为主，能查列表和详情，但没有新增、编辑、删除入口
  - 后端接口已经有：
      - POST/PATCH/DELETE /_management/projects/{project_id}/testcase/cases
  - 前端 client 也已经有：
      - apps/platform-web/src/lib/management-api/testcase.ts
  - 但页面层还没接这些动作：
      - apps/platform-web/src/app/workspace/testcase/cases/page.tsx

  2. platform-web 还缺“生成页和管理页的联动”

  - 现在生成页只是固定接到 test_case_agent
  - 还没有把“本次会话的 batch_id / 最新保存结果”直接联动到：
      - 用例管理页
      - PDF 解析页
  - 也没有“保存成功后跳转查看本批次”的动作
  - 相关页面：
      - apps/platform-web/src/app/workspace/testcase/generate/page.tsx

  3. platform-web 的 PDF 解析 页还缺产品层增强

  - 现在能看 summary_for_model / parsed_text / structured_data / provenance
  - 但还没有：
      - 原始 PDF 预览/下载
      - 和会话/批次的跳转联动
      - 更强的结构化展示，而不只是大段 JSON
  - 当前实现：
      - apps/platform-web/src/app/workspace/testcase/documents/page.tsx

  4. platform-api 缺 testcase 专项测试

  - 我刚扫过，apps/platform-api/tests 下没看到 testcase 专属测试
  - 也就是接口有了，但缺：
      - 权限校验测试
      - project_id 透传测试
      - interaction-data-service 异常映射测试
      - CRUD contract 测试
  - 服务实现本身在：
      - apps/platform-api/app/services/interaction_data_service.py
      - apps/platform-api/app/api/management/testcase.py

  5. platform-web 也缺 testcase 页面级测试

  - 现在页面代码在，但我没看到 testcase 专属前端测试
  - 缺的通常是：
      - management-api client 测试
      - 页面加载态/空态/错误态测试
      - 路由切换和筛选交互测试

  6. 文档需要跟上二期事实

  - runtime-service 二期现在已经是“上传即落库”
  - 但平台侧有些一期文档表述还偏“正式保存后才可见”
  - 这个要在下一轮平台开发时一起顺手校准，不然文档和行为会打架

  我建议下一步按这个顺序做

  1. 先补 platform-api 的 testcase 接口测试，先把 BFF 层站稳
  2. 再做 platform-web 的用例管理写操作 UI：新增、编辑、删除
  3. 然后做“生成页 -> 本批次管理页”的联动
  4. 最后补 PDF 解析 页的体验增强和前端测试

  如果你同意，我下一步就直接从第 1 项开始，先做 apps/platform-api 的 testcase 接口测试
  和必要收口。
```





和agent进行对话

```
我们这一轮还是讨论,关于你说的专属前端的测试，这部分内容我们暂时先不加,这个也放到后面
  再补充,现在我还想增加一个功能，就是前端测试用例，excel形式导出的功能,从前端可以下载,
  讨论一下这个功能要怎么,其他的你说的我都认可,另外在讨论一个事情，我看测试的时候用的是
  deepseek chat的模型,为啥没有使用其他的模型  你说的其他内容我也都认可,接下来我们也会
  一一完成
 
```





和agent进行对话

```
把我们这套智能体平台开发，从智能体开发到平台到前端到调试的一个开发验证测试联调提交总结成一个文档,形成我们的开发范式,就从刚才我们开发过程中进行总结，各个应用层都应该遵守什么样的开发方式，其中我认为一个比较重要的核心思想就是代码一定要简洁，先讨论好要设计什么，各个点都有哪些内容？上下游链路怎么对接？怎么沟通，还有我们测试的时候一定要使用真实的数据，不mock，这样才能更好更准确地发现问题，例如，只能抵这一层，如果不依赖前端，然后要怎么使用真实的数据进行测试和验证等等，后续开发都可以借鉴这一套流程，减少失误，提高编写代码的准确率，写入到根目录下的readme文档中
之后同样按照规范提交代码到远端
```





和agent进行对话

```
我们继续开发，先讨论，直接进入 apps/platform-web 用例管理页的表单体验细化和PDF管理的批次关联展示增强 这部分我们打算如何做
另外，这块的逻辑，如果上传PDF后，中间件是如何发现文档重复的？这块的逻辑是中间件内自己验证吗，我不希望还需要调用其他组件或者服务，如果单次对话遇到相同的PDF不需要解析
另外保存到interaction-data-service后会有去重逻辑吗？ 我预期是有去重，以我们新上传为主，覆盖掉之前的内容

```



和agent对话

```
我认可你说的，按照你说的茶城开发方案文档

另外：对于当前project，我们是怎么确定的，我看到前端没有一个地方可以确定我们使用的是哪个project的地方，这个可能以前就有的问题  当前我们用的projectId=5f419550-a3c7-49c6-9450-09154fd1bf7d
```









## 效果展示

前端：

![image-20260331201450514](./assets/image-20260331201450514.png)



对话

![image-20260401111713115](./assets/image-20260401111713115.png)



生成用例



![image-20260401115008123](./assets/image-20260401115008123.png)





![image-20260401115048237](./assets/image-20260401115048237.png)

![image-20260401115508409](./assets/image-20260401115508409.png)



![image-20260401115141424](./assets/image-20260401115141424.png)
