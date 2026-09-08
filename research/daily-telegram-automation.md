# Daily ChatGPT → Telegram Automation — Living Research & Implementation Plan

**Last researched:** 2026-09-08  
**Goal:** Every day, ChatGPT itself researches current Ethiopia/university news, writes a very short briefing, and the resulting text reaches Telegram chat `6725547584`. The AI writer must be ChatGPT; replacing it with a static RSS script is not acceptable.

## 1. Exact target architecture

```text
Daily scheduler
    ↓
ChatGPT Scheduled Task / ChatGPT Work
    ↓
Web research + synthesis
    ↓
Compact final text (3–5 bullets)
    ↓
Delivery bridge
    ↓
Telegram bot → chat 6725547584
```

The hard engineering distinction is **generation vs delivery**. Telegram MCP/Bot API can deliver text, but it does not by itself wake this ChatGPT runtime every morning. A scheduler can trigger a separate AI/API workflow, but that is not the same as ChatGPT Scheduled Tasks. Therefore the preferred design keeps ChatGPT as the generator and uses Android as the local delivery bridge.

## 2. Current best solution: ChatGPT Task → Android notification → Automate → Telegram Bot API

### Why this is currently #1
- ChatGPT Scheduled Tasks can run one-time or recurring tasks and can perform recurring work and web-based monitoring. OpenAI documents daily recurring tasks for Free/Go and more frequent schedules for eligible paid plans. See: https://help.openai.com/en/articles/10291617
- On Android, ChatGPT task notifications can be delivered as mobile push notifications when notification permission is enabled.
- Android's NotificationListenerService is specifically designed to receive posted notifications. See: https://developer.android.com/reference/android/service/notification/NotificationListenerService
- Automate has a `Notification posted` block that exposes the notification title and message as variables, and an `HTTP request` block capable of POSTing data. See: https://llamalab.com/automate/doc/block/notification_posted.html and https://llamalab.com/automate/doc/block/http_request.html
- Telegram's Bot API can then receive the generated text through `sendMessage`.

### Required user-side setup
1. Install/update Automate on the Samsung M12.
2. Give Automate notification access.
3. Create one ChatGPT Scheduled Task with the exact briefing prompt.
4. Allow ChatGPT notifications.
5. Create one Automate flow:
   - Notification posted
   - filter package = ChatGPT
   - filter for the task notification/title if necessary
   - extract `Message`
   - HTTP POST to Telegram Bot API `sendMessage`
   - use chat ID `6725547584`
6. Enter the Telegram bot token into Automate's protected credential/keychain mechanism if available; never commit it to GitHub.
7. Disable battery optimization for ChatGPT and Automate on the M12 so Android does not kill the bridge.
8. Run an end-to-end test with a short test task before relying on the daily task.

### Important uncertainty to test
The exact ChatGPT notification payload can vary by app version and task notification type. Automate can capture notification title/message, but the full task answer must be verified on the actual phone. If the notification only contains a completion notice and not the full generated text, switch to a second bridge described below instead of pretending the payload contains the answer.

## 3. Strong fallback: ChatGPT Task → visible result → Android/Tasker/Automate bridge

If the ChatGPT mobile notification does not include the full answer, investigate Android accessibility/UI extraction as a fallback. Automate has interface/accessibility blocks and can interact with visible app UI. This is less robust than notification extraction because UI layout can change.

Use only as a fallback; do not make screen scraping the primary architecture.

## 4. Strong fallback: ChatGPT Task → email notification → Gmail/automation → Telegram

OpenAI Scheduled Tasks support Push and Email notifications. However, task email behavior must be tested: a completion notification is not guaranteed to contain the complete task response. Therefore email should be treated as a trigger/heartbeat, not assumed to be the content transport.

If the email contains the full generated text in the current implementation, Gmail can become a reliable bridge. If it only says the task finished, this route cannot satisfy the exact requirement without another way to retrieve the task result.

## 5. Direct ChatGPT Scheduled Task → Telegram connector

This would be the cleanest architecture if ChatGPT Scheduled Tasks could directly invoke Telegram as a supported connected app. Current OpenAI documentation lists supported event-triggered apps such as Gmail, Slack, and GitHub; Telegram is not listed there. Scheduled Tasks also do not expose a generic arbitrary MCP webhook/action interface in the documented feature set.

Conclusion: **do not assume the existing Composio Telegram connection can automatically be called by a ChatGPT Scheduled Task.** The fact that ChatGPT can use a connector interactively does not prove scheduled tasks can invoke it unattended.

## 6. Composio Telegram

Existing connected Telegram account: `telegram_yere-gyle`.

