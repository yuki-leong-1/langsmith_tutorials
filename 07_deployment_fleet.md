# 07 Deployment, Studio, Fleet: Running and Building Agents (Overview)

> Sources: [LangSmith Deployment](https://docs.langchain.com/langsmith/deployment) · [Deploy quickstart](https://docs.langchain.com/langsmith/deployment-quickstart) · [Studio](https://docs.langchain.com/langsmith/studio) · [Fleet](https://docs.langchain.com/langsmith/fleet) · Verified 2026-10-28

The earlier chapters were about "observing and evaluating", which doesn't depend on where your agent runs. This chapter covers the other half of LangSmith: **running your agents for you** (Deployment + Studio), and **building agents without code** (Fleet).

| Product | Who it's for | In one sentence |
|---------|--------------|-----------------|
| **Deployment** | Engineers | A production-grade runtime that hosts your agents: durable execution, streaming, horizontal scaling, versioning and rollback |
| **Studio** | Engineers | A visual IDE for agents, connectable locally or to a live deployment |
| **Fleet** | Colleagues who don't write code | Build agents from templates or a natural-language description, connect various apps, and use them in Slack |

---

## 1. LangSmith Deployment

### What can be deployed

Framework-agnostic:

- **LangGraph / LangChain**: native support.
- **Google ADK**: wrap with `deployments-wrap-sdk`.
- **Claude Agent SDK, Strands, CrewAI, AutoGen, and others**: use the Functional API or `deployments-wrap-sdk`.
- **Deep Agents**: use Managed Deep Agents.

### Where it runs

| Model | Control plane | Agent Server and data | Requirement |
|-------|---------------|-----------------------|-------------|
| **Cloud** | Hosted by LangChain | Hosted by LangChain (AWS / GCP) | Plus or above |
| **Hybrid** | Hosted by LangChain | Your own VPC | — |
| **Self-hosted (with control plane)** | Your own Kubernetes | Your own Kubernetes | Enterprise |
| **Standalone server** | None | Docker / Compose / K8s, with your own PostgreSQL and Redis | License required |

### Fastest way to get started (CLI deployment)

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

- `langgraph deploy` is currently **Beta**. After changing your code, run it again to update the deployment of the same name in place.
- `langgraph deploy list` / `logs` / `delete <ID>`: list, view logs, and delete deployments.
- You can also click **deploy** directly in the Studio that `langgraph dev` opens, or create a deployment from a GitHub repository in the web UI.

### Call a deployed agent

Open your deployment under **Deployments** in the web UI and copy the **API URL**:

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

(Requires `pip install langgraph-sdk`.)

The three core concepts after deployment are **assistant** (a configuration), **thread** (conversation state), and **run** (one execution). A deployed agent can pull prompts and context from the [Prompt & Context Hub](06_prompt_engineering.md) at runtime, so you can change behavior without redeploying.

---

## 2. Studio

Studio is an IDE built for debugging agents. It connects to a local `langgraph dev` or to a live deployment (click **Studio** in the top right of the deployment detail page).

Main features:

- Visualize the graph structure and see which nodes execution passed through and the intermediate state at each step.
- Edit state mid-run and **time travel** (go back to an earlier step and re-execute).
- Manage assistants, threads, and long-term memory.
- Add a run directly to a dataset, edit prompts in the Playground, and run experiments over a dataset.
- One-click deploy to LangSmith Cloud.

Two modes:

| Mode | Best for |
|------|----------|
| **Graph mode** | Engineers: full detail (nodes, intermediate state, LangSmith integration) |
| **Chat mode** | Business users: test only the conversational behavior; requires the graph's state to include `MessagesState` |

The deployment animation (11 MB) is not included in this repo: [Deploy from Studio](https://docs.langchain.com/langsmith/deployment-quickstart).

---

## 3. Fleet (formerly Agent Builder)

> Since 2026-03, **Agent Builder has been renamed LangSmith Fleet**. Existing agents, configurations, and integrations are unaffected.

Fleet is an agent platform for **people who don't write code**:

- **Two ways to start**: describe the agent you want in natural language and let Fleet configure it (it pauses to ask you at key steps); or start from a [template](https://www.langchain.com/templates) and modify it.
- **Connect apps**: sign in to the services you want and authorize the agent; remote MCP connections are also supported.
- **Where to use it**: chat in Fleet, or install it in Slack.
- **Safety**: important actions require human approval.
- **Call from code**: agents built in Fleet can also be called by your applications.
- **Self-hosted**: Beta.

Typical uses: routine repetitive work such as drafting emails, summarizing updates, and organizing information.

Entry point: [smith.langchain.com/agents](https://smith.langchain.com/agents). For details, see [Fleet](https://docs.langchain.com/langsmith/fleet).

## Next Steps

- Automatic troubleshooting after launch, the built-in AI assistant, code sandboxes, and the model gateway: [08](08_engine_chat_sandbox.md)
