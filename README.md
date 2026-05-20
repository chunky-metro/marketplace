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

## Layout

```
.claude-plugin/
  marketplace.json    # the catalog
README.md             # this file
LICENSE               # MIT
```

Plugins live in their own repos (`source: {source: "github", repo: "..."}`) rather than as subdirectories of this marketplace, so each plugin keeps its own versioning, issues, and release cycle.

## Adding a plugin

1. Plugin lives in its own repo under `chunky-metro/<plugin-slug>`.
2. Open a PR against this repo adding an entry to `.claude-plugin/marketplace.json`.
3. Bump `metadata.version` in the same PR.

See [Anthropic's plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces.md) for the full schema.

## Related

- [chunky-metro/fleetvoxes](https://github.com/chunky-metro/fleetvoxes) — private fleet-internal marketplace
