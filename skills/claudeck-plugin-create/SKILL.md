---
name: claudeck-plugin-create
description: "Scaffold a new Claudeck plugin with JS, CSS, and optional server routes. Requires Claudeck (npx claudeck) — the browser UI for Claude Code."
argument-hint: <plugin-name> <short description of what the plugin does>
tags: claudeck, plugin, tab-sdk, ui
prerequisites: "Claudeck must be installed (npx claudeck or npm i -g claudeck). See https://www.npmjs.com/package/claudeck"
---

# Create a new Claudeck Plugin

You are scaffolding a new tab-sdk plugin called "$ARGUMENTS".

Parse the arguments: the first word is the **plugin name** (kebab-case), the rest is the **description** of what the plugin should do.

## Plugin Locations

Claudeck supports two plugin directories:

| Directory | URL Path | Use When |
|-----------|----------|----------|
| `plugins/<name>/` (project) | `/plugins/<name>/` | Building a built-in plugin that ships with Claudeck |
| `~/.claudeck/plugins/<name>/` (user) | `/user-plugins/<name>/` | Creating a personal/user-installed plugin that persists across updates |

**Default: Create user plugins in `~/.claudeck/plugins/<name>/`** unless the user explicitly asks for a built-in plugin. User plugins are the recommended approach since they persist across npm upgrades and don't require modifying the Claudeck source code.

To resolve the user plugins directory, use `~/.claudeck/plugins/` (or `$CLAUDECK_HOME/plugins/` if the env var is set).

## Rules

1. The plugin name MUST be kebab-case (e.g. `my-plugin`, `code-metrics`)
2. All files go in the chosen plugin directory — NO other files need to be modified (no main.js, no style.css, no index.html)
3. The plugin is auto-discovered by the server at runtime via `GET /api/plugins`
4. Create at minimum: `client.js` and optionally `client.css`
5. If the plugin needs server-side API routes, also create `server.js` (for user plugins, requires `CLAUDECK_USER_SERVER_PLUGINS=true` in `~/.claudeck/.env`)
6. If the plugin needs persistent config, also create `config.json` (auto-copied to `~/.claudeck/config/` on first run)

## manifest.json (required)

Create `manifest.json` in the plugin directory:

```json
{
  "id": "<name>",
  "name": "<Title>",
  "version": "1.0.0",
  "description": "<Short description — max 120 chars>",
  "author": "<github-username>",
  "icon": "<emoji>",
  "hasServer": false,
  "minClaudeckVersion": "1.4.1"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Lowercase alphanumeric + hyphens only (`^[a-z0-9-]+$`). Must match directory name. |
| `name` | string | Yes | Human-readable plugin name |
| `version` | string | Yes | Semantic version (e.g. `1.0.0`) |
| `description` | string | Yes | Short description shown in marketplace (max 120 chars) |
| `author` | string | Yes | Author name or GitHub username |
| `icon` | string | No | Emoji or short SVG string |
| `hasServer` | boolean | No | Set `true` if plugin includes `server.js`. Default `false`. |
| `minClaudeckVersion` | string | No | Minimum Claudeck version required (e.g. `1.4.1`) |
| `homepage` | string | No | URL to plugin docs or repo |
| `keywords` | array | No | Search keywords (array of strings) |
| `authorGithub` | string | No | Author's GitHub username (used for profile link in marketplace) |

## Client JS file template

Create `client.js` in the plugin directory following this structure:

```javascript
// <Title> — Tab SDK plugin
// <Description of what the plugin does>
import { registerTab } from '/js/ui/tab-sdk.js';

