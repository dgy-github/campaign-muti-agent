# campaign-muti-agent —— 多 Agent 指挥 / 运行 / 演练框架

[English](README.md) | 简体中文

`campaign` 是一个面向工程级多 Agent 系统的可运行参考实现。它的核心观点是：**Agent 团队不应该只是几个角色提示词在聊天室里互相对话，而应该像生产系统一样被指挥、治理、演练、观测和恢复。**

这个框架把多 Agent 编排落成一套具体系统：

- **指挥层**：`ExecutionOrder` 和 `Task` 定义目标、约束、技能、预算和验收标准。
- **运行层**：Coordinator / Executor / Retriever / Reviewer / Reserve 通过显式 A2A 消息协作。
- **治理层**：每个任务和每次 LLM 调用都可以经过 `PolicyGate`，检查预算、权限、数据出域和注入风险。
- **事件源状态**：`EventLog` 只追加不覆盖；运行态通过事件回放派生，并按 `run_id` 隔离。
- **演练层**：`EvalHarness` 和 `ChaosDrill` 把上线准备度变成可衡量门禁，而不是凭感觉判断。
- **A2A 传输**：支持进程内、HTTP JSON-RPC、SSE 流式、能力发现、幂等重试和远程 Agent 代理。

它刻意保持小而可读，但足够覆盖严肃 Agent 工程话题：信任边界、HITL、分布式协作、成本控制、RAG 记忆、链路追踪、熔断，以及可测试的失败模式。

## 架构页面

在线 HTML 架构页：

- https://dgy-github.github.io/campaign-muti-agent/

本地文件：

```text
campaign-architecture.html
```

这页把框架解释成一个“战役指挥中心”：指挥层 -> 运行时 -> A2A 协议 -> 治理 -> 演练。

## 它和普通多 Agent Demo 有什么不同

大多数 multi-agent demo 停在角色提示词。`campaign` 关注的是让 Agent 系统能活下来的部分：

- **状态不是猜出来的**：任务状态来自事件回放，而不是可变全局状态。
- **裁判和执行者分离**：`Reviewer` 独立验收执行结果，不让 worker 自评自夸。
- **本地和远程共用一套协议**：本地 worker 和远程 worker 都接收 `Message` 信封。
- **人工介入可恢复**：input-required 和 approval 状态可以跨进程 resume。
- **系统可以响亮地失败**：混沌演练、熔断、制动和治理违规都是一等事件。
- **测试完全离线**：测试使用 mock provider 和本地 store，不需要 API key，也不需要网络。

## 快速开始

```powershell
python -m pip install -e ".[dev]"
python -m pytest -q
python -m campaign
python -m campaign.examples.capstone_demo
python -m campaign.examples.distributed_demo
```

预期测试结果：

```text
172 passed
```

## 仓库地图

| 路径 | 职责 |
| --- | --- |
| `campaign/core/events.py` | 追加式事件日志、回放、订阅、回滚/截断。 |
| `campaign/core/state.py` | 从事件派生运行态，并按 `run_id` 隔离。 |
| `campaign/core/models.py` | `ExecutionOrder`、`Task`、`AgentSpec` 和校验契约。 |
| `campaign/protocol.py` | A2A `Message`、`Part`、`AgentCard` 和任务状态。 |
| `campaign/transport.py` | 进程内与 HTTP JSON-RPC 传输、SSE、发现、远程代理。 |
| `campaign/roles/` | Coordinator、Executor、Retriever、Reviewer、Reserve。 |
| `campaign/routing/` | 技能注册表和 ROI 路由器。 |
| `campaign/governance/` | Policy-as-code、Governor、PolicyGate。 |
| `campaign/resilience/` | 熔断器、检查点回滚、动员减员。 |
| `campaign/observability/` | Tracer span、meter、score。 |
| `campaign/llm/` | OpenAI-compatible chat / embedding adapter，支持 mock。 |
| `campaign/memory.py` | Scratchpad 和 session memory。 |
| `campaign/knowledge.py` | SQLite TF-IDF 知识库，预留神经 embedding 接口。 |
| `campaign/eval/` | Eval harness 和 chaos drill。 |
| `campaign/app/runtime.py` | Runtime 装配、run/resume/discover API。 |
| `docs/` | 方法论、A2A、扩展、RAG、运行手册、坑点和工作日志。 |

## 设计边界

- 分布式路径是可运行 MVP：HTTP transport + 共享 SQLite event store + remote proxy。生产级分布式一致性仍需要共享预算、health、breaker 和 leader coordination。
- 长期知识默认是词法 TF-IDF，不是神经 embedding；保留 adapter seam 是为了方便后续接真实 embedding。
- 外部内容一律当作不可信输入。检索数据进入 agent 上下文前可以标记为 `untrusted`。

## 为什么做这个项目

`campaign` 是给想理解 Agent 应用怎样变成真实系统的人看的：

- Agent 如何在没有隐藏共享状态的情况下协作？
- 任务暂停等待人工审批后，系统怎样恢复？
- 预算和权限检查怎样避免变成装饰？
- 本地 worker 和远程 worker 怎样共享同一协议？
- 失败怎样沉淀为演练数据，而不是线上事故？

这就是框架的核心：**不是更多 Agent，而是更好的指挥系统。**

## 仓库名说明

GitHub 仓库名使用 `campaign-muti-agent`，以匹配发布地址。Python 可导入包名仍然是 `campaign`。
