---
name: harny-long-jobs
description: "Hands an agent a whole job bigger than one chat turn, with Harny's long mode: dozens to hundreds of items in (emails, requests, tickets, invoices), finished files out, on the developer's own model key. Use when asked to process a batch, run through many files, or finish a multi-step task without stalling."
license: Apache-2.0
compatibility: "Needs network access to api.harny.ai and a HARNY_API_KEY (the setup command in the skill creates one). Long mode needs the Python SDK."
---

# Harny: long jobs

Use Harny's long mode when an agent must finish a whole job bigger than one chat turn: dozens to hundreds of items in (emails, requests, tickets, invoices), finished files out. It is the same agent loop with a bigger budget and a folder to work in, on the developer's own model key. Long mode is Python only. Do not feed hundreds of items through one chat turn: it stalls or overspends.

## Steps

1. Use the app key the user entered in this plugin's `harny_app_key` setting; the plugin passes it to the shell as `HARNY_API_KEY`. If the setting is empty, run `harny init` (see Setup).
2. Put the input files in a workspace folder your deploy can write to. Finished files are written there.
3. Create the agent with `long_task=LongTaskConfig(workspace=...)` in its config (long mode is off without it).
4. Start the job with `run_task`. To resume after a stop, call it again with the same message.

**Setup.** Python: `pip install harny`. TypeScript or Node: `npm install @harny/sdk`. Key: run `harny init --name "<app name>" --about "<what it does>" --customers "<who uses it>"` once. It creates the app, saves `HARNY_API_KEY` (the app key, starts with `fsa_`) to `.env`, writes a Harny section into `AGENTS.md` and `CLAUDE.md`, and prints a claim link for the owner. If `HARNY_API_KEY` is already set, skip it. Never open, print or paste `.env`, and never put the key in browser or mobile code. Both SDKs read `HARNY_API_KEY` from the environment when no key is passed.

**Python** (3.10 or newer):

```python
from harny import HarnyClient
from harny.agent import Agent, AgentConfig, FileSessionStore
from harny.agent.long_task import LongTaskConfig

async with HarnyClient() as harny:
    agent = await Agent.create(
        harny=harny,
        provider="your-provider",
        model="your-model-id",
        config=AgentConfig(
            instructions="You work through a folder of tenant emails for the office manager.",
            long_task=LongTaskConfig(workspace="./jobs/renewals"),
        ),
        store=FileSessionStore("./data/sessions"),
    )
    result = await agent.run_task(user="demo-user", message="List the tenants whose lease ends within 45 days.")
```

**TypeScript / Node:** the TypeScript SDK has no long mode. Run the job from Python, or use the plain agent loop for one turn (skill harny-agent).

**Limits per turn (defaults).** 100 model calls, 3,600 seconds, up to 10 turns in a chain, spend cap 5,000,000 tokens.

**State and resume.** The task's state is kept in `.harny/` in the workspace. The same message resumes. A different message in the same folder is refused, and so is a second task running there at the same time.

**Model and key.** The model and the key are the developer's own: `provider=` and `model=` name the model, and the key is in the provider's usual environment variable; Harny never sees it. On a transient provider error the SDK retries the call twice; if it still fails, the turn stops and says why.

**Shell.** The folder and any shell are yours; Harny hosts nothing for a long job. `LongTaskConfig(shell=...)` takes your own executor, such as a container. `allow_local_shell` is off by default, and a local shell does not start while the process holds `HARNY_*` or `*_API_KEY` variables unless you set `local_shell_sees_keys=True`.

Check the answer against the files before you act on it.

**What agents get wrong**

- Sending hundreds of items through one chat turn instead of `run_task`.
- Looking for a TypeScript long mode. There is none.
- Running two tasks in one workspace folder, or changing the message when resuming.
- Giving the job a local shell with the keys in its environment.
- Calling `run_task` on an agent whose config has no `long_task`. It needs `AgentConfig(long_task=LongTaskConfig(...))`.

**Docs**

- https://harny.ai/docs/agent-harness/long-jobs
- https://harny.ai/long-jobs.md
- https://harny.ai/guides/run-long-jobs-with-an-agent.md
- https://harny.ai/docs/agent-harness/limits

The loop it runs on: harny-agent. Memory a job can read: harny-memory.
