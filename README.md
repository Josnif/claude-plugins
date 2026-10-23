# josnif-plugins

A Claude Code plugin marketplace.

| Plugin | What it does |
|---|---|
| [pr-review-kit](plugins/pr-review-kit) | Review a PR or branch and get a merge verdict plus one copy-paste markdown comment. Re-reviews track every earlier item, and follow-ups become ready-to-file issues. |

## Install

```text
/plugin marketplace add <github-user>/<this-repo>
/plugin install pr-review-kit@josnif-plugins
```

## Layout

```
.claude-plugin/marketplace.json     # lists the plugins in this repo
plugins/<name>/.claude-plugin/plugin.json
plugins/<name>/skills/<skill>/SKILL.md
```

## License

MIT
