# CRM AI Agent Cookbook

<p align="center">
  <strong>Build a Telegram-based AI assistant for CRM workflows with Yandex Cloud, AI Studio, amoCRM MCP, Workflows, SpeechKit, Cloud Functions, API Gateway, and YDB.</strong>
</p>

<p align="center">
  <a href="./crm_ai_agent_cookbook.ipynb"><img alt="Notebook" src="https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter"></a>
  <img alt="Yandex Cloud" src="https://img.shields.io/badge/Yandex%20Cloud-Serverless-blue">
  <img alt="Telegram Bot" src="https://img.shields.io/badge/Telegram-Bot-26A5E4?logo=telegram&logoColor=white">
  <img alt="amoCRM" src="https://img.shields.io/badge/amoCRM-MCP%20Server-purple">
  <img alt="Python" src="https://img.shields.io/badge/Python-Cloud%20Functions-3776AB?logo=python&logoColor=white">
</p>

---

## Overview

This repository contains a step-by-step cookbook notebook for deploying an **AI assistant for CRM operations** using **amoCRM** as an example CRM system.

The assistant lets sales managers interact with CRM data directly from a **Telegram bot** by sending text or voice messages. It can search for clients, leads, contacts, companies, tasks, notes, and pipelines, as well as create or update CRM records through an AI-driven workflow.

> The main cookbook is written in Russian: [`crm_ai_agent_cookbook.ipynb`](./crm_ai_agent_cookbook.ipynb).

---

## What you will build

By following the notebook, you will deploy a serverless CRM assistant that can:

- 💬 receive text messages from Telegram;
- 🎙️ transcribe voice messages with Yandex SpeechKit;
- 🤖 send user requests to an AI Studio agent through the Responses API;
- 🔌 connect the agent to amoCRM through an MCP server;
- 🔐 authorize Telegram users through a YDB document table;
- 🧩 orchestrate the request lifecycle with Yandex Workflows;
- 🌐 expose the Telegram webhook through API Gateway;
- 🧹 clean up all created cloud resources when finished.

---

## Architecture

![CRM AI Agent architecture](./assets/architecture.png)

The solution uses the following Yandex Cloud services and components:

| Layer | Component | Purpose |
|---|---|---|
| User interface | Telegram Bot | Receives text and voice requests from users |
| Entry point | API Gateway | Accepts Telegram webhook events |
| Orchestration | Workflows | Validates users, calls functions, waits for AI responses, sends replies |
| AI layer | AI Studio Agent | Interprets CRM requests and decides how to act |
| CRM integration | amoCRM MCP Server | Gives the AI agent access to selected amoCRM tools |
| Voice processing | SpeechKit | Converts Telegram voice messages to text |
| Compute | Cloud Functions | Handles request preparation and response retrieval |
| Storage | Managed Service for YDB | Stores authorized Telegram chat IDs and AI response IDs |
| Secrets | Lockbox | Stores Telegram and amoCRM tokens |

---

## Repository structure

```text
.
├── crm_ai_agent_cookbook.ipynb   # Main step-by-step cookbook notebook
├── assets/
│   └── architecture.png          # Solution architecture diagram
└── README.md                     # Repository overview and usage guide
```

---

## Prerequisites

Before starting, make sure you have:

- an active or trial Yandex Cloud billing account;
- `admin` permissions in the target Yandex Cloud folder;
- an amoCRM account and a long-lived amoCRM token;
- a Telegram bot created through [BotFather](https://t.me/BotFather);
- access to Yandex AI Studio, Workflows, Cloud Functions, API Gateway, Lockbox, SpeechKit, and Managed Service for YDB.

The notebook creates and uses these service accounts:

| Service account | Used by | Required roles |
|---|---|---|
| `sa-apigw` | API Gateway | `serverless.workflows.executor` |
| `sa-workflows` | Workflows | `lockbox.payloadViewer`, `functions.functionInvoker`, `ydb.editor` |
| `sa-ai-agent` | Cloud Functions | `ai.assistants.viewer`, `ai.languageModels.user`, `ai.speechkit-stt.user`, `lockbox.payloadViewer`, `serverless.mcpGateways.invoker`, `functions.viewer` |
| `sa-mcp-server` | MCP server | `lockbox.payloadViewer` |

---

## Quick start

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd <your-repository-name>
```

### 2. Open the cookbook

Open the notebook in Jupyter, VS Code, or another notebook-compatible environment:

```bash
jupyter notebook crm_ai_agent_cookbook.ipynb
```

You can also review it directly on GitHub.

### 3. Prepare cloud resources

Follow the notebook sections to:

1. create service accounts and assign roles;
2. create Telegram and amoCRM secrets in Lockbox;
3. create a serverless YDB database named `crm-bot-db`;
4. create a document table named `chat-bot-users`;
5. add authorized Telegram `chat_id` values to the table.

### 4. Configure the CRM AI agent

In Yandex AI Studio:

1. create an MCP server from the **amoCRM** template;
2. connect the required amoCRM tools;
3. create an AI agent named `crm-ai-agent`;
4. add the agent instructions from the notebook;
5. attach the private amoCRM MCP server to the agent.

### 5. Deploy Cloud Functions

The notebook defines two Python functions:

| Function | Purpose |
|---|---|
| `crm-ai-agent-request` | Handles Telegram messages, transcribes voice input, and starts an AI agent response |
| `crm-ai-agent-result` | Retrieves the final AI agent response by `response_id` |

Both functions use Python dependencies such as:

```text
openai==2.9.0
pyTelegramBotAPI==4.27
```

### 6. Create the workflow

Create a Yandex Workflows workflow named `crm-bot-workflows` using the YaWL specification from the notebook.

The workflow:

1. checks whether the Telegram user is authorized in YDB;
2. sends the request to the AI agent function;
3. polls the AI response status;
4. returns the final answer to Telegram;
5. denies access for unknown users.

### 7. Create the API Gateway

Create an API Gateway named `crm-bot-api-gw` and paste the OpenAPI specification from the notebook.

After deployment, save the gateway service domain and configure the Telegram webhook:

```python
BOT_TOKEN = "<telegram_bot_token>"
API_GW_DOMAIN = "<api_gateway_service_domain>"
```

The webhook URL should point to:

```text
<API_GW_DOMAIN>/handle
```

---

## Testing

After deployment:

1. Open your Telegram bot.
2. Send:

   ```text
   Расскажи, что ты умеешь делать
   ```

3. Try a CRM query:

   ```text
   Выведи информацию о сделках компании <название_компании>
   ```

4. Test a voice message.
5. Use `/clear` from the bot menu to reset the conversation context.

Expected reset response:

```text
Контекст предыдущего общения очищен. Начнем с чистого листа.
```

---

## Security notes

- Do not commit real Telegram or amoCRM tokens to the repository.
- Store all secrets in Yandex Lockbox.
- Add only trusted Telegram `chat_id` values to the YDB authorization table.
- Keep the MCP server private.
- Assign the minimum required roles to each service account.
- Delete demo resources after testing to avoid unnecessary costs.

---

## Cleanup

To stop paying for created resources, delete:

- API Gateway `crm-bot-api-gw`;
- Workflow `crm-bot-workflows`;
- Cloud Functions `crm-ai-agent-request` and `crm-ai-agent-result`;
- AI agent `crm-ai-agent`;
- MCP server `amocrm-mcp-server`;
- YDB database `crm-bot-db`;
- Lockbox secrets for Telegram and amoCRM tokens;
- log groups, if you enabled custom logging.

---

## Troubleshooting

| Issue | What to check |
|---|---|
| Telegram bot does not respond | Webhook URL, API Gateway domain, workflow execution logs |
| User receives access denied | `chat_id` exists in the `chat-bot-users` YDB table |
| Voice messages fail | SpeechKit role on `sa-ai-agent`, audio payload handling, function logs |
| AI response is empty or stuck | Responses API status, workflow polling loop, `response_id` storage |
| amoCRM actions fail | MCP server configuration, amoCRM token, selected MCP tools |
| Function cannot access secrets | `lockbox.payloadViewer` role and correct Lockbox secret IDs |

---

## Suggested improvements

Ideas for extending this cookbook:

- add Terraform or CLI automation for resource provisioning;
- add screenshots for every cloud-console step;
- add a minimal demo dataset for amoCRM;
- add structured logging and monitoring dashboards;
- add CI checks for notebook validity;
- provide separate Russian and English README files.

---

## License

Add your preferred license before publishing the repository.