Composio provides managed Telegram integration through its tool/MCP layer and can send Telegram messages. This is excellent for interactive ChatGPT → Telegram delivery, but the remaining question is whether a currently exposed Composio scheduler/trigger can independently wake a ChatGPT generation job every day. The available connector search did not expose a dependable recurring scheduler that can execute this exact ChatGPT-runtime workflow.

Composio is therefore retained as the **interactive/manual delivery path and future upgrade path**, not falsely declared to be the daily scheduler.

Reference: https://composio.dev/content/telegram-mcp-connect-your-ai-to-your-telegram-chats

## 7. Telegram MCP ecosystem researched

### Bot API MCP servers — best when the destination is a bot

1. **timoncool/telegram-api-mcp** — 169 Telegram Bot API methods, meta mode, retries, rate limiting, circuit breaker, token masking. https://github.com/timoncool/telegram-api-mcp
2. **TONresistor/telegram-mcp** — 161 Bot API methods, meta mode with `telegram_find` + `telegram_call`. https://github.com/TONresistor/telegram-mcp
3. **FantomaSkaRus1/telegram-bot-mcp** — 174 Bot API tools. https://github.com/FantomaSkaRus1/telegram-bot-mcp
4. **node2flow-th/telegram-bot-mcp-community** — 27 Bot API tools including messages, webhooks and files. https://github.com/node2flow-th/telegram-bot-mcp-community
5. **NexusX-MCP/telegram-mcp-server** — focused Bot API server with send_message/get_updates/get_bot_info. https://github.com/NexusX-MCP/telegram-mcp-server
6. **guangxiangdebizi/telegram-mcp** — broad Bot API interface with messaging, media and chat management. https://github.com/guangxiangdebizi/telegram-mcp
7. **abhinavkale-dev/telegram-mcp-server** — simple Bot API MCP server. https://github.com/abhinavkale-dev/telegram-mcp-server

For this project, these are mostly **delivery engines**, not schedulers or ChatGPT task triggers.

### MTProto/user-account MCP servers — powerful but unnecessary for this goal

1. **tensakulabs/telegram-mcp** — minimal Telethon/MTProto server. https://github.com/tensakulabs/telegram-mcp
2. **DmitryKhali/telegram-mcp** — personal-account Telegram access using Telethon. https://github.com/DmitryKhali/telegram-mcp
3. **TONresistor/telethon-mcp** — broad Telethon API access with meta tools and rate-limit protection. https://github.com/TONresistor/telethon-mcp
4. **Matancoo/telegram-mcp** — 89 Telethon/MTProto tools. https://github.com/Matancoo/telegram-mcp
5. **Shaan-alpha/telegram-mcp** — local Telethon MCP with read/write tools. https://github.com/Shaan-alpha/telegram-mcp
6. **Muhammadyunusxon/telegram-mcp** — personal Telegram control including scheduled messages. https://github.com/Muhammadyunusxon/telegram-mcp
7. **mcp-telegram / @overpod/mcp-telegram** — GramJS/MTProto userbot MCP with messages, media, contacts, groups, sessions and privacy. https://github.com/mcp-telegram

MTProto is powerful when the AI must operate as the user's personal Telegram account. For this project a Bot API is simpler and safer because the destination is a single bot chat.

## 8. Other automation platforms researched

### Zapier — technically capable, but changes the AI architecture
Zapier has Schedule by Zapier + Telegram and also Telegram + ChatGPT (OpenAI). It supports daily scheduled Telegram messages and exposes Telegram/ChatGPT through Zapier MCP. Sources:
- https://zapier.com/apps/schedule/integrations/telegram
- https://zapier.com/apps/telegram/integrations/chatgpt

Architecture would be:
`Zapier schedule → OpenAI/ChatGPT action → Telegram`.

This is a proven automation architecture, but the AI step is an OpenAI API/Zapier action, not the exact ChatGPT Scheduled Task runtime. It is therefore the **best cloud alternative if exact ChatGPT runtime is relaxed**.

### n8n
Self-hosted/cloud n8n can combine Schedule Trigger → web research/API → LLM → Telegram. This is excellent if the AI can be an API model. It does not automatically turn a ChatGPT consumer scheduled task into a callable API endpoint.

### Pipedream
Pipedream can schedule workflows and call APIs/Telegram. Same limitation: excellent for API-based AI generation, not a documented direct trigger for a ChatGPT consumer task.

### Make
Make can run scheduled scenarios and Telegram actions. Same architectural limitation: it is an external automation/AI workflow, not ChatGPT Scheduled Tasks itself.

### GitHub Actions
GitHub Actions supports scheduled workflows with cron and IANA timezone-aware scheduling. The shortest supported schedule interval is 5 minutes. Source: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax

It is good for deterministic RSS/API pipelines. It is **not** currently a direct way to invoke ChatGPT consumer Scheduled Tasks. GitHub Models also cannot be treated as the replacement for ChatGPT because that product was retired.

