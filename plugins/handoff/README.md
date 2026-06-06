# handoff

A `/handoff` slash command that bridges work and home devices by writing to the [`mculp/handoff`](https://github.com/mculp/handoff) repo.

## What it does

`/handoff` figures out what you're handing off and pushes it to `mculp/handoff` so you (or another Claude instance on another device) can pick it up.

| Invocation | Behavior |
| :--- | :--- |
| `/handoff https://...` | Append the URL to the `## Links` section of `README.md` |
| `/handoff <text>` | Treat as a title; create `<slug>.md` entry and link from `## Commits` |
| `/handoff <slug> \| <title>` | Explicit slug + title for the entry file |
| `/handoff` (no args) | Infer from conversation context → git status → ask |

In every case, the command commits and pushes to `mculp/handoff` so the content is immediately available on other devices. It prints the canonical `https://github.com/mculp/handoff/blob/main/<slug>.md` URL after pushing.

## Install

```
/plugin marketplace add chunky-metro/marketplace
/plugin install handoff@chunky-metro
```

## Setup

The command expects a local clone of `mculp/handoff` at `~/repos/mculp/handoff`. If missing, the command will clone it on first run.

Requires `gh` CLI authenticated for the `mculp` GitHub account.

## Privacy guardrails

Before writing anything, the command scans for and redacts:

- API keys, tokens, passwords, secrets
- Internal hostnames or private network details
- Private filesystem paths (`~/.claude/...`)
- Wrapbook business specifics that haven't been publicly disclosed

If anything matches, it's replaced with `[REDACTED]` and the user is told what was redacted.