registerTab({
  id: '<name>',
  title: '<Title>',
  icon: '<emoji or SVG>',       // emoji string (e.g. '📊') or SVG markup (12×12 recommended)
  lazy: true,                    // defer init() until first open (recommended)
  // position: 0,               // optional: 0-based insert index in tab bar
  // shortcut: 'Ctrl+Shift+X',  // optional: informational shortcut label

  init(ctx) {
    // ── Build DOM ──
    const root = document.createElement('div');
    root.className = '<name>-tab';
    root.style.cssText = 'display:flex;flex-direction:column;flex:1;overflow:hidden;';

    // Build the UI innerHTML here...

    // ── Context API ──
    // ctx.pluginId              — your plugin's ID string
    // ctx.on(event, fn)         — subscribe to event bus (returns unsubscribe fn)
    // ctx.off(event, fn)        — remove an event listener
    // ctx.emit(event, data)     — publish to the app event bus
    // ctx.getState(key)         — read from the reactive store
    // ctx.onState(key, fn)      — subscribe to store changes (returns unsubscribe fn)
    // ctx.api                   — full API module for fetch calls (see API section below)
    // ctx.getProjectPath()      — current project path (may be '' if none selected)
    // ctx.getSessionId()        — current session ID (string or null)
    // ctx.getTheme()            — current theme: 'dark' or 'light'
    // ctx.storage.get(key)      — read from plugin-scoped localStorage
    // ctx.storage.set(key, val) — write to plugin-scoped localStorage
    // ctx.storage.remove(key)   — remove from plugin-scoped localStorage
    // ctx.toast(msg, opts)      — show notification (opts: {duration, type: 'info'|'success'|'error'})
    // ctx.showBadge(count)      — show number badge on tab button
    // ctx.clearBadge()          — hide the badge
    // ctx.setTitle(text)        — update the tab button label at runtime
    // ctx.dispose()             — unsubscribe all listeners (auto-called on destroy)
    //
    // IMPORTANT: ALWAYS use ctx.getProjectPath() to read the project path.
    //   NEVER use document.getElementById('project-select') directly.
    // IMPORTANT: ALWAYS use ctx.on('projectChanged', fn) to react to project changes.
    //   NEVER add your own change listener to the project select element.

    return root;  // must return an HTMLElement
  },

  onActivate(ctx)   { /* tab became visible — ctx is passed */ },
  onDeactivate(ctx) { /* tab was hidden — ctx is passed */ },
  onDestroy(ctx)    { /* tab unregistered — cleanup — ctx is passed */ },
});
```

**IMPORTANT**: Use absolute import paths (e.g. `'/js/ui/tab-sdk.js'`, `'/js/core/api.js'`) since plugins are served from `/plugins/<name>/client.js` (built-in) or `/user-plugins/<name>/client.js` (user). Absolute paths work identically for both locations.

### registerTab Config Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique tab identifier (used as `data-tab` value) |
| `title` | string | No | Tab button label (defaults to formatted `id`) |
| `icon` | string | No | Emoji (e.g. `'📊'`) or SVG markup (12×12 recommended) |
| `position` | number | No | 0-based insert index in the tab bar. Omit to append at end |
| `shortcut` | string | No | Informational keyboard shortcut label |
| `lazy` | boolean | No | If `true`, `init()` deferred to first open. Recommended. Default `false` |
| `init(ctx)` | function | Yes | Build your UI. Receives context object. **Must return an HTMLElement.** |
| `onActivate(ctx)` | function | No | Called each time tab becomes visible. Receives `ctx`. |
| `onDeactivate(ctx)` | function | No | Called when another tab becomes active. Receives `ctx`. |
| `onDestroy(ctx)` | function | No | Called when plugin is disabled/removed. Receives `ctx`. |

### Context API (ctx)

The `ctx` object passed to `init()` and all lifecycle hooks:

| Method | Description |
|--------|-------------|
| `ctx.pluginId` | Your plugin's ID string |
| `ctx.on(event, fn)` | Subscribe to the app event bus. Returns an unsubscribe function. |
| `ctx.off(event, fn)` | Remove a specific event listener |
| `ctx.emit(event, data)` | Publish to the app event bus |
| `ctx.getState(key)` | Read from the reactive store |
| `ctx.onState(key, fn)` | Subscribe to store key changes. Returns an unsubscribe function. |
| `ctx.api` | Full API module — all `fetch()` helpers (see API section) |
| `ctx.getProjectPath()` | Current project path (string, may be `''` if none selected) |
| `ctx.getSessionId()` | Current session ID (string or `null`) |
| `ctx.getTheme()` | Current theme: `'dark'` or `'light'` |
| `ctx.storage.get(key)` | Read from plugin-scoped localStorage (namespaced to `claudeck-plugin-{id}-{key}`) |
| `ctx.storage.set(key, value)` | Write to plugin-scoped localStorage |
| `ctx.storage.remove(key)` | Remove from plugin-scoped localStorage |
| `ctx.toast(msg, opts)` | Show notification. `opts: {duration, type}` where type is `'info'`, `'success'`, or `'error'` |
| `ctx.showBadge(count)` | Show a number badge on the tab button. Pass `0` to hide. |
| `ctx.clearBadge()` | Hide the badge |
| `ctx.setTitle(text)` | Update the tab button label at runtime |
| `ctx.dispose()` | Unsubscribe all event/state listeners (auto-called on destroy) |

**Auto-cleanup:** All listeners registered via `ctx.on()` and `ctx.onState()` are automatically unsubscribed when the plugin is destroyed.

### Available Events

Subscribe via `ctx.on(event, fn)`:

| Event | Payload | Description |
|-------|---------|-------------|
| `ws:message` | `msg` (object) | Every WebSocket message — see [message types](#wsmessage-types) below |
| `ws:connected` | — | WebSocket first connected |
| `ws:reconnected` | — | WebSocket reconnected after drop |
| `ws:disconnected` | — | WebSocket connection lost |
| `projectChanged` | `path` (string) | User switched to a different project |
| `rightPanel:tabChanged` | `tabId` (string) | A different tab became active |
| `rightPanel:opened` | `tabId` (string) | The right panel was opened |

**Most useful:** `ws:message` (react to live streaming/tool calls), `projectChanged` (reload data on project switch).

**Note:** Use `ctx.on` for events (things that *happen*), use `ctx.onState` for state (values that *change*).

### ws:message Types

The `msg` object from `ws:message` has a `type` field:

| Type | Description | Key Fields |
|------|-------------|------------|
| `session` | New session created | `msg.sessionId` |
| `text` | Streaming text chunk | `msg.text` |
| `tool` | Tool call started | `msg.name`, `msg.input` |
| `tool_result` | Tool call finished | `msg.content`, `msg.isError` |
| `result` | Query completed | `msg.text` |
| `done` | Session finished | — |
| `error` | Error occurred | `msg.error` |
| `aborted` | User aborted | — |
| `permission_request` | Tool needs approval | `msg.tool`, `msg.input` |

### Available State Keys

Read via `ctx.getState(key)`, subscribe via `ctx.onState(key, fn)`:

| Key | Type | Description |
|-----|------|-------------|
| `sessionId` | string \| null | Currently active session ID |
| `view` | string | Current view: `'home'` or `'chat'` |
| `parallelMode` | boolean | Whether parallel mode is active |
| `projectsData` | array | All registered projects `[{name, path}, ...]` |
| `sessionTokens` | object | Token usage: `{input, output, cacheRead, cacheCreation}` |
| `prompts` | array | Saved prompt templates |
| `workflows` | array | Saved workflows |
| `agents` | array | Agent definitions |
| `ws` | WebSocket \| null | The live WebSocket instance |
| `streamingCharCount` | number | Characters received in current stream |
| `notificationsEnabled` | boolean | Whether browser notifications are on |
| `attachedFiles` | array | Files attached to current message |

**Most useful:** `sessionId` (reload data on session switch), `projectsData` (know when projects are loaded).

**Timing note:** When `init()` runs, `projectsData` or `sessionId` may not be populated yet. Use `ctx.onState()` to react when data arrives.

### Common API Functions (ctx.api)

**Projects & Files:**

| Function | Description |
|----------|-------------|
| `ctx.api.fetchProjects()` | List all registered projects |
| `ctx.api.fetchFiles(path)` | List files in a directory |
| `ctx.api.fetchFileContent(basePath, filePath)` | Read a file's content |
| `ctx.api.writeFileContent(basePath, filePath, content)` | Write content to a file |
| `ctx.api.fetchFileTree(basePath, dir)` | Get recursive file tree |
| `ctx.api.searchFiles(basePath, query)` | Search for files by name |
| `ctx.api.browseFolders(dir)` | Browse folder structure |
| `ctx.api.execCommand(command, cwd)` | Execute a shell command |

**Sessions:**

| Function | Description |
|----------|-------------|
| `ctx.api.fetchSessions(projectPath)` | List sessions for a project |
| `ctx.api.fetchMessages(sessionId, opts)` | Get messages for a session |

**Stats:**

| Function | Description |
|----------|-------------|
| `ctx.api.fetchStats(projectPath)` | Get usage stats |
| `ctx.api.fetchHomeData()` | Get home dashboard data |
| `ctx.api.fetchAccountInfo()` | Get account details |

### Project-aware Plugin Pattern

If your plugin needs to react to project changes, follow this pattern:

```javascript
init(ctx) {
  const root = document.createElement('div');

  function loadData() {
    const project = ctx.getProjectPath();
    if (!project) return;
    fetch(`/api/plugins/<name>/data?project=${encodeURIComponent(project)}`)
      .then(r => r.json())
      .then(data => { /* render */ });
  }

  ctx.on('projectChanged', loadData);
  ctx.onState('projectsData', () => loadData());
  loadData(); // try immediately — may be empty on first call

  return root;
}
```

## Client CSS file template

Create `client.css` in the plugin directory with styles scoped to `.<name>-tab` to avoid conflicts:

```css
/* <Title> — Tab SDK plugin styles */

