# Rashinban for AI — User Guide

## What This Is

Rashinban for AI is a Chrome extension that extracts actionable UI elements (CSS selectors, coordinates, and states) from web pages and passes them to AI agents through a local MCP server. Because the AI agent does not need to parse screenshots or full DOM trees, this cuts down on token consumption and response time.

The project has two parts:

- **Chrome extension** (`chrome-extension/`): monitors pages and extracts UI elements
- **Local MCP server** (`mcp-server/`): bridges the extension and your AI agent

## Requirements

- Node.js (installed)
- Google Chrome
- An MCP-compatible local AI agent environment (e.g. LM Studio)

## Installation

### 1. Load the Chrome extension

1. Open `chrome://extensions` in Chrome
2. Turn on "Developer mode" (top right)
3. Click "Load unpacked"
4. Select the `chrome-extension` folder

### 2. Install the MCP server's dependencies

Run the following from the project folder:

```powershell
cd "mcp-server"
npm install
```

### 3. Start the MCP server

```powershell
node server.js
```

A successful start looks like this:

```
[Rashinban] === セキュリティ設定 ===
[Rashinban] 認証トークン: <a token string appears here>
[Rashinban] 承認済み拡張ID: （初回接続で自動登録）
[Rashinban] =======================
[Rashinban] WebSocketサーバー起動: ws://127.0.0.1:3765
[Rashinban] MCPサーバー起動: http://127.0.0.1:3766/mcp
```

(The server's log lines are in Japanese; the lines you actually need are the ones showing `ws://127.0.0.1:3765` and `http://127.0.0.1:3766/mcp`, which confirm the server started.)

**Copy the displayed auth token — you'll need it in the next step.**

> The server uses two ports. If either conflicts with another application:
> - **3765** (extension ↔ server WebSocket): change `WS_PORT` in `mcp-server/server.js` and `WS_URL` in `chrome-extension/content.js` to the same new number, then restart the server and reload the extension.
> - **3766** (AI agent ↔ server HTTP): change `HTTP_PORT` in `mcp-server/server.js` and update the connection URL in each AI agent's config accordingly.

### 4. Set the token in the extension

1. Open `chrome://extensions`, find Rashinban for AI, and click "Details"
2. Open "Extension options"
3. Paste the token you copied and save

After saving the token in the extension, **keep the server running**. As of v0.5.0, the server is a persistent process: AI agents no longer spawn it themselves — they connect over HTTP to the server you started. This also means **multiple AI agents can connect and use the tool at the same time**.

### 5. Connect your AI agent(s)

The connection URL is the same for every client: `http://127.0.0.1:3766/mcp`

**LM Studio**: open `C:\Users\<username>\.lmstudio\mcp.json` (create it if it doesn't exist), add the following, then restart LM Studio.

```json
{
  "mcpServers": {
    "rashinban-for-ai": {
      "url": "http://127.0.0.1:3766/mcp"
    }
  }
}
```

**Claude Code**: run this in a terminal.

```powershell
claude mcp add --transport http rashinban http://127.0.0.1:3766/mcp
```

**Claude Desktop**: go to "Settings → Connectors → Add custom connector" and register the URL above. If custom connectors aren't available in your environment, you can bridge via `mcp-remote` in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "rashinban-for-ai": {
      "command": "npx",
      "args": ["mcp-remote", "http://127.0.0.1:3766/mcp"]
    }
  }
}
```

> **Migrating from v0.4.x or earlier**: the old stdio-style config (`command: node`, `args` pointing to `server.js`) no longer works. Replace it with the URL-based config shown above in each client.

## Basic Usage

1. With the MCP server running, open the web page you want to interact with
2. Reload the page (tabs that were already open when you loaded the extension won't be affected until reloaded)
3. From your AI agent (e.g. LM Studio's Developer mode), call the `get_ui_elements` tool
4. The page's interactive elements (selectors, coordinates, states) are returned as JSON.

## Multi-Agent Use (v0.5.0+)

The server supports simultaneous connections from multiple AI agents.

- **Session isolation**: each agent gets its own MCP session; their states never mix
- **DOM change fan-out**: page changes are delivered to every connected agent. One agent calling `get_dom_changes` does not consume the others' queues
- **Per-agent michishirube**: pass `agent_id` to `save_michishirube` to keep each agent's waypoint separate. If omitted, a session-specific ID is used (it changes when the session ends, so **explicitly set `agent_id` for handoffs**)
- **Tab targeting**: pass `clientId` (from `list_connected_tabs`) to `get_ui_elements` so that agent A works on tab 1 while agent B works on tab 2 in parallel

## Available MCP Tools

| Tool | Description |
|---|---|
| `get_ui_elements` | Get interactive UI elements. Accepts `clientId` to target a specific tab (defaults to the most recently active tab) |
| `list_connected_tabs` | List the browser tabs currently connected (clientId, URL, title) |
| `get_dom_changes` | Get dynamic page changes (newly appeared elements). Only the caller's own session queue is consumed |
| `save_michishirube` | Save a waypoint (michishirube) per agent (`agent_id` defaults to a session-specific ID) |
| `get_latest_michishirube` | Get a saved waypoint. Pass `agent_id` to read another agent's waypoint |
| `list_michishirube` | List saved waypoints (agent_id, saved_at, goal, status) |
| `clear_michishirube` | Delete waypoints. `all: true` deletes every agent's waypoint |

## Verifying It's Working

### Check the console log

Open DevTools (F12) → Console on the target page. Extraction results appear there.

```
[Rashinban] 42個の操作可能要素を検出 - https://...
  enabled: 38 / disabled: 2 / hidden: 1 / その他: 1
```

### Check the popup's connection indicator

Click the extension icon to open the popup. A green indicator means connected.

> As of v0.5.0 the server is a persistent process, so the indicator should stay green whenever the server is running and the token is set correctly. If it stays "disconnected", check that the server is running and that the token matches.

### Check the server-side log

The terminal running the MCP server shows connection and DOM-change activity.

```
[Rashinban] 認証成功: <extension ID> → <URL>
[Rashinban] DOM変化を受信: 6個の新要素 (https://...)
```

## Language Selection

The extension's popup and options page support 5 languages: Japanese, English, Simplified Chinese, Spanish, and French. Select a language from the dropdown in either screen — the choice is shared between both.

## Sending Feedback

Both the popup and options page have a feedback button that opens a feedback form in a new tab. Bug reports, feature requests, and general feedback are all welcome there.

## Security

This tool is protected by three layers of security:

1. **Localhost-only binding** — rejects connections from outside the local machine
2. **Extension ID verification** — the first connection is auto-approved and recorded; later connections are checked against it
3. **Token authentication** — authenticates communication between the extension and the MCP server

These settings are stored in `mcp-server/config.json`. Since this file contains your auth token, **never share it with others or commit it to a repository** (it's excluded via `.gitignore`, but note that zip compression or folder copying does *not* respect `.gitignore`).

## When Something Goes Wrong

See [Troubleshooting](./Troubleshooting_EN.md).
