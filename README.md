# chunky-metro marketplace

Public Claude Code plugin marketplace for the [Chunky Metro](https://github.com/chunky-metro) org.

## Add this marketplace

```
/plugin marketplace add chunky-metro/marketplace
```

Then browse and install plugins:

```
/plugin marketplace list chunky-metro
/plugin install <plugin-name>@chunky-metro
```

## Available plugins

| Plugin | Description |
| :----- | :---------- |
| [discord-fleet](https://github.com/chunky-metro/discord-fleet) | Fleet-customized Discord MCP plugin (fork of `claude-plugins-official/discord`). |
| [handoff](./plugins/handoff) | `/handoff` command that pushes URLs, summaries, or inferred context to `mculp/handoff` for cross-device pickup. |

## Layout

```
.claude-plugin/
  marketplace.json      # the catalog
plugins/                # bundled plugins (path source)
  <plugin-name>/
    .claude-plugin/
      plugin.json
    commands/ | agents/ | hooks/ | skills/ | ...
README.md               # this file
LICENSE                 # MIT
```

Plugins can be either **bundled** as subdirectories of `plugins/` (relative-path source — single repo, single PR ships everything) or **external** as their own `chunky-metro/<plugin-slug>` repo (github source — independent versioning and release cycle). Bundled is the default for small / single-author plugins; promote to external when a plugin grows its own contributor base or release cadence.

## Adding a plugin

**Bundled** (default):

1. Create `plugins/<plugin-slug>/` with `.claude-plugin/plugin.json`, plus whatever components it needs (`commands/`, `agents/`, `hooks/`, `skills/`).
2. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<plugin-slug>"` (string shorthand; relative paths must start with `./`).
3. Bump `metadata.version` in the same PR.

**External**:

1. Plugin lives in its own repo under `chunky-metro/<plugin-slug>`.
2. Open a PR against this repo adding an entry with `"source": {"source": "github", "repo": "chunky-metro/<plugin-slug>"}`.
3. Bump `metadata.version` in the same PR.

See [Anthropic's plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces.md) for the full schema.

## Related

- [chunky-metro/fleetvoxes](https://github.com/chunky-metro/fleetvoxes) — private fleet-internal marketplace
