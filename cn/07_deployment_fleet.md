# 07 Deployment、Studio、Fleet：运行和构建 agent（简介）

> 来源：[LangSmith Deployment](https://docs.langchain.com/langsmith/deployment) · [Deploy quickstart](https://docs.langchain.com/langsmith/deployment-quickstart) · [Studio](https://docs.langchain.com/langsmith/studio) · [Fleet](https://docs.langchain.com/langsmith/fleet) · 核实日期 2026-10-28

前面几篇讲的是「观察和评估」，跟你的 agent 跑在哪里无关。这一篇讲 LangSmith 的另一半：**帮你运行 agent**（Deployment + Studio），以及**不写代码造 agent**（Fleet）。

| 产品 | 给谁用 | 一句话 |
|------|--------|--------|
| **Deployment** | 工程师 | 托管运行 agent 的生产级运行时：持久执行、流式输出、水平扩展、版本和回滚 |
| **Studio** | 工程师 | agent 的可视化 IDE，本地或线上都能连 |
| **Fleet** | 不写代码的同事 | 从模板或用自然语言描述来造 agent，连接各种应用，在 Slack 里用 |

---

## 1. LangSmith Deployment

### 能部署什么

框架无关：

- **LangGraph / LangChain**：原生支持。
- **Google ADK**：用 `deployments-wrap-sdk` 包一层。
- **Claude Agent SDK、Strands、CrewAI、AutoGen 等**：用 Functional API 或 `deployments-wrap-sdk`。
- **Deep Agents**：用 Managed Deep Agents。

### 部署在哪

| 形态 | 控制面 | Agent Server 和数据 | 要求 |
|------|--------|---------------------|------|
| **Cloud** | LangChain 托管 | LangChain 托管（AWS / GCP） | Plus 及以上 |
| **Hybrid** | LangChain 托管 | 你自己的 VPC | — |
| **Self-hosted（带控制面）** | 你自己的 Kubernetes | 你自己的 Kubernetes | Enterprise |
| **Standalone server** | 无 | Docker / Compose / K8s，自带 PostgreSQL 和 Redis | 需要 license |

### 最快的上手路径（命令行部署）

```bash
uv tool install langgraph-cli

# 1. Create an app from the template
langgraph new my-agent --template new-langgraph-project-python
cd my-agent

# 2. Try it locally (opens Studio in the browser)
langgraph dev

# 3. Deploy to LangSmith Cloud
export LANGSMITH_API_KEY="<your-langsmith-api-key>"
langgraph deploy
```

- `langgraph deploy` 目前是 **Beta**。改完代码重新执行一次就会原地更新同名部署。
- `langgraph deploy list` / `logs` / `delete <ID>`：查看、看日志、删除部署。
- 也可以在 `langgraph dev` 打开的 Studio 里直接点 **deploy**，或者在网页上从 GitHub 仓库创建部署。

### 调用部署好的 agent

在网页 **Deployments** 里打开你的部署，复制 **API URL**：

```python
from langgraph_sdk import get_sync_client

client = get_sync_client(url="<your-deployment-url>", api_key="<your-langsmith-api-key>")

for chunk in client.runs.stream(
    None,  # threadless run
    "agent",  # assistant name defined in langgraph.json
    input={"messages": [{"role": "human", "content": "Say hello."}]},
    stream_mode="updates",
):
    print(chunk.event, chunk.data)
```

（需要 `pip install langgraph-sdk`。）

部署后的三个核心概念：**assistant**（一份配置）、**thread**（会话状态）、**run**（一次执行）。部署好的 agent 可以在运行时从 [Prompt & Context Hub](06_prompt_engineering.md) 拉 prompt 和上下文，改行为不用重新部署。

---

## 2. Studio

Studio 是专门调试 agent 的 IDE，连接本地的 `langgraph dev`，或者线上的部署（在部署详情页右上角点 **Studio**）。

主要功能：

- 可视化图结构，看执行经过了哪些节点、每一步的中间状态。
- 运行中途修改状态、**time travel**（回到之前的某一步重新执行）。
- 管理 assistant、thread、长期记忆。
- 直接把某次运行加进数据集、在 Playground 里改 prompt、对数据集跑实验。
- 一键部署到 LangSmith Cloud。

两种模式：

| 模式 | 适合 |
|------|------|
| **Graph mode** | 工程师：看全部细节（节点、中间状态、LangSmith 集成） |
| **Chat mode** | 业务人员：只测对话效果；要求 graph 的 state 包含 `MessagesState` |

部署动图（11 MB，没放进仓库）：[Deploy from Studio](https://docs.langchain.com/langsmith/deployment-quickstart)。

---

## 3. Fleet（原 Agent Builder）

> 2026-03 起 **Agent Builder 改名为 LangSmith Fleet**，已有的 agent、配置和集成都不受影响。

Fleet 是给**不写代码的人**用的 agent 平台：

- **两种起步方式**：用自然语言描述你想要的 agent，让 Fleet 自己配置（关键步骤会停下来问你）；或者从[模板](https://www.langchain.com/templates)开始改。
- **连接应用**：登录要用的服务授权给 agent，也支持远程 MCP 连接。
- **在哪用**：在 Fleet 里聊天，或者装到 Slack 里。
- **安全**：重要操作需要人工审批。
- **从代码调用**：Fleet 里造好的 agent 也可以被你的应用调用。
- **自托管**：Beta。

典型用途：起草邮件、汇总更新、整理信息这类日常重复工作。

入口：[smith.langchain.com/agents](https://smith.langchain.com/agents)，详细文档见 [Fleet](https://docs.langchain.com/langsmith/fleet)。

## 下一步

- agent 上线后的自动排障、内置 AI 助手、代码沙箱、模型网关：[08](08_engine_chat_sandbox.md)
