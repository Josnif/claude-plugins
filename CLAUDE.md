# plugins (josnif-plugins marketplace)

A Claude Code plugin marketplace that will be published on GitHub for anyone to use. Read `HANDOFF.md` first for the history and the next steps.

## Conventions

- Layout: `.claude-plugin/marketplace.json` lists the plugins; each plugin lives in `plugins/<name>/` with `.claude-plugin/plugin.json` and `skills/<skill>/SKILL.md`.
- Skills must stay **project-agnostic**: detect toolchains, never hard-code pnpm, a specific repo, or company names.
- Keep `version` in sync between `marketplace.json` and the plugin's `plugin.json` when bumping.
- Never commit or push. The user makes all commits themselves.
- Skills must never post to GitHub (comments or issues) without explicit confirmation; they produce text to paste.
