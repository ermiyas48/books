# Composio + Telegram Reliability Audit — 2026-09-08

## Scope
Reviewed the available reconstructed ChatGPT history and saved workflow notes, then re-tested the active integrations relevant to the user's automation setup. Hyperbrowser and OpenRouter were explicitly excluded from this repair pass.

## Verified working
- Telegram Composio connection: active.
- Telegram bot identity check: successful.
- Telegram private destination reachability check: successful.
- Telegram message delivery: successful; a fresh university briefing was sent and the API returned a message ID.
- Google Calendar current-time lookup: successful.
- Google Calendar today's events lookup: successful; no events returned for 2026-09-08.
- Gmail unread+important last-24-hours check: successful; no matching messages on the selected default mailbox.
- Google Tasks cross-list check: successful; 3 incomplete tasks returned, including overdue items.
- GitHub authenticated access: active; repository/search tools execute.
- Notion and Notion MCP: active; private page creation/read-back capability is available.
- Manus, Perplexity, Mistral and other previously audited integrations remain classified according to the latest verified tests in the ChatGPT workflow notes.

## Telegram daily briefing

### Intended format
- 3–5 compact items maximum.
- Prioritize Ethiopian universities/higher education.
- Prioritize AASTU and AAU admissions, UAT/entrance exams, venues, deadlines, placement and cost-sharing.
- Include major Ethiopia education developments and only important breaking national news.
- Use fresh research and exact dates.
- Prefer official university/government sources.
- End with one priority/action line.

### Delivery path
The Composio Telegram delivery path is now directly verified. The bot and private destination are reachable, and a real briefing was delivered successfully on 2026-09-08.

### Important limitation
An unattended daily scheduler that performs fresh web research, generates the briefing, and then sends it through Telegram is NOT currently proven installed. Composio tool discovery did not expose a suitable first-party recurring AI workflow scheduler. Do not claim unattended daily automation is active until its trigger, research/generation step, and Telegram delivery are all tested end-to-end.

### Scheduler findings
- Composio trigger documentation supports recurring triggers in general, but the available connected-tool search did not return a ready-to-use scheduler for this exact AI + web research + Telegram workflow.
- Third-party scheduler candidates were discovered, but none is currently connected and none alone provides the complete research/generation pipeline.
- A previously documented Android Automate bridge remains a fallback: ChatGPT scheduled task -> notification content -> Automate -> Telegram. It must be tested on the actual Android app version because task notifications may contain only completion status rather than the generated result.

## Reliability rules
1. Never execute an app tool until its connection is confirmed ACTIVE.
2. For a new external workflow, search tools first and use a fresh Composio session.
3. If discovery is stale or misses an expected tool, retry the same search with `tool_search` and a fresh session.
4. Review the returned execution plan and complete schemas before execution.
5. Resolve IDs from prerequisite read actions; never guess IDs.
6. For Telegram, verify bot identity and destination before sending when the destination is uncertain.
7. For Telegram polling, never run multiple getUpdates consumers simultaneously; this can cause HTTP 409 conflicts. Prefer direct send for outbound briefing delivery.
8. For Telegram, use plain text unless formatting is necessary; malformed Markdown/HTML can cause HTTP 400 parsing errors.
9. For scheduled briefings, verify the trigger and perform at least one real end-to-end delivery test before calling the automation active.
10. Do not store or commit API keys, bot tokens, cookies, recovery codes, or other credentials.
11. Do not claim a connection is healthy merely because its account status says active; perform a representative real action.
12. Treat 401 as authentication failure, 403 as permission/scope failure, 404 as stale/wrong resource, 409 as concurrency/state conflict, 429 as rate limiting, and 5xx as provider/server failure; recover according to the specific cause.
13. For Gmail multi-account workflows, explicitly select the intended account rather than relying on an ambiguous default.
14. For calendar relative-date workflows, obtain the current time/timezone first and construct explicit RFC3339 windows.
15. For paginated APIs, continue until pagination is exhausted when completeness matters.

## Current unresolved items
- Unattended daily Telegram briefing: delivery is fixed and verified, but the complete autonomous scheduler is not yet proven.
- Hyperbrowser: explicitly skipped per user instruction.
- OpenRouter: explicitly skipped per user instruction.

## Security
Never place Telegram bot credentials or other secrets in this repository. Use the secure Composio connection. Any bot token previously exposed in chat should be treated as compromised and rotated in BotFather if it has not already been rotated.

## Source-of-truth rule
For future repairs, use the latest live Composio connection/tool tests plus this document. Do not rely solely on old 'active' labels or historical assumptions.