.<name>-tab {
  font-family: var(--font-sans);
  color: var(--text);
}
```

**IMPORTANT:** Prefix ALL CSS classes with your plugin ID to avoid collisions (e.g. `.my-plugin-header`, `.my-plugin-list`). Do not use broad selectors like `body`, `*`, or bare element selectors.

### CSS Design Tokens

Use these CSS custom properties from the app theme:

**Colors:**
- `var(--text)`, `var(--text-secondary)`, `var(--text-dim)` — text hierarchy
- `var(--accent)`, `var(--accent-dim)` — green accent (primary action)
- `var(--success)`, `var(--warning)`, `var(--error)` — status colors
- `var(--purple)`, `var(--user)`, `var(--cyan)`, `var(--amber)` — additional accents

**Backgrounds:**
- `var(--bg)`, `var(--bg-secondary)`, `var(--bg-tertiary)`, `var(--bg-elevated)` — background hierarchy

**Borders:**
- `var(--border)`, `var(--border-subtle)` — border colors

**Typography:**
- `var(--font-display)` — Chakra Petch (headings)
- `var(--font-sans)` — Outfit (body text)
- `var(--font-mono)` — JetBrains Mono (code)

**Layout:**
- `var(--radius)` (4px), `var(--radius-md)` (8px), `var(--radius-lg)` (12px)

**Effects:**
- `var(--glow)` — subtle green glow
- `var(--shadow-sm)`, `var(--shadow-md)` — shadows
- `var(--ease-smooth)` — smooth easing

### Common CSS Patterns

```css
/* Plugin container — fill the tab pane */
.<name>-tab {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 16px;
  font-family: var(--font-sans);
  color: var(--text);
}

