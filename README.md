# Claudeck Skills

Community skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), purpose-built for [Claudeck](https://www.npmjs.com/package/claudeck) — the browser UI for Claude Code.

## Installation

```bash
npx skills add https://github.com/hamedafarag/claudeck-skills
```

> **Prerequisites:** Claudeck must be installed (`npx claudeck` or `npm i -g claudeck`). See the [Claudeck npm page](https://www.npmjs.com/package/claudeck) for setup instructions.

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **Claudeck Plugin Create** | `/claudeck-plugin-create <name> <description>` | Scaffold a new Claudeck plugin with JS, CSS, and optional server routes using the tab-sdk |

## How It Works

Each skill is a self-contained `SKILL.md` file inside `skills/<skill-name>/`. When installed, Claude Code loads these skills and makes them available as slash commands in your conversation.

### Claudeck Plugin Create

Generates a fully functional Claudeck tab-sdk plugin with:

- **`client.js`** — UI component registered via `registerTab()`, with DOM structure, event handlers, and real-time WebSocket updates
- **`client.css`** — Scoped styles using the app's CSS custom properties (dark terminal aesthetic)
- **`server.js`** *(optional)* — Express router auto-mounted at `/api/plugins/<name>/`
- **`config.json`** *(optional)* — Default configuration, auto-copied to `~/.claudeck/config/`

Plugins are created in `~/.claudeck/plugins/` by default (user plugins), so they persist across Claudeck updates. Simply reload the browser after creation — no manual registration required.

## Contributing

Have a skill idea for Claudeck? Contributions are welcome!

1. Create a new directory under `skills/<your-skill-name>/`
2. Add a `SKILL.md` following the [Claude Code skills format](https://docs.anthropic.com/en/docs/claude-code/skills)
3. Open a pull request

## License

MIT
