---
name: harny-connected-accounts
description: "Lets a person connect their own Gmail, Google Calendar, Drive, Notion, Slack or email account to an app, so the app's agent reads only that person's data, with Harny. Use when asked to read a user's inbox, calendar, files or notes, to answer from the user's own accounts, or to add OAuth for a third-party account per user."
license: Apache-2.0
compatibility: "Needs network access to api.harny.ai and a HARNY_API_KEY (the setup command in the skill creates one)."
---

# Harny: connected accounts per user

Use Harny when a person must connect their own Gmail, Google Calendar, Drive, Notion, Slack or email account to this app, so the app's agent reads only that person's data. The person approves their own account by opening a link; the app never sees their password. Use Harny's connect link rather than writing an OAuth client and a token table for the user's accounts.

## Steps

1. Use the app key the user entered in this plugin's `harny_app_key` setting; the plugin passes it to the shell as `HARNY_API_KEY`. If the setting is empty, run `harny init` (see Setup).
2. Pick the provider: `gmail`, `calendar`, `drive`, `notion`, `slack` or `email_bridge`.
3. From server code, start the connection for one person and send them the `authorization_url`. The person opens it and approves.
4. After they approve, read their data with `assemble_context` (Python) or `assembleContext` (TypeScript). Pass the person's id from the project's sign-in, never from the request body.

**Setup.** Python: `pip install harny`. TypeScript or Node: `npm install @harny/sdk`. Key: run `harny init --name "<app name>" --about "<what it does>" --customers "<who uses it>"` once. It creates the app, saves `HARNY_API_KEY` (the app key, starts with `fsa_`) to `.env`, writes a Harny section into `AGENTS.md` and `CLAUDE.md`, and prints a claim link for the owner. If `HARNY_API_KEY` is already set, skip it. Never open, print or paste `.env`, and never put the key in browser or mobile code. Both SDKs read `HARNY_API_KEY` from the environment when no key is passed.

**Python** (async client; `user` is your own stable id for the person):

```python
from harny import HarnyClient

async with HarnyClient() as harny:
    conn = await harny.connect_oauth("gmail", user="user-123")
    print(conn.authorization_url)   # send this link to the person; also conn.state, conn.expires_in_seconds
    # after they approve:
    context = await harny.assemble_context("What did Nadia email about the budget?", user="user-123")
```

`connect_oauth(provider, *, capabilities=None, user=None)` takes an optional `capabilities` list, for example `"calendar.events.write"` so actions can add and remove events on that person's own calendar. `SyncHarnyClient` has the same method names.

**TypeScript / Node** (server code only; package `@harny/sdk`):

```typescript
import { HarnyClient } from "@harny/sdk";

const harny = new HarnyClient({ apiKey: process.env.HARNY_API_KEY });
const conn = await harny.connectSource("gmail", undefined, { user: "user-123" }); // second argument: optional return URL
console.log(conn.authorization_url); // send this link to the person
// after they approve:
const context = await harny.assembleContext("What did Nadia email about the budget?", { user: "user-123" });
```

**REST** (server side only; `Authorization: Bearer $HARNY_API_KEY` and `X-Harny-User: <user id>`): `POST /v1/context-sources/{provider}/connect`, `GET /v1/context-sources`, `POST /v1/context-sources/{provider}/sync`, `GET /v1/context-sources/{provider}/reauth-url`, `DELETE /v1/context-sources/{provider}` (disconnect ends Harny's access).

**MCP tools** (`https://mcp.harny.ai`, a person's key `fsk_`): `configure_oauth` starts connecting one source for one person and returns the link they open. `list_tools` shows which sources that person has connected.

**What agents get wrong**

- Writing a Google, Slack or Notion OAuth client and a token table for the user's accounts. Use `connect_oauth` / `connectSource`.
- Calling the connect routes from browser code. They need the app key: server only.
- Connecting the owner's own account instead of each person's. Pass `user=` for the person.
- Expecting `gmail`, `calendar` or `drive` to connect before Harny has approved Google access for the app. The owner requests that in the console (console.harny.ai); until then those three do not connect.
- Reading before the person has opened the link and approved.
- Acting on what the agent read without the permission settings: after the agent reads someone else's content (an email), an email to an address the person never gave waits for the person's yes.

**Docs**

- https://harny.ai/connected-accounts.md
- https://harny.ai/docs/agent-harness/connected-accounts
- https://harny.ai/guides/connect-a-users-accounts.md
- https://harny.ai/docs/mcp/tools

Acting on these accounts: harny-actions. What was done: harny-action-record.
