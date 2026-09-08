# Daily ChatGPT → Telegram Automation Research

**Goal:** Every day, ChatGPT itself researches current Ethiopian education/university news, writes the compact briefing text, and the exact text is delivered automatically to Telegram.

**Repository:** `ermiyas48/books`
**Research status:** living document; update this page whenever new alternatives, connectors, schedulers, or proven implementations are discovered.
**Last researched:** 2026-09-08

## 1. Target architecture

Preferred:

`Daily scheduler → ChatGPT scheduled task → fresh web research → compact briefing text → Telegram delivery`

The critical requirement is that the **AI-written text remains the source of truth**. A deterministic RSS script or separate news model is only a fallback and should not be described as ChatGPT.

## 2. ChatGPT-native scheduling

OpenAI documents Scheduled Tasks as recurring work that ChatGPT runs later, including daily updates. Tasks are supported on Web, iOS, Android, and macOS; task completion can generate push/email notifications. Free and Go plans can run a recurring task no more than once per day; paid eligible plans allow more frequent schedules. OpenAI also documents event-triggered tasks, but those are separate and currently support Gmail, Slack, and GitHub activity rather than arbitrary Telegram webhooks.

Sources:
- https://help.openai.com/en/articles/10291617
- https://help.openai.com/en/articles/6825453-

Important limitation: the public ChatGPT Scheduled Task interface does not expose a generic outbound webhook containing the generated answer. Therefore another bridge is needed for direct Telegram delivery.

## 3. Telegram delivery routes

### A. Existing Composio Telegram connection — BEST current delivery connector

Already connected and verified in this environment.

- Toolkit: Telegram
- Connection alias: `telegram_yere-gyle`
- Bot: `ErmiAgentBot`
- Destination: configured private chat
- Tool: `TELEGRAM_SEND_MESSAGE`

This is the preferred direct delivery path whenever the automation runtime can invoke Composio.

Composio also supports hosted MCP sessions and dynamic tool access across many apps.

Source:
- https://github.com/ComposioHQ/composio

### B. Direct Telegram Bot API MCP servers

These use a BotFather bot token and are appropriate when the runtime can host an MCP server.

1. **TONresistor/telegram-mcp** — production-oriented Bot API server; 162 Bot API methods, token-optimized meta mode, retries/rate limiting/health metrics. Meta mode exposes 2 tools: `telegram_find` and `telegram_call`; standard mode exposes the full method set.
   - https://github.com/TONresistor/telegram-mcp
   - Strong option for a self-hosted bot bridge.

2. **NexusX-MCP/telegram-mcp-server** — Bot API MCP service with `get_bot_info`, `send_message`, and `get_updates`.
   - https://github.com/NexusX-MCP/telegram-mcp-server
   - Simple, focused bot bridge.

3. **guangxiangdebizi/telegram-mcp** — comprehensive Bot API MCP server; supports stdio and an SSE mode and exposes messaging/media/forward/delete/chat operations.
   - https://github.com/guangxiangdebizi/telegram-mcp
   - Useful when a remote HTTP/SSE MCP transport is required.

4. **py2755/aiogram-mcp** — turns an existing aiogram Telegram bot into an MCP server; documented as 30 tools, 7 resources, 3 prompts, and real-time event notifications.
   - https://github.com/py2755/aiogram-mcp
   - Excellent choice if an existing aiogram bot is already part of the architecture.

### C. Telegram user-account MCP servers (MTProto/Telethon/GramJS)

These are not bot connectors. They operate as the user's Telegram account and can access personal chats/groups/channels that a bot cannot.

1. **chigwell/telegram-mcp** — Telethon/MTProto, broad account/chat/media/admin capabilities, Docker support and active maintenance.
   - https://github.com/chigwell/telegram-mcp
   - Good general-purpose user-account MCP.

2. **TONresistor/telethon-mcp** — production-oriented Telethon MCP with 6 meta-tools + raw Telegram API access covering 742 methods, persistent caching, rate-limit protection.
   - https://github.com/TONresistor/telethon-mcp
   - Most powerful user-account API coverage among the researched projects.

3. **Matancoo/telegram-mcp** — Telethon + FastMCP, documented as 89 tools across 13 categories.
   - https://github.com/Matancoo/telegram-mcp
   - Feature-rich but newer/smaller project.

4. **newink/telegram-mcp** — MTProto personal-account connector for dialogs, messages, search, media, etc.
   - https://github.com/newink/telegram-mcp
   - Good when the AI needs access to real account history.

