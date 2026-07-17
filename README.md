# Microsoft Agent Framework: Jupyter Notebooks for Ramp-Up Learning Process

**Microsoft Agent Framework** is an open-source development kit for building _AI agents_ and _multi-agent workflows_. It combines and extends constructs and concepts from two other Microsoft agentic frameworks: _Semantic Kernel_ and _AutoGen_.

This repo provides Jupyter notebooks to ramp up (Level 200: Agents V2) and build practical knowledge with Microsoft Agent Framework's _Python SDK_.

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
agent = ai_client.create_agent(
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
This notebook, `AF_03_Agents_Observability.ipynb`, explains how to enable **Open Telemetry** on an AI Foundry agent, so that any interactions are automatically logged and can be viewed in the connected _Azure Application Insights_ resource.

- Ensure that you install the required `azure-monitor-opentelemetry-exporter` Python package:

``` PowerShell
pip install azure-monitor-opentelemetry-exporter
```

- Open Telemetry can be auto-enabled through AI Foundry's connected App Insights resource by calling the following asynchronous function of the _AzureAIAgent_ class:

``` Python
await ai_client.setup_azure_ai_observability()
```

- You can now visualise the collected traces in Azure Monitor, by using the **Investigate -> Agents** menu section in Azure App Insights: 
![Observability_AppInsights](images/Observability_AppInsights.png)

- Alternatively, these traces can be visualised in a built-in Graphana dashboard as shown below:
![Observability_Graphana](images/Observability_Graphana.png)

## Notebook 4: Agents - Memory
This notebook, `AF_04_Agents_Memory.ipynb`, demonstrates how to achieve persistence in agent conversations by using thread _serialisation_ and _deserialisation_. The scenario simulates a hotel concierge agent that checks in a guest, loses its in-memory state and then recovers the conversation context to process a room service request.

You will learn how to:
- Create a new thread for a conversation:

``` Python
guest_thread = agent.get_new_thread()
```

- Serialise the thread state to save it to persistent storage (e.g., a JSON file) after the first interaction:

``` Python
serialised_thread_data = await guest_thread.serialize()

# Save to file
with open(FILE_NAME, "w") as f:
    json.dump(serialised_thread_data, f, indent=4)
```

- Deserialise the thread state after the application restart to restore the conversation history into a new agent instance:

``` Python
# Load from file
with open(FILE_NAME, "r") as f:
    thread_data_reloaded = json.load(f)
    
restored_thread = await agent_new.deserialize_thread(thread_data_reloaded)
```

The whole conversation, including the app restart, may look like this:
``` JSON
User (Check-in):
Hello, my name is Alex Reed and I am checking into room 1205.

Agent:
Welcome, Alex Reed! I have you checked into room 1205. If you need any assistance during your stay, feel free to ask. How can I help you today?

... (Application Restart/Memory Cleared) ...

User (Room Service):
I would like to order room service: a club sandwich and a pot of tea.

Agent:
Certainly, Alex! I’ll place an order for a club sandwich and a pot of tea to be delivered to room 1205. Is there any particular time you would like this to be served?
```
