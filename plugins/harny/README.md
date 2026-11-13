# Harny

Per-user memory, connected accounts, permitted actions and the action record for your agents.

Harny gives the agent inside your app, for each user: a memory of that user that stays accurate as facts change, their connected accounts (mail, calendar, files, notes, chat), actions on those accounts inside permissions you set, and a record of every action. One API key, your own model. The agent loop runs on your key in TypeScript or Python, and the Python harness also runs long jobs. A coding agent can set it up for a business with no account: one command makes the app, saves its key where you never read it, and prints a link the owner opens later to claim the app.

This plugin is six skills and nothing else. It has no hooks, scripts, commands or servers.

| Skill | Claude uses it when the task is |
|---|---|
| harny-memory | an app that remembers each of its users across sessions |
| harny-connected-accounts | letting a user connect their own Gmail, Calendar, Drive, Notion, Slack or email |
| harny-actions | an app's agent acting for a user inside permissions you set, with undo |
| harny-action-record | an audit log or action history for an app's agent |
| harny-agent | the agent loop for an app's users, on your own model key |
| harny-long-jobs | a job bigger than one turn: many items in, finished files out (Python) |

## What it runs and sends

- The plugin sends nothing. The skills are text that Claude reads. Its one start-of-session line runs only if you enter the optional Harny app key in the plugin settings (stored in your system's secure storage): it hands the key to the shell as `HARNY_API_KEY`.
- When Claude follows a skill, it writes code in your project. That code calls https://api.harny.ai with your own `HARNY_API_KEY`.
- The setup step is `pip install harny` (Python) or `npm install @harny/sdk` (TypeScript), then `harny init`.
- `harny init` creates the app with no account, saves the key to `.env`, adds rules that keep assistants from reading `.env`, writes a short Harny section into `AGENTS.md` and `CLAUDE.md`, and prints a claim link for the owner. With `--no-agent-files` it leaves `AGENTS.md` and `CLAUDE.md` unchanged.
- The memory belongs to your app's own users and is written by your app's code. Nothing from your conversation with Claude is saved to Harny by this plugin.

## Install

```
/plugin marketplace add harny-ai/skills
/plugin install harny@harny
```

Apache-2.0 · https://harny.ai · support@harny.ai