5. **DmitryKhali/telegram-mcp** — Telethon user account; send operations require explicit confirmation in the documented implementation.
   - https://github.com/DmitryKhali/telegram-mcp
   - Better for human-in-the-loop use than unattended sending.

6. **jgalea/telegram-mcp** — Telethon/MTProto with about 40 tools including scheduling, reactions, admin operations and passive SQLite caching.
   - https://github.com/jgalea/telegram-mcp/blob/main/README.md
   - Broad feature set; evaluate maintenance before production deployment.

7. **m0n0x41d/telegram-mcp** — Telethon-based personal-account MCP with read/send/search/media operations and resumable scan workflow tools.
   - https://github.com/m0n0x41d/telegram-mcp
   - Good for structured message ingestion/search as well as sending.

8. **nguyenvanduocit/telegram-mcp** — Go + gotd/td, documented as 59 tools plus compound workflow tools and prompts; supports stdio and HTTP transports and Docker.
   - https://github.com/nguyenvanduocit/telegram-mcp
   - Attractive for a lightweight remote server without Python.

9. **tamlut-modnys/telegram-mcp-server** — Telethon/FastMCP Telegram MCP server with list/read/search/send operations and optional SSE debugging mode.
   - https://github.com/tamlut-modnys/telegram-mcp-server
   - Useful as a simple server reference.

10. **Shaan-alpha/telegram-mcp** — small local Telethon MCP: `get_me`, list chats, history, search and send. Uses a reusable session string.
    - https://github.com/Shaan-alpha/telegram-mcp
    - Very simple to audit/modify.

11. **prem-research/telegram-mcp** — Telethon-based MCP with a separate HTTP server and MCP server, including `send_message` and unread-message retrieval.
    - https://github.com/prem-research/telegram-mcp
    - Interesting for splitting HTTP collection from MCP exposure.

12. **tensakulabs/telegram-mcp** — minimal Telethon/MTProto MCP focused on two tools (`send_message`, `get_history`) for bot interaction.
    - https://github.com/tensakulabs/telegram-mcp
    - Useful as a tiny learning/reference implementation.

13. **sparfenyuk/mcp-telegram** — MTProto MCP bridge; researched README documents mostly read-only access.
    - https://github.com/sparfenyuk/mcp-telegram
    - Not suitable as the primary Telegram sender unless write support is added.

14. **mcp-telegram/mcp-telegram** — GramJS/MTProto hosted and self-hosted project; documented as supporting messages, media, reactions, polls, stickers, contacts, groups, sessions, privacy and more, with a hosted cloud version aimed at ChatGPT/Claude-compatible clients.
    - https://github.com/mcp-telegram/mcp-telegram
    - Potentially the most interesting hosted user-account MCP route to investigate further.

## 4. Telegram client libraries useful for custom bridges

- **Telethon** — mature Python MTProto client; can send messages and supports bots as well.
  - https://github.com/LonamiWebs/telethon
- **GramJS** — JavaScript/Node MTProto client. The upstream repository is archived as of July 14, 2026; its README directs users toward the actively maintained `teleproto` fork.
  - https://github.com/gram-js/gramjs

For a new Node-based implementation, prefer an actively maintained successor rather than archived GramJS unless compatibility requires it.

## 5. Scheduler / orchestration alternatives

### 5.1 ChatGPT Scheduled Tasks

**Best match for the requirement that ChatGPT writes the briefing.** It natively runs recurring prompts and can search the web for monitoring/update tasks. Main missing interface: no documented arbitrary webhook carrying the generated output directly to Telegram.

### 5.2 Composio

Composio provides authenticated tool access, triggers and hosted MCP sessions. It already has the Telegram and Perplexity connections in this environment. Research found recurring webhook scheduling via a separate `cronfree_time_scheduler` connector, but that connector is not currently authenticated here.

Useful conceptual path:
`Cron/Webhook scheduler → AI action → Telegram`

Source:
- https://github.com/ComposioHQ/composio

### 5.3 CronFree recurring webhook scheduler

Composio tool discovery exposes `CRONFREE_TIME_SCHEDULER_CREATE_RECURRING_WEBHOOK_SCHEDULE`, which can repeatedly POST to a public HTTPS webhook on selected weekdays/months/hours/minutes/timezone. This is promising as the scheduler half of a future bridge, but it still requires a webhook endpoint that can invoke ChatGPT or an equivalent AI runtime.

