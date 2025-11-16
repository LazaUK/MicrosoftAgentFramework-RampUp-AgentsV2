# Microsoft Agent Framework: Jupyter Notebooks for Ramp-Up Learning Process
**Microsoft Agent Framework** is an open-source development kit for building _AI agents_ and _multi-agent workflows_. It combines and extends constructs and concepts from two other Microsoft agentic frameworks: _Semantic Kernel_ and _AutoGen_.

This repo provides Jupyter notebooks to ramp up (Level 200) and build practical knowledge of Microsoft Agent Framework's _Python SDK_.

> [!WARNING]
> To successfully run these notebooks, you must have an **Azure AI Foundry** project and **AI model deployment**. Please ensure you have the following environment variables set up in your system:
> | Environment Variable             | Description                                                                     |
> | -------------------------------- | ------------------------------------------------------------------------------- |
> | `AZURE_FOUNDRY_PROJECT_ENDPOINT` | The endpoint URL for your Azure AI Foundry project.                             |
> | `AZURE_FOUNDRY_GPT_MODEL`        | The name of the model deployment to be used by the agent, e.g., _gpt-4.1-mini_. |

## 📑 Table of Contents
- [Notebook 0: Quick Start](#notebook-0-quick-start)
- [Notebook 1: Agents - Tools](#notebook-1-agents---tools)
- [Notebook 2: Agents - Middleware]()
- [Notebook 3: Agents - Observability]()
- [Notebook 4: Agents - Memory]()
- [Notebook 5: Agents - ]()

## Notebook 0: Quick Start
This notebook, `AF_00_GettingStarted_QuickStart.ipynb`, provides general introduction to the Agentic Framework. It focuses on the core steps required to get a basic agent running.

- Configure your environment and set up necessary imports, incl. suppressing user warnings from `agent_framework_azure` Python package:

``` Python
# Import required packages
import os
import asyncio
from agent_framework import ChatAgent
from agent_framework.azure import AzureAIAgentClient
from azure.identity.aio import DefaultAzureCredential

import warnings
warnings.filterwarnings(
    action = "ignore",
    category = UserWarning,
    module = "agent_framework_azure"
)
```

- Create an AI Client using _AzureAIAgentClient_ to connect to your Azure AI Foundry project:

``` Python
ai_client = AzureAIAgentClient(
    agent_name = "Azure AI Agentic Client",
    project_endpoint = PROJECT_ENDPOINT,
    model_deployment_name = MODEL_DEPLOYMENT,
    async_credential = DefaultAzureCredential()
)
```

- Instantiate a basic Chat Agent using the _ChatAgent_ class:

``` Python
agent = ChatAgent(
    chat_client = ai_client,
    name = "Haiku Poet Agent",
    instructions = "You are a haiku poet. Respond to user queries with a haiku."
)
```

- Run the agent using both **non-streaming** _agent.run(prompt)_ and **streaming** _agent.run_stream(prompt)_ methods to test its response capabilities.

``` Python
response = await agent.run(prompt)
...
async for streaming_update in agent_alt.run_stream(prompt_alt):
    if streaming_update.text:
        print(streaming_update.text, end="", flush=True)
```

The code creates a Haiku Poet Agent that responds to user queries with a haiku:

``` JSON
User: What is life?

Agent: River's gentle flow,  
Moments dance in fleeting light,  
Life blooms, then fades soft.
```

## Notebook 1: Agents - Tools