/* Scrollable list */
.<name>-list {
  flex: 1;
  overflow-y: auto;
}

/* Action button */
.<name>-btn {
  padding: 6px 16px;
  border-radius: var(--radius-md);
  background: var(--accent);
  color: #fff;
  border: none;
  cursor: pointer;
  font-family: var(--font-sans);
  font-size: 12px;
}
.<name>-btn:hover {
  filter: brightness(1.1);
}
```

## Server JS file template (optional)

Create `server.js` in the plugin directory if your plugin needs API endpoints. The router is auto-mounted at `/api/plugins/<name>/`:

```javascript
import { Router } from "express";

const router = Router();

router.get("/", (req, res) => {
  res.json({ status: "ok" });
});

export default router;
```

**Note for user plugins:** Server-side routes require `CLAUDECK_USER_SERVER_PLUGINS=true` in `~/.claudeck/.env`. This is disabled by default for security.

To use config files, import `configPath`. The import path depends on the plugin location:
- **Built-in plugins** (`plugins/<name>/`): `import { configPath } from "../../server/paths.js";`
- **User plugins** (`~/.claudeck/plugins/<name>/`): Use the server's config path directly via the route context, or read/write files using `os.homedir()`:

```javascript
import { readFileSync, writeFileSync } from "fs";
import { join } from "path";
import { homedir } from "os";