### 5.4 GitHub Actions

GitHub Actions can schedule workflows with cron and timezone. It is excellent for always-on deterministic jobs, but it is **not ChatGPT itself**. A true ChatGPT-generated daily briefing would require a compatible external AI API or another way to invoke ChatGPT.

Existing repo `ermiyas48/books` is suitable as a control/configuration repository.

### 5.5 Android/Termux scheduler

The existing phone environment is useful because it can stay close to the Telegram bot and can run Termux/Automate workflows. The key architecture is:
`ChatGPT task notification → Android notification bridge → Termux/Automate → Telegram`

This retains ChatGPT as the writer but depends on Android reliably exposing the full generated text in the notification. This must be tested on the actual device; notification truncation is the main risk.

### 5.6 External automation platforms

Candidate classes worth evaluating if direct ChatGPT webhooks remain unavailable:
- n8n
- Activepieces
- Pipedream
- Make
- Zapier
- Node-RED
- Cloudflare Workers + scheduled/HTTP trigger
- Google Apps Script / Gmail relay

Key criterion: they must receive the **actual ChatGPT-generated text**, not merely a 'task completed' notice.

## 6. Approaches rejected or downgraded

### Email-only ChatGPT task notification

Not sufficient as the primary bridge unless the actual generated task result is included in the email body. OpenAI's documented task notification settings support push/email notifications, but there is no documented guarantee that a task's full generated result is available as an email payload suitable for extraction.

### Deterministic RSS → Telegram

Works technically, but violates the user's core requirement because the final text is produced by a script rather than ChatGPT. Keep only as an emergency fallback.

### Perplexity as permanent writer

Perplexity is connected and can do excellent web-grounded research, but the requested system is explicitly **ChatGPT writes the text**. Perplexity can be a research fallback or auxiliary source, not the default writer.

## 7. Best architecture ranking

| Rank | Architecture | ChatGPT writes final text? | Fully unattended? | Complexity | Verdict |
|---|---|---:|---:|---:|---|
| 1 | ChatGPT Scheduled Task → Android notification → Automate/Termux → Telegram | Yes | Yes, if Android task/notification is reliable | Medium | **Best practical route now** |
| 2 | ChatGPT Scheduled Task → first-class outbound webhook → Telegram/Composio | Yes | Yes | Low | **Best overall if/when available** |
| 3 | ChatGPT Scheduled Task → email/result relay → automation → Telegram | Maybe | Yes | Medium | Needs verified full-result email payload |
| 4 | CronFree webhook → external ChatGPT-capable runtime → Telegram | Potentially | Yes | High | Strong infrastructure route, but needs ChatGPT invocation endpoint |
| 5 | GitHub Actions → external AI API → Telegram | No (unless API is ChatGPT-compatible and intentionally used) | Yes | Medium | Good fallback, not exact requirement |
| 6 | Perplexity scheduled/research automation → Telegram | No | Yes | Low/Medium | Research fallback only |
| 7 | RSS-only → Telegram | No | Yes | Low | Emergency fallback only |

## 8. Current environment state

- Telegram Composio connection is active and tested.
- GitHub Composio connection is active and has admin access to `ermiyas48/books`.
- Perplexity Composio connection is active.
- The repository currently had no workflow runs or repository secrets at the time of this research.
- The daily-briefing skill is saved in the ChatGPT Library.
- This page is now the living GitHub research log.

## 9. Next investigations

1. Determine whether ChatGPT Scheduled Tasks expose the full result through Android notification text on the current Android app version.
2. Build a minimal Automate/Termux notification-to-Telegram proof of concept and test maximum text length/truncation.
3. Investigate whether the hosted `mcp-telegram` service can be connected directly as an MCP endpoint to an AI client that can run scheduled tasks.
4. Investigate Composio/Rube custom MCP + CronFree webhook combinations for an end-to-end hosted bridge.
5. Compare n8n, Activepieces, Pipedream, Make, and Cloudflare Worker patterns specifically for forwarding a ChatGPT task result.
6. Keep this page updated with every materially new connector, scheduler, protocol, or proven implementation.

## 10. Research rule for future sessions

Whenever this automation is discussed, inspect this page first, then research what has changed since its last update. Append new findings rather than replacing old evidence. Never claim an automation path is live until a real end-to-end test proves: **scheduled trigger → ChatGPT-generated text → Telegram message received**.
