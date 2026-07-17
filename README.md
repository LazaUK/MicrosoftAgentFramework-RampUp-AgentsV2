# Microsoft Agent Framework: Jupyter Notebooks for Ramp-Up Learning Process

**Microsoft Agent Framework** is an open-source development kit for building _AI agents_ and _multi-agent workflows_. It combines and extends constructs and concepts from two other Microsoft agentic frameworks: _Semantic Kernel_ and _AutoGen_.

This repo provides Jupyter notebooks to ramp up (Level 200: Agents V2) and build practical skills of using Microsoft Agent Framework's _Python SDK_.

> [!WARNING]
> To successfully run these notebooks, you must have an **Microsoft Foundry** project and **AI model deployment** in Azure. Please ensure you have the following environment variables set up in your system:
> | Environment Variable             | Description                                                                     |
> | -------------------------------- | ------------------------------------------------------------------------------- |
> | `AZURE_FOUNDRY_PROJECT_ENDPOINT` | The endpoint URL for your Microsoft Foundry project in Azure.                   |
> | `AZURE_FOUNDRY_GPT_MODEL`        | The name of the model deployment to be used by the agent, e.g., _gpt-4.1-mini_. |

## 📑 Table of Contents
- [Notebook 0: Quick Start](#notebook-0-quick-start)
- [Notebook 1: Agents - Tools](#notebook-1-agents---tools)
- [Notebook 2: Agents - Middleware](#notebook-2-agents---middleware)
- [Notebook 3: Agents - Observability](#notebook-3-agents---observability)
- [Notebook 4: Agents - Memory](#notebook-4-agents---memory)
- [Notebook 5: Prompt Agents (Foundry Deployment)](#notebook-5-prompt-agents-foundry-deployment)

## Notebook 0: Quick Start
This notebook, `AF_00_GettingStarted_QuickStart.ipynb`, provides a general intro to the Agent Framework. It covers the core steps required to get a basic agent running.

- Configure your environment and set up necessary imports using the `agent_framework` package:

``` Python
# Import required packages
import os
import asyncio
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import DefaultAzureCredential
```

- Create an **AI Client** using _FoundryChatClient_ to connect to your Microsoft Foundry project:

``` Python
client = FoundryChatClient(
    project_endpoint = PROJECT_ENDPOINT,
    model = MODEL_DEPLOYMENT,
    credential = DefaultAzureCredential(),
)
```

- Instantiate an **AI Agent** using either the standard *Agent* class or the client's *.as_agent()* helper function:

``` Python
# standard method
agent = Agent(
    client = client,
    name = "haiku-poet-agent",
    instructions = "You are a haiku poet. Respond to user queries with a haiku."
)

# alternative factory method
agent_alt = client.as_agent(
    name = "alternative-haiku-poet-agent",
    instructions = "You are a haiku poet. Respond to user queries with a haiku."
)
```

- Run the agent using both **non-streaming** and **streaming** *agent.run(prompt, stream=True)* methods to test its response capabilities:

``` Python
response = await agent.run(prompt)
...
async for streaming_update in agent_alt.run(prompt_alt, stream=True):
    if streaming_update.text:
        print(streaming_update.text, end="", flush=True)
```

The output creates a _Haiku Poet Agent_ that responds to user queries with a haiku:

``` JSON
User: What is life?

Agent: Life flows like a stream,  
Moments blend in endless dance -  
Dreams wake with the dawn.
```

## Notebook 1: Agents - Tools
This notebook, `AF_01_Agents_Tools_MCP.ipynb`, demonstrates how to integrate _tools_ with your AI agent, specifically focusing on a **Hosted MCP (Model Context Protocol) Tool**. Tools allow your agents to access external information and perform actions.

You will learn how to:
- Define a Hosted MCP Tool through the *.get_mcp_tool()* function of the **FoundryChatClient** class:

``` Python
mcp_doc_tool = client.get_mcp_tool(
    name = "microsoft-learn-mcp-tool",
    url = "https://learn.microsoft.com/api/mcp",
    approval_mode = "never_require"
)
```

- Then, equip the agent with the tool inside the *.as_agent()* constructor method:

``` Python
agent = client.as_agent(
    name = "microsoft-documentation-agent",
    instructions = """
    You are an expert on Microsoft technologies.
    Use the MCP documentation tool to fetch accurate, up-to-date information 
    from Microsoft Learn when answering questions.
    Keep responses concise (max 2-3 paragraphs).
    """,
    tools = [mcp_doc_tool]
)
```

- Stream the tool-equipped agent output to answer complex queries with MCP tool-backed fresh content:

``` JSON
User:
What is the difference between Prompt and Hosted Agents in Foundry?

Agent:
The main difference between Prompt agents and Hosted agents in Microsoft Foundry is how they are authored, managed, and run... [rest of the generated response]
```

## Notebook 2: Agents - Middleware
This notebook, `AF_02_Agents_Middleware.ipynb`, shows how to implement **agent middleware** to intercept and modify agent responses. Middleware provides a powerful way to enable content moderation, logging and result transformation without modifying core agent logic.

You will learn how to:
- Define **agent middleware** with the `@agent_middleware` decorator, to process AI agent's context:

``` Python
@agent_middleware
async def moderator_middleware(context, next):
    """Agent middleware that moderates output by replacing 'badword' with '***'."""
    print("Moderator: Starting agent invocation...")
    
    await next()
    
    if context.result and hasattr(context.result, "text"):
        # Scans context.result and masks prohibited words
        ...
```

- Attach the custom middleware to your agent profile using the *middleware* constructor parameter:

``` Python
agent = client.create_agent(
    name = "Storyteller-Agent",
    instructions = "You are a creative storyteller. Limit your responses to 1 paragraph.",
    middleware = [moderator_middleware]
)
```

- Intercept interactions, scans agent's output for inappropriate content and automatically replaces it before sharing the response with end users.

``` JSON
User:
Please, tell me a story about a curious red panda.

Moderator: Starting agent invocation...
Moderator: Replaced prohibited words
Moderator: Done

Agent:
In the heart of a misty bamboo forest, a curious red panda named Kiko set out on an adventure. Every day, Kiko would scamper through the trees... ignoring the warnings of an old owl about a “***” lurking near...
```

## Notebook 3: Agents - Observability
This notebook, `AF_03_Agents_Observability.ipynb`, explains how to automatically *log interactions* to connected **Azure Application Insights** resource and visualise *telemetry metrics* natively.

- Ensure that you install the required `azure-monitor-opentelemetry-exporter` Python package:

``` PowerShell
pip install --upgrade azure-monitor-opentelemetry-exporter
```

- Call the newly refactored async registration API method directly off your active *Foundry client* instance:

``` Python
await client.configure_azure_monitor(enable_live_metrics=True)
```

- You can now visualise the collected traces in Azure Monitor, by using the **Investigate -> Agents** menu section in Azure App Insights: 
![Observability_AppInsights](images/Observability_AppInsights.png)

- Alternatively, these traces can be visualised in a built-in Graphana dashboard as shown below:
![Observability_Graphana](images/Observability_Graphana.png)

## Notebook 4: Agents - Memory
This notebook, `AF_04_Agents_Memory.ipynb`, demonstrates how to achieve chat session persistence using the Agent Framework's **AgentSession** state-sharing serialisation and deserialisation APIs (`to_dict()` and `from_dict()`). The scenario simulates a hotel concierge agent that checks in a guest, loses its in-memory state and then recovers the conversation context to process a room service request.

You will learn how to:
- Create a trackable runtime session object:

``` Python
guest_session = agent.create_session()
```

- Serialise the session state to *JSON dictionary* to persist data on your local machine:

``` Python
serialised_session_data = guest_session.to_dict()

# Save thread state to a local JSON file
FILE_NAME = "concierge_session_memory.json"
with open(FILE_NAME, "w") as f:
    json.dump(serialised_session_data, f, indent=4)
```

- Rehydrate and deserialise the session state after the application restart:

``` Python
with open(FILE_NAME, "r") as f:
    session_data_reloaded = json.load(f)
    
restored_session = AgentSession.from_dict(session_data_reloaded)

# Execute ongoing prompts seamlessly by passing the session object
response_final = await agent_new.run(PROMPT2, session=restored_session)
```

- The resulting state reconstruction outputs reflect flawless context restoration:

``` JSON
User (Check-in):
Hello, my name is Alex Reed and I am checking in.

Agent:
Welcome, Alex Reed! It’s a pleasure to have you with us. I am assigning you room 402. If you need any assistance or information during your stay, please don't hesitate to ask. Enjoy your stay!

... (Application Restart/Memory Cleared) ...

User (Room Service):
I would like to order room service: a club sandwich and a pot of tea.

Agent:
Certainly, Alex! I have placed an order for a club sandwich and a pot of tea to be delivered to room 402. Is there anything else you would like to add or any preferences for your tea?
```

## Notebook 5: Prompt Agents (Foundry Deployment)
This notebook, `AF_05_Agents_Foundry.ipynb`, demonstrates how to transition from local agent definitions to fully managed Foundry-based cloud configurations.

You will learn how to:
- Deploy a local agent blueprint to your Microsoft Foundry project as a versioned *prompt agent* asset:

```python
cloud_agent = await project_client.agents.create_version(
    agent_name = agent.name,
    description = agent.description,
    definition = to_prompt_agent(agent),
    headers = {"Accept-Encoding": "identity"}
)
```

> [!IMPORTANT]
> We pass an explicit `Accept-Encoding: identity` header workaround to ensure the response policy doesn't encounter network compression pipeline conflicts during deployment.

- Instantiate a remote handle using the **FoundryAgent** class to run queries against your prompt agent in Foundry:

``` Python
deployed_agent = FoundryAgent(
    project_endpoint = PROJECT_ENDPOINT,
    agent_name = DEPLOYED_AGENT_NAME,
    agent_version = DEPLOYED_AGENT_VERSION,
    credential = DefaultAzureCredential()
)
```

- Programmatically delete (for *housekeeping* purposes) the versioned prompt agent once your execution tests are complete:

``` Python
await project_client.agents.delete_version(
    agent_name = DEPLOYED_AGENT_NAME,
    agent_version = DEPLOYED_AGENT_VERSION,
    headers = {"Accept-Encoding": "identity"}
)
```

- The resulting orchestration executes in Azure cloud, returning agent's prediction from a Foundry-based prompt agent:

``` JSON
User: Will my Python code run successfully on the first try tomorrow?

Agent:
Ah, seeker of the code, the celestial stars whisper a tale of both challenge and triumph...
On the dawn of your coding journey tomorrow, the moons hint at a gentle error or two, like hidden runes waiting to be deciphered. But with your keen insight and steady hand, these shadows shall vanish, revealing the true magic of your work.
```
