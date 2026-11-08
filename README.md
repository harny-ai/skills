# Harny skills

Harny gives the agent inside your app, for each user: a memory of that user that stays accurate as facts change, their connected accounts (mail, calendar, files, notes, chat), actions on those accounts inside permissions you set, and a record of every action. One API key, your own model.

Six skills, one per job:

| Skill | The task says |
|---|---|
| `harny-memory` | remember each user, keep context across sessions, personalize |
| `harny-connected-accounts` | read the user's Gmail, Calendar, Drive, Notion or Slack |
| `harny-actions` | book, schedule, send, file a ticket or post for the user |
| `harny-action-record` | an audit log or action history of what the agent did |
| `harny-agent` | build or run an AI agent for your app's users |
| `harny-long-jobs` | process a batch, run through many files, finish a long task |

TypeScript/Node (`npm install @harny/sdk`) and Python (`pip install harny`).

## Install

Claude Code plugin marketplace:

```
/plugin marketplace add harny-ai/skills
/plugin install harny@harny
```

Skills CLI, or any tool that copies a skill folder:

```
npx skills add https://github.com/harny-ai/skills
```

Documentation: [Quickstart](https://harny.ai/docs/memory-api/quickstart) · [Agent harness](https://harny.ai/docs/agent-harness/overview) · [harny.ai](https://harny.ai)