const cfgDir = process.env.CLAUDECK_HOME || join(homedir(), ".claudeck", "config");
const cfgFile = join(cfgDir, "<name>-config.json");
```

### Calling Your Own Server Routes from client.js

```javascript
async function loadMyData() {
  const res = await fetch('/api/plugins/<name>/data');
  return res.json();
}

async function saveMyData(data) {
  await fetch('/api/plugins/<name>/data', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
}
```

## Config JSON file template (optional)

Create `config.json` in the plugin directory with default values. It will be auto-copied to `~/.claudeck/config/` on first run:

```json
{
  "enabled": false,
  "settings": {}
}
```

## Available imports

From `/js/core/`:
- `store.js` — `getState(key)`, `setState(key, val)`, `onState(key, fn)`
- `events.js` — `on(event, fn)`, `emit(event, data)`
- `dom.js` — `$` (cached DOM query map)
- `constants.js` — `CHAT_IDS`, `BOT_CHAT_ID`
- `api.js` — all fetch helpers (`fetchSessions`, `fetchFileTree`, `execCommand`, etc.)
- `utils.js` — `escapeHtml()`, `getToolDetail()`, `formatBytes()`, etc.

From `/js/ui/`:
- `tab-sdk.js` — `registerTab()`, `unregisterTab()`, `getRegisteredTabs()`
- `commands.js` — `registerCommand()` for slash commands
- `formatting.js` — `renderMarkdown()`, `highlightCodeBlocks()`
- `parallel.js` — `getPane()`, `panes`
- `right-panel.js` — `openRightPanel()`, `toggleRightPanel()`

**Note:** Most plugins only need `tab-sdk.js`. The `ctx` object provides access to events, state, and the API module without direct imports. Only import from `/js/core/` or `/js/ui/` if you need functionality not exposed through `ctx`.

## What to build

Now implement the plugin based on the user's description. Build a fully functional plugin — not just a skeleton. Include:
- Complete DOM structure with proper layout
- Event handlers and interactivity
- Real-time updates via `ctx.on('ws:message', ...)` if relevant
- Session awareness via `ctx.onState('sessionId', ...)` if relevant
- Project awareness via `ctx.getProjectPath()` and `ctx.on('projectChanged', ...)` if the plugin loads project-scoped data
- Proper CSS styling matching the app's dark terminal aesthetic using design tokens
- Badge counts via `ctx.showBadge()` where meaningful
- Plugin-scoped storage via `ctx.storage` for persistent data

**CRITICAL**: Never access the project select DOM element directly. Always use `ctx.getProjectPath()` and `ctx.on('projectChanged', fn)` from the tab SDK context.

## After creating the files

1. Tell the user the plugin is ready
2. Remind them to reload the browser — the plugin will be auto-discovered
3. Show the file paths that were created
4. If it's a user plugin with `server.js`, remind them to set `CLAUDECK_USER_SERVER_PLUGINS=true` in `~/.claudeck/.env` and restart the server
5. Mention which directory was used (built-in vs user) and why