### Cloudflare Workers / AWS / Firebase / generic cron
All can schedule a webhook or serverless function and call Telegram Bot API. They are reliable delivery/scheduling infrastructure but require an AI API or another callable generation endpoint. They do not natively wake a ChatGPT consumer task.

## 9. Android/Termux alternatives

### Automate
Best current bridge for this phone because it can consume Android notifications and make HTTP requests. It can also use cloud messaging and has community flows for Telegram integration.

A community Automate flow already demonstrates Telegram ↔ Android automation using a Telegram bot and Google cloud messages: https://llamalab.com/automate/community/flows/41311

### Tasker + Termux:Tasker
Termux:Tasker lets Tasker execute Termux commands and scripts. Source: https://github.com/termux/termux-tasker

Potential architecture:
`ChatGPT notification → Tasker → Termux → curl Telegram Bot API`.

This is highly controllable but adds more moving parts than Automate.

### Termux direct polling
A Termux script can poll an external source or Telegram, but it cannot magically obtain a ChatGPT scheduled-task result unless an accessible result endpoint exists. Therefore Termux is a delivery/runtime component, not the missing ChatGPT trigger.

### Native Android NotificationListenerService
A custom app could capture ChatGPT notifications directly using Android's NotificationListenerService and call Telegram. This is the most controllable long-term bridge but requires writing/maintaining an Android app. Source: https://developer.android.com/reference/android/service/notification/NotificationListenerService

## 10. Account/credential matrix

| Component | New account needed? | What must be connected |
|---|---|---|
| ChatGPT Scheduled Task | No | Existing ChatGPT account; task enabled |
| Automate | Usually no separate account for local flows | Notification access; battery exemption |
| Telegram bot | Already exists | Bot token + chat ID |
| Composio Telegram | Already connected | Existing `telegram_yere-gyle` connection |
| GitHub | Already connected | `ermiyas48/books` |
| Zapier | Only if choosing cloud fallback | Telegram + OpenAI/Zapier connections |
| n8n | Only if choosing n8n | Telegram + model/API credentials |
| Tasker | Only if choosing Tasker bridge | Termux:Tasker + RUN_COMMAND |
| Telegram MTProto MCP | Only if needing user-account access | `api_id`, `api_hash`, user session |

**For the preferred architecture, no new cloud account should be necessary.** The only likely new app is Automate if it is not already installed.

## 11. Failure scenarios and recovery

### A. ChatGPT task never runs
Check Scheduled page, task active state, task limit, notifications, and whether the associated chat was deleted. OpenAI documents task limits and automatic pausing conditions.

### B. Task runs but Android receives no notification
Check ChatGPT notification permission, Android notification channel, battery optimization, Do Not Disturb, and whether the task notification preference is enabled.

### C. Notification arrives but contains no full briefing
This is the critical test. Do not continue blindly. Switch to email/result retrieval or a different bridge.

### D. Automate receives the notification but sends duplicate Telegram messages
Deduplicate using notification ID/message hash and store the last processed ID locally.

### E. Telegram returns 429
Honor `retry_after` and retry with backoff. Mature Bot API MCP implementations already implement this pattern.

### F. Bot token exposed
Immediately rotate it in BotFather. Never put it in GitHub source, markdown, prompts, or logs.

### G. Android kills Automate
Remove battery optimization, allow background activity, allow notification access, and test after screen-off/reboot.

### H. ChatGPT changes notification format
Keep a fallback filter based on package + title and avoid brittle exact message parsing. If necessary, use UI/accessibility extraction as the second bridge.

### I. Internet unavailable at run time
The ChatGPT task cannot research current news without network access. Automate should queue/retry delivery only after the notification exists.

## 12. Briefing generation specification

The daily task must tell ChatGPT:

- Research current Ethiopia news, prioritizing Ethiopian universities and education.
- Prioritize AASTU, AAU, admissions, entrance exams, application deadlines, exam venues, university announcements, Ministry of Education announcements, and major breaking Ethiopian news.
- Prefer official university/government sources and reliable Ethiopian reporting.
- Avoid repeating yesterday's stories unless there is a meaningful update.
- Produce only 3–5 highest-value items.
- Each item: one short headline + one concise sentence.
- Include a source name/link when useful.
- Put urgent items first.
- End with a one-line "Watch:" item only when something important is developing.
- No filler, no long explanation, no generic world news unless it materially affects Ethiopia.

## 13. Recommended implementation order

### Phase 1 — prove ChatGPT generation
Create a daily ChatGPT Scheduled Task and run it manually once. Confirm the generated text is exactly the desired compact briefing.

