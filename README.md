# Foundry agent using MCP – my own server and client

Microsoft Learn lab **03 – Extend agents with Model Context Protocol (MCP) tools**
from the course **AI-3026: Develop AI agents on Azure**.

- Lab instructions: https://microsoftlearning.github.io/mslearn-ai-agents/Instructions/Exercises/03-mcp-integration.html
- All exercises (I'm following these): https://microsoftlearning.github.io/mslearn-ai-agents/

## What this project does

| File | What it does |
|---|---|
| `agent.py` | Foundry agent uses the **remote** Microsoft Learn Docs MCP server (`https://learn.microsoft.com/api/mcp`). Each tool call needs approval; the code auto-approves in a loop. |
| `server.py` | My **own** MCP server ("Inventory", built with FastMCP) with 2 tools: `get_inventory_levels`, `get_weekly_sales`. |
| `client.py` | The **middleman**: starts `server.py` locally, gives its tools to a Foundry agent (`inventory-agent`), runs them when the agent asks, and sends the results back. |

Why a client is needed: the agent runs in the cloud (Foundry), but `server.py` runs on my laptop with no web address, so the cloud can't reach it directly.

## Bugs I fixed in the lab's `client.py`

1. `input_list` was never emptied, so old tool results were re-sent and the agent repeated old answers. Fix: `input_list = []` at the start of each chat turn.
2. Tool results were sent with `previous_response_id` instead of into the conversation, which left unanswered tool calls in the history (error 400 "No tool output found for function call"). Fix: send them with `conversation=conversation.id`.

## How to run

1. Deploy `gpt-5` in a Foundry project (Tokens per minute ≥ 150000).
2. Python 3.12 or 3.13 (3.14 is not supported):
```
   py -3.12 -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
```
3. Create a `.env` file:
```
   PROJECT_ENDPOINT="https://<resource>.services.ai.azure.com/api/projects/<project>"
   MODEL_DEPLOYMENT_NAME="gpt-5"
```
4. `az login`, then `python agent.py` or `python client.py`.