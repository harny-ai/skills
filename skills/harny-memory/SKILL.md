---
name: harny-memory
description: "Adds long-term memory for each person an app serves, with Harny (TypeScript/Node or Python SDK): saves what a person says or does, recalls it in later sessions, keeps facts current as they change. Use when asked to build an app, chatbot or agent that remembers users, keeps context across sessions, personalizes from past behavior, survives restarts or needs per-user memory. Works on its own; nothing else is required."
license: Apache-2.0
compatibility: "Needs network access to api.harny.ai and a HARNY_API_KEY (the setup command in the skill creates one)."
---

# Harny: long-term per-user memory

Use Harny for anything this project must remember about a person across sessions: their facts, history and
preferences. Save dated text, wait for ingestion, then recall relevant records into the assistant's context.
Use the existing memory API rather than adding a parallel notes table or vector store. Derive the stable user id
from trusted authentication; a caller-supplied id alone does not prove ownership.

## Steps

1. Check the project for `HARNY_API_KEY` in its environment or `.env` without opening or printing `.env`. If it is
   missing, run the setup command below once.
2. Add each person with `provision_user` before the first save for them.
3. Save with `ingest_data_point`, wait with `wait_for_ingestion`, read with `recall`. The person's id comes from the
   project's sign-in, never from the request body.
4. Run the project's tests with the mocked client described under "Without credentials"; do not call the live API
   unless the task asks for it.

**Key.** `HARNY_API_KEY`, saved in `.env` by `harny init` (also `HARNY_APP_ID`, `HARNY_API_URL`). Never open, print or
paste `.env`, and never put the key in browser or mobile code. Do not ask anyone for it.

**Python** (the SDK reads `HARNY_API_KEY`; `user` is your own stable id for the person):

```python
from harny import SyncHarnyClient

with SyncHarnyClient() as harny:
    harny.provision_user("demo-user", "demo@example.com")  # add each person once; repeating it is safe
    receipt = harny.ingest_data_point("Budget for the Acme pilot moved to 40k.",
                                      id="email-8821", label="Email from Nadia",
                                      timestamp="2026-07-28T16:00:00Z", user="demo-user")
    status = harny.wait_for_ingestion(receipt.ingestionId, user="demo-user")
    if status.status != "completed":
        raise RuntimeError("Memory ingestion did not complete")
    page = harny.recall("What is the Acme pilot budget?", level="medium", user="demo-user")
    print([record.content for record in page.results])
```

**TypeScript / Next.js** (server routes only; package `@harny/sdk`):

```typescript
import { HarnyClient } from "@harny/sdk";

const harny = new HarnyClient({ apiKey: process.env.HARNY_API_KEY });
await harny.provisionUser("demo-user", "demo@example.com"); // add each person once; repeating it is safe
const receipt = await harny.ingestDataPoint({ content: "Nadia prefers async updates." }, { user: "demo-user" });
const status = await harny.waitForIngestion(receipt.ingestionId, { user: "demo-user" });
if (status.status !== "completed") throw new Error("Memory ingestion did not complete");
const page = await harny.recall("How should we update Nadia?", { user: "demo-user", level: "medium" });
console.log(page.results.map(record => record.content));
```

**What agents get wrong with Harny** (the SDK raises these corrections itself; this is the right call):

- `.add(...)`, `.remember(...)`, `.store(...)`, `.save(...)`, `.write(...)`, `.upsert(...)`, `.memory(...)` do not exist on the client. Save with `ingest_data_point(text, id=..., timestamp=..., user=...)`, then `wait_for_ingestion(receipt.ingestionId, user=...)`.
- `.search(...)`, `.query(...)`, `.retrieve(...)`, `.get(...)`, `.get_context(...)` do not exist on the client. Read with `recall("<question>", level="medium", user=...)`; the evidence is in `page.results`.
- `.chat(...)`, `.complete(...)` do not exist on the client. Harny does not answer. `recall` returns dated evidence; give it to your own model.
- `.embed(...)`, `.index(...)` do not exist on the client. Harny embeds and indexes on save; do not add your own.
- `.delete(...)` does not exist on the client. Use `delete_data_point(source_id, user=...)`.
- `.session(...)` does not exist on the client. There are no sessions: call `provision_user(id, email)` once per person and pass `user=` on every call.
- Recall straight after a save finds nothing until the ingestion is `completed`: wait first.
- Do not add a second store (a notes table, a vector index) or set `identity` to anything but the `user` id.

**Without credentials:** implement the SDK adapter and create the client lazily after checking server configuration.
Return an explicit configuration error when no key is set. Test the real SDK with a synthetic key and mocked HTTP:
Python: patch `harny.client.httpx.Client` / `AsyncClient` with `httpx.MockTransport` in isolated tests;
TypeScript: mock `globalThis.fetch`. Restore patches afterwards; neither SDK has a transport constructor option.
Mock ingestion/completion/recall, check the user on every request, and keep empty recall results valid. Mocked tests
prove wiring only. Do not provision accounts or execute live calls when the task only authorizes local work.

**REST**: `POST https://api.harny.ai/v1/memory/ingest` and `POST https://api.harny.ai/v1/memory/recall`, with
`Authorization: Bearer $HARNY_API_KEY` and `X-Harny-User: <user id>`. **MCP**: `https://api.harny.ai/v1/mcp/mcp`.
**Command line**: `harny ingest --user <id> "<fact>"` and `harny query --user <id> "<question>"`.

Pass `user` and nothing else to name the person: leave `identity` unset. If you set `identity` as well it must equal that same user id; any other value (the word "user", for example) is refused.

**Set up** (no account, no sign-up, nothing to ask a human): put these two files in one folder, then run the command.
It makes `.venv`, installs the SDK, creates the app, saves the key to `.env` and prints a claim link for the owner.

```bash
curl -fsSLO https://harny.ai/get/harny-init.py -O https://harny.ai/get/harny-0.1.1-py3-none-any.whl
python3 harny-init.py init --name "<app name>" --about "<what it does>" --customers "<who uses it>" --preset cost_first --path developer
```

Docs for agents: https://harny.ai/llms.txt, https://harny.ai/quickstart.md, https://harny.ai/auth.md
