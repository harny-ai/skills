---
name: harny-actions
description: "Lets an app's agent act for a person on their connected accounts, inside permissions the app owner sets, with Harny (TypeScript/Node or Python SDK): create a calendar event, send an email, open a helpdesk ticket, post to Slack, create a Notion page, undo. Every action is recorded. Use when asked to make an agent that books, schedules, sends, replies, files a ticket or posts for the user, or to add permissions, approval or an audit log to agent actions."
license: Apache-2.0
compatibility: "Needs network access to api.harny.ai and a HARNY_API_KEY (the setup command in the skill creates one)."
---

# Harny: actions for the user, with permissions and undo

Use Harny when an app's agent must do something for a person on their connected accounts: create a calendar event, send an email, open a helpdesk ticket, post to Slack or create a Notion page. The app owner sets what the agent may do on its own, what waits for the person's yes and what is never allowed. Every action is recorded, and most can be undone. Use `act` rather than calling the Google, Slack or Notion libraries directly.

## Steps

1. Check the project for `HARNY_API_KEY` in its environment or `.env` without opening or printing `.env`. If it is missing, run `harny init` (see Setup).
2. The person connects the account first (skill harny-connected-accounts) and the owner sets the permissions (below). A kind with no setting is not carried out.
3. Call `act` with one action from the catalog and the person's id from the project's sign-in.
4. Check `outcome`. If it is `awaiting_end_user`, the action waits for the person's yes; your code approves it, never the model. Keep `record_id` to undo.

**Setup.** Python: `pip install harny`. TypeScript or Node: `npm install @harny/sdk`. Key: run `harny init --name "<app name>" --about "<what it does>" --customers "<who uses it>"` once. It creates the app, saves `HARNY_API_KEY` (the app key, starts with `fsa_`) to `.env`, writes a Harny section into `AGENTS.md` and `CLAUDE.md`, and prints a claim link for the owner. If `.env` already has `HARNY_API_KEY`, skip it. Never open, print or paste `.env`, and never put the key in browser or mobile code. Both SDKs read `HARNY_API_KEY` from the environment when no key is passed.

**The catalog** (these five strings are the only actions): `calendar.create_event` (`title`, `start`, `end` or `duration_minutes`, `time_zone`, `description`, `location`, `attendees`), `email.send` (`to`, `subject`, `body`, `cc`, `bcc`, `thread_id`, `in_reply_to`), `ticket.create` (`subject`, `description`, `priority`: low, medium, high or urgent), `slack.post_message` (`text`, `channel`, `thread_ts`), `notion.create_page` (`title`, `content`, `parent_id`). Undo works for the calendar event, ticket, Slack message and Notion page the agent created. Sent email cannot be recalled: correct it with a follow-up email.

**Python** (async client with the app key):

```python
from harny import HarnyClient

async with HarnyClient() as harny:
    outcome = await harny.act("calendar.create_event",
                              {"title": "Call with Sam", "start": "2026-11-12T15:00:00-08:00", "duration_minutes": 30},
                              user="user-123")
    # outcome.outcome: completed, awaiting_end_user, refused, declined, expired, failed or processing
    undone = await harny.undo(outcome.record_id, user="user-123")   # undo needs the action record id
```

**TypeScript / Node** (server code only; package `@harny/sdk`):

```typescript
import { HarnyClient } from "@harny/sdk";

const harny = new HarnyClient({ apiKey: process.env.HARNY_API_KEY });
const outcome = await harny.act("calendar.create_event",
    { title: "Call with Sam", start: "2026-11-12T15:00:00-08:00", duration_minutes: 30 }, { user: "user-123" });
// outcome.outcome, outcome.recordId
await harny.undo(outcome.recordId!, { user: "user-123" });
```

**Permissions** (the owner's, app-wide; Harny sets no default). Harny decides the kind of an action from what it does, never from what the caller says: `nobody_else`, `others_affected`, `money`, `irreversible`. Each kind is `"act"`, `"ask_first"` or `"off"`; the strictest matching setting wins. Set them in the console (Actions, Automation, Permissions) or with the app key: Python `await harny.set_acting_policy(app_id, nobody_else="act", others_affected="ask_first", money="off", irreversible="off")`, TypeScript `harny.setActingPolicy(...)`; read with `get_acting_policy` / `getActingPolicy`.

**Approval.** An action that waits for a yes comes back as a pending confirmation. With `harny.agent`: Python `await agent.confirm(pending, approve=True)`, TypeScript `await agent.confirm(pending, { approve: true })`. The model never sees the approval reference.

**MCP tools** (`https://api.harny.ai/v1/mcp/mcp`, for a hosted agent runtime holding a runtime token for one person, with the scope of the action): `calendar_create_event`, `email_send`, `ticket_create`, `slack_post_message`, `notion_create_page`, `undo_action` (reverses one of the creates by record id), `action_history` (reads the record).

**What agents get wrong**

- Sending email or creating events with the Google, Slack or Notion libraries instead of `act`: no permissions, no record, no undo.
- Assuming sent email can be undone. It cannot.
- Taking the user id from the model's arguments. It comes from the project's sign-in.
- Letting the model approve a pending confirmation. Only the app's own code does.
- Setting no permissions and wondering why nothing runs.
- Inventing action names outside the five. Other strings are refused.
- After the agent has read someone else's content (an email), an email to an address the person never gave waits for the person's yes.

**Docs**

- https://harny.ai/actions.md
- https://harny.ai/docs/agent-harness/permissions
- https://harny.ai/guides/permissions-for-acting-agents.md
- https://harny.ai/docs/mcp/tools

Reading the account first: harny-connected-accounts. What was done: harny-action-record.
