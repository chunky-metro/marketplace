---
description: Hand off the current thing (URL, summary, file, or inferred context) to mculp/handoff so it's reachable on another device
argument-hint: [URL | <slug> | <title> | <title> | <empty>]
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion, WebFetch, Glob, Grep
---

You are running `/handoff $ARGUMENTS` to push content to the `mculp/handoff` repo so the user can access it on a different device (work ↔ home / phone / vox / pearl / mini.local).

The user's stated goal: **"I need to access it on a different device or network."** That goal dictates the default: commit and push. Do not skip the push.

## Resolve the working clone

The `mculp/handoff` repo should be cloned at `~/repos/mculp/handoff`. Before doing anything else:

```bash
if [ ! -d ~/repos/mculp/handoff/.git ]; then
  mkdir -p ~/repos/mculp
  gh repo clone mculp/handoff ~/repos/mculp/handoff
fi
cd ~/repos/mculp/handoff && git pull --ff-only origin main
```

Always `git pull --ff-only` first so you don't push stale state from another device.

## Dispatch by argument shape

Inspect `$ARGUMENTS` and pick one of the four flows below.

### Flow A — URL (`$ARGUMENTS` matches `^https?://`)

Single URL. Append to the `## Links` section of `README.md` and push.

1. (Optional, best-effort) Use WebFetch to grab the page `<title>`. If you get one in a few seconds, format as `[Title](URL)`. If WebFetch is slow or fails, just write the raw URL.
2. Open `~/repos/mculp/handoff/README.md`. Find the `## Links` section.
3. Append a new bullet at the end of the list: `- [Title](URL)` or `- <URL>`.
4. If the URL is already in `## Links`, tell the user and stop — no duplicate.
5. Commit message: `links: <hostname-of-url>`
6. Push to `origin/main`.
7. Print the README anchor: `https://github.com/mculp/handoff#links`

No new file. No `## Commits` entry.

### Flow B — Explicit slug + title (`$ARGUMENTS` contains `|`)

Format: `<slug> | <title>`. Original behavior from the previous handoff.md. Use when the user knows what they want to name the entry.

1. Parse `slug` and `title` from `$ARGUMENTS`. Normalize slug to kebab-case.
2. Read the README and the most recent 2 entries (`ls -t *.md | head -3`) to anchor voice. Only on first invocation per session.
3. Ask the user one focused question: **"What's the core thing future-you needs to know about this?"** Use the answer as the spine.
4. Run the privacy scan (see below) before writing.
5. Draft `<slug>.md` at the repo root following the style rules (see below).
6. Append a new numbered link to `## Commits` in `README.md`:
   `N. [<Title>](https://github.com/mculp/handoff/blob/main/<slug>.md)`
   Use the next available number.
7. Commit message: `handoff: <title>`
8. Push to `origin/main`.
9. Print the canonical URL: `https://github.com/mculp/handoff/blob/main/<slug>.md`

### Flow C — Title only (`$ARGUMENTS` is non-empty, no URL, no pipe)

Treat as the entry title. Generate a kebab-case slug from it, then proceed exactly like Flow B starting at step 2.

### Flow D — No arguments (`$ARGUMENTS` is empty)

Smart inference. Try the following in order; stop at the first that yields a clear thing to hand off.

**D1. Conversation context.** Has this conversation been working on something a Claude instance on another device would benefit from picking up? Signals:

- A research thread or problem-in-progress
- A partially-built thing with state worth preserving
- A decision tree or set of options under consideration
- A reference list (links, file paths, commands) the user accumulated

If yes, summarize it. Choose the format:

- **HTML visualization** if the content is naturally structural (a timeline, a tree of options, a comparison matrix, a flowchart). Self-contained single-file HTML, no external deps. Save as `<slug>.html` at the repo root. **Still add it to `## Commits`** with the same `[Title](...).html` link format.
- **Markdown** otherwise. Keep it ≤ 1 page (~50–80 lines). Capture: what's unique, current state, open questions, relevant links/files. Save as `<slug>.md`.

Anything unique is probably important — name dates, decisions, paths, and rationales. Don't hand off a generic recap; future-you already knows the basics.

**D2. Working directory.** If conversation context is thin, run `git status` in the user's primary working dir (`$PWD` or `~/repos`). If there's a single notable artifact — a ruby script, a markdown doc, a config file — that's probably the thing.

- Copy it into the handoff repo with its original filename (or rename to a clearer slug)
- If it's a single file < 500 lines, paste it inline in a markdown wrapper with context
- If it's bigger, link to it and summarize

**D3. Fall back.** If neither D1 nor D2 yields anything obvious, use AskUserQuestion to ask:

- "What did you mean to hand off?" with options like `[URL] / [The current conversation] / [A specific file] / [Cancel — typo]`

After choosing content, proceed through the same steps as Flow B from step 4 (privacy scan → draft → README link → commit → push → print URL).

## Privacy guardrails (run BEFORE writing any file)

Scan the content you're about to write for:

- API keys, tokens, passwords, OAuth secrets, JWT-shaped strings
- Email addresses other than `matt@culpepper.co`
- Internal hostnames (`*.internal`, `*.local` other than `mini.local` which the user has publicly named, internal IP ranges)
- Private filesystem paths (`~/.claude/...`, absolute `$HOME` paths) except where the user has explicitly shared them
- Wrapbook business specifics — financials, internal team names, customer names, unannounced features, internal infrastructure. Be conservative: when in doubt, redact.

For each match, replace with `[REDACTED: <category>]` and tell the user at the end what was redacted. Do not just silently strip — the user needs to know.

`mini.local` is fine to mention (user has publicly named it). `vox` and `pearl` as device nicknames are fine. The mculp/handoff repo is public-adjacent.

## Entry style rules (when creating a new `.md` entry)

Match the repo's existing voice. Read `README.md` and 1–2 recent entries on first run to confirm.

- One markdown file per topic, at repo root, named `<slug>.md`
- Top of file: `# <Title>` (the human-readable title)
- Second line: a short narrative hook — what triggered this entry and the date (ISO `YYYY-MM-DD`) it was captured
- Body organized with `## H2` sections — typical sections include `## What shipped`, `## What's different`, `## Open questions`, `## Next steps`, `## Links`
- Bullets and short paragraphs over walls of prose
- Code fences (```) for commands, URL templates, config snippets
- A `## Links` section at the bottom when there are external references
- **No frontmatter** — these are plain narrative files, not data
- **No emojis** unless the user explicitly asks
- The file must carry its own date context — there's no chronological log elsewhere

Existing entries average ~30–80 lines. Keep it tight.

## Push step (all flows except cancellation)

```bash
cd ~/repos/mculp/handoff
git add <files-you-touched>
git commit -m "<commit-message>"
git push origin main
```

Then print the canonical URL so the user can paste it into the other device:

- For new entries: `https://github.com/mculp/handoff/blob/main/<slug>.md` (or `.html`)
- For URL-only links: `https://github.com/mculp/handoff#links`

## What NOT to do

- Don't skip the push. Cross-device access is the entire point.
- Don't invent dates or events. If the user hasn't told you when something happened, ask.
- Don't link to private paths (`~/.claude/...`, internal hostnames). Entries should make sense to a future reader without access to the author's machine.
- Don't write a closing summary paragraph that just restates the file. End on the last useful section.
- Don't add the entry to any other index, sidebar, or table of contents — `README.md` is the only index.
- Don't include sensitive info — run the privacy scan first.
- Don't disclose Wrapbook business specifics that haven't been publicly disclosed.
- Don't include any emojis in entries.
- Don't force-push. Don't push from a stale clone — always `git pull --ff-only` first.