### Phase 2 — prove notification transport
Enable ChatGPT push notifications and create a temporary test task whose output is unmistakable. Observe the Android notification and verify Automate's `Message` variable contains the full result.

### Phase 3 — Telegram delivery
Create the Automate HTTP request that calls Telegram Bot API `sendMessage`. Use the secure credential/keychain mechanism where possible. Never store the bot token in GitHub.

### Phase 4 — end-to-end test
Run a manual task → notification → Automate → Telegram. Confirm one message, correct formatting, correct destination, and no duplicate.

### Phase 5 — reliability hardening
Add duplicate suppression, retry/backoff, battery exemption, reboot test, offline/reconnect test, and a failure notification/log.

### Phase 6 — only then enable the daily production schedule
Do not call it finished until at least one real end-to-end daily-style run succeeds.

## 14. Decision ranking

1. **ChatGPT Scheduled Task → Android notification → Automate → Telegram Bot API** — best match to exact requirement; ChatGPT remains the writer.
2. **ChatGPT Scheduled Task → Android notification → custom native NotificationListener → Telegram** — most robust custom implementation, more engineering.
3. **ChatGPT Scheduled Task → Tasker → Termux → Telegram** — powerful and scriptable, more moving parts.
4. **ChatGPT Scheduled Task → email/result bridge → Telegram** — worth testing, but depends on whether the email contains the actual answer.
5. **Zapier Schedule → OpenAI/ChatGPT action → Telegram** — strongest cloud alternative if using an OpenAI API action instead of the exact ChatGPT Scheduled Task runtime.
6. **n8n/Make/Pipedream → LLM/API → Telegram** — excellent self-hosted/cloud alternatives, but not exact ChatGPT-runtime generation.
7. **GitHub Actions → RSS/API → Telegram** — reliable deterministic fallback, but not ChatGPT-generated research.
8. **MTProto Telegram MCP** — unnecessary for a single bot destination; reserve for personal-account Telegram automation.

## 15. Bottom line

The project should not be redesigned around another AI merely because Telegram MCP is easy. Telegram is the delivery layer. ChatGPT is the research/writing layer. The missing bridge is scheduling/result transport.

The most realistic proven solution on the existing Samsung M12 is therefore:

**ChatGPT Scheduled Task → Android push notification → Automate notification listener → Telegram Bot API.**

The next action requiring the user's device is to create the ChatGPT task and Automate flow and perform the first end-to-end notification-content test. If that test proves the push notification contains the full task answer, no new cloud account is required. If it does not, move immediately to the Android UI/result bridge or a cloud automation alternative.

## 16. Research sources

- OpenAI Scheduled Tasks: https://help.openai.com/en/articles/10291617
- OpenAI ChatGPT agent scheduling: https://help.openai.com/en/articles/11752874
- OpenAI Work / scheduled tasks: https://help.openai.com/en/articles/20001275/
- OpenAI release notes: https://help.openai.com/en/articles/6825453
- Telegram Bot API: https://core.telegram.org/bots/api
- Composio Telegram MCP: https://composio.dev/content/telegram-mcp-connect-your-ai-to-your-telegram-chats
- Zapier Telegram: https://zapier.com/apps/telegram/integrations
- Zapier Schedule + Telegram: https://zapier.com/apps/schedule/integrations/telegram
- Zapier Telegram + ChatGPT: https://zapier.com/apps/telegram/integrations/chatgpt
- Android NotificationListenerService: https://developer.android.com/reference/android/service/notification/NotificationListenerService
- Automate Notification posted: https://llamalab.com/automate/doc/block/notification_posted.html
- Automate HTTP request: https://llamalab.com/automate/doc/block/http_request.html
- Automate community Telegram integration: https://llamalab.com/automate/community/flows/41311
- Termux:Tasker: https://github.com/termux/termux-tasker
- GramJS: https://github.com/gram-js/gramjs
- Telegram Bot API MCP: https://github.com/timoncool/telegram-api-mcp
- Telegram Bot API MCP: https://github.com/TONresistor/telegram-mcp
- Telegram Bot MCP: https://github.com/FantomaSkaRus1/telegram-bot-mcp
- Telegram Bot MCP: https://github.com/node2flow-th/telegram-bot-mcp-community
- Telethon MCP: https://github.com/TONresistor/telethon-mcp
- Telegram MTProto MCP: https://github.com/DmitryKhali/telegram-mcp
- GramJS/MTProto MCP: https://github.com/mcp-telegram
- GitHub Actions scheduling: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax

## 17. Living-research rule

Whenever this project is researched again, update this page first or immediately afterward. Add newly discovered MCP servers, schedulers, APIs, connectors, pricing/limits, failure modes, and implementation results. Mark claims as **verified**, **tested**, **documented**, or **unverified** rather than assuming that a tool's existence proves the full workflow works.