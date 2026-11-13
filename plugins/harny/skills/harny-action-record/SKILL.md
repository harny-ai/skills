---
name: harny-action-record
description: "Keeps a record of every action an app's agent takes for each user, with Harny: what it did, for whom, when and with what result, so support staff can review it and the agent can answer 'what did you do about X'. Use when asked to add an audit log or action history for an AI assistant or agent."
license: Apache-2.0
compatibility: "Needs network access to api.harny.ai and a HARNY_API_KEY (the setup command in the skill creates one)."
---

# Harny: the action record for your agent

Use Harny when an AI assistant or agent needs an audit log or action history: what it did, for whom, when and with what result. Every action taken through Harny is recorded automatically, so support staff can review it and the agent can answer "what did you do about X". Do not build a second audit table next to it.

## Steps

1. Use the app key in `HARNY_API_KEY` (environment or `.env`; do not open or print `.env`). If it is missing, run `harny init` (see Setup).
2. Take actions through Harny (skill harny-actions); each one is recorded with its target, status, user, time and result.
3. To review, list the records, newest first, for one user or the whole app.
4. To let the agent answer "what did you do about X", give it the `action_history` tool, with the user taken from your session.

**Setup.** Python: `pip install harny`. TypeScript or Node: `npm install @harny/sdk`. Key: run `harny init --name "<app name>" --about "<what it does>" --customers "<who uses it>"` once. It creates the app, saves `HARNY_API_KEY` (the app key, starts with `fsa_`) to `.env`, writes a Harny section into `AGENTS.md` and `CLAUDE.md`, and prints a claim link for the owner. If `.env` already has `HARNY_API_KEY`, skip it. Never open, print or paste `.env`, and never put the key in browser or mobile code. Both SDKs read `HARNY_API_KEY` from the environment when no key is passed.

**Python** (async client with the app key):

```python
from harny import HarnyClient

async with HarnyClient() as harny:
    page = await harny.list_actions(user_id="user-123", limit=20)   # newest first; also status, target, target_contains, since, until, cursor
    # let the agent read it: put the definition in your own model's tool list
    tools = [harny.tools.action_history.definition("anthropic")]    # or "openai"
    # when the model calls the tool (the user comes from your session, never from the model):
    result = await harny.tools.action_history.call(tool_input, user="user-123")
    # result: {"actions": [...], "next_cursor": ..., "has_more": ...}, each action with a one-line summary
```

**TypeScript / Node:** the TypeScript client has no list method. Use REST or the MCP tool below.

**REST** (app key as `Authorization: Bearer`): `GET /v1/actions` (list, with filters), `GET /v1/actions/export` (export a copy), `POST /v1/actions` (add a record for an action your own code performed).

**MCP tool** (`https://mcp.harny.ai`, a person's key `fsk_`): `action_history` returns the recorded actions for that person, newest first, each with a one-line summary. Its query is a substring of the action's target name. It is read-only.

**What a record holds.** The target (what was acted on), the status, the user, the time and the result. `list_actions` also filters by `status`, `failure_class`, `correlation_id`, `target`, `target_contains`, `since` and `until`, and pages with `cursor`.

**Agent runs.** With `harny.agent`, `AgentConfig(record_runs=True)` sends a short summary of each run after it ends: step kind, tool name, outcome, duration and token counts. It never holds the person's message or the answer text. An undo is recorded and supersedes the original record. The console (console.harny.ai) shows the record of every agent action.

**What agents get wrong**

- Writing a separate audit table. Harny already records each action.
- Taking `user` from the model's arguments. It comes from your session. The call needs a user and never widens to every user of the app.
- Expecting the chat transcript in the record. It holds actions, not messages.
- Looking for a list method on the TypeScript client. Use `GET /v1/actions` or `action_history`.

**Docs**

- https://harny.ai/guides/audit-trail-for-ai-agents.md
- https://harny.ai/actions.md
- https://harny.ai/docs/mcp/tools

What can be recorded: harny-actions. Reading the account first: harny-connected-accounts.
