# Troubleshooting

## Issue: Not sure if the MCP server is running

**Check**

```powershell
Get-Process node -ErrorAction SilentlyContinue
```
→ No output means the server isn't running.

```powershell
Get-NetTCPConnection -LocalPort 3765, 3766 -ErrorAction SilentlyContinue
```
→ No output means the server isn't running (3765 = extension side, 3766 = AI agent side).

**Fix**

```powershell
cd "<path to project>\mcp-server"
node server.js
```
You should see both of these lines once it's up:

```
[Rashinban] WebSocketサーバー起動: ws://127.0.0.1:3765
[Rashinban] MCPサーバー起動: http://127.0.0.1:3766/mcp
```

As of v0.5.0 the server is a persistent process — keep it running while your AI agents use the tool.

---

## Issue: The server exits saying port 3765 (or 3766) is already in use

**Cause**
Another server process is already holding the port. The two common cases:

1. A previous instance you forgot to close.
2. **A leftover stdio-style config (v0.4.x or earlier) in one of your AI agents** — a client (LM Studio, Claude Desktop, etc.) still configured with the old `command: node` setup silently spawns `server.js` in the background and grabs the port. In this case, stopping the process isn't enough: also switch that client's MCP config to the HTTP form (see the user guide, step 5), or the conflict will come back every time the client restarts.

**Fix**

1. Find the process ID using the port
   ```powershell
   Get-NetTCPConnection -LocalPort 3765, 3766 | Select-Object LocalPort, OwningProcess
   ```
2. Stop that process
   ```powershell
   Stop-Process -Id <process ID> -Force
   ```
3. Confirm it's gone, then restart
   ```powershell
   Get-Process node -ErrorAction SilentlyContinue
   node server.js
   ```

---

## Issue: Your auth token was accidentally shown on screen or may have been exposed

**Fix (rotate the token)**

1. Stop the server (`Ctrl + C` in the terminal where it's running)
2. Delete `config.json`
   ```powershell
   Remove-Item "<path to project>\mcp-server\config.json" -ErrorAction SilentlyContinue
   ```
3. Restart the server (a new token is generated automatically)
   ```powershell
   node server.js
   ```
4. Set the new token in the extension's options page
   - `chrome://extensions` → extension's "Details" → "Extension options"
   - Paste the token and save

Note: `config.json` also stores the approved extension ID. Deleting it resets that approval too, so the extension will need to re-register on its next connection (see below).

---

## Issue: `Uncaught Error: Extension context invalidated.` (reloading the tab doesn't fix it)

**Cause**
When you reload the extension at `chrome://extensions` (the ↺ button), content scripts in tabs that were already open become orphaned. In this state, reloading the tab with F5 alone won't recover them. If `Extension context invalidated` keeps appearing repeatedly in the console, this is what's happening.

**Fix**

If F5-reloading the tab doesn't help, try disabling and re-enabling the extension itself:

1. Open `chrome://extensions`
2. Toggle Rashinban for AI **OFF**
3. Wait a few seconds, then toggle it back **ON**
4. F5-reload every affected tab

If that still doesn't work, remove the extension entirely from `chrome://extensions` and reload it via "Load unpacked." Note that your saved token is stored in `chrome.storage.local` — removing the extension deletes it, so you'll need to re-enter it in the options page afterward.

**Confirming it's resolved**
You should see lines like the following in the MCP server terminal for each recovered tab:
```
[Rashinban] 認証成功: <extension ID> → <URL>
```

---

## Issue: `Token mismatch, connection rejected: extensionId=undefined`

**Cause**
Usually one of two things:

1. **Right after deleting and regenerating `config.json`** — the extension is still trying to reconnect with stale token/connection info.
2. **Multiple tabs were open when the extension was updated** — tabs that were open before the update keep retrying with stale connection info because they were never reloaded.

**Fix**

1. Reload the extension itself at `chrome://extensions` (the ↺ button)
2. Reload **every tab** where the extension is active (F5) — even one missed tab will keep producing errors
3. Check the options page to confirm the new token was saved correctly

**Confirming it's resolved**
You should start seeing lines like:
```
[Rashinban] 認証成功: <extension ID> → <URL>
```
If a stale tab is still out there, you'll see a few rounds of `token mismatch` followed by `disconnected` before it settles. If it doesn't stop, check for a tab you missed.

---

## Issue: The AI says it "can't use tools" or "doesn't have that capability"

**Cause**
This happens often, and it's not a configuration problem — the extension is already connected and the tools are already available. Usually it's because the chat predates the tools being registered, so the model is still working from a state where it remembers having no tools.

**Fix**

Start a new chat and try again.

---

## Issue: The tool call succeeds, but the AI's answer is oddly vague (no specific element names or coordinates)

**Cause**
This one doesn't throw an error. If the MCP server's terminal log correctly shows the number of elements found (e.g. "Found 38 interactive elements") but the AI's answer is generic — something like "it includes buttons and links" without naming any of them — this is what's happening.

The local LLM's (e.g. Ollama) context window (`num_ctx`) is too small, so the returned JSON (which grows with the number of elements) gets truncated before the model ever sees it. Since nothing errors out, this is easy to miss — and the model itself usually won't flag that its data was cut off; it just answers in vague generalities instead.

**Fix**

1. In your client's model settings (e.g. AnythingLLM), check the Context Window (Max Tokens) value
2. If it's set to auto, or to a small value like 2048–4096, increase it to **8192 or higher (recommended)** and save
3. Call the tool again and check whether the AI's answer includes specific element names and coordinates

**Confirming it's resolved**
Ask something like "name 3 of the elements you found, specifically" — if it returns real labels and types (link, button, etc.), it's working correctly.

---

## Issue: Can't connect to the MCP server (localhost) when using Claude Code

**Cause**
Claude Code has an optional sandboxing feature (`/sandbox`). When it's enabled, there's a known issue where outbound connections to localhost (127.0.0.1 / ::1) get blocked — even if you add localhost/127.0.0.1 to `sandbox.network.allowedDomains`, the Seatbelt-level block can still take effect first (see GitHub `anthropics/claude-code` Issue #28018, unresolved as of June 2026).

**Fix**

1. Don't enable `/sandbox` when using this tool — by default, Claude Code runs on your own machine's network, so this restriction doesn't apply.
2. If you do want to use `/sandbox`, check the status of the issue above and verify separately that localhost connections actually go through. There's currently no reliable workaround on this tool's side.

**Confirming it's resolved**
If the MCP server terminal shows `[Rashinban] 認証成功: <extension ID> → <URL>`, the connection is working fine.

Note: this is a known Claude Code issue, not a bug in this tool.

---

## Issue: Can't even test in a browser from Cowork (Claude desktop's Cowork mode)

**Cause**
Cowork mode runs Claude in a cloud sandbox whose network access is locked to a host-side allowlist set by an admin. This blocks not just this tool's MCP server (localhost), but also ordinary, non-allowlisted sites (observed in practice with peraichi.com and ja.wikipedia.org) — both get rejected as "blocked by your organization's policy."

This is unrelated to the Claude Code `/sandbox` issue above — it's a Cowork-specific host restriction. Even using Cowork's built-in Claude in Chrome browser tool doesn't help, since the underlying requests still hit the same allowlist; page content and element detection both failed.

**Fix**

1. Test this tool from Claude Code (default settings, no `/sandbox`) or a local client like LM Studio instead.
2. If you specifically need to test from Cowork, have an admin check the network access settings under Admin settings → Capabilities — personal/individual Cowork accounts may not have this configurable at all.

**Confirming it's resolved**
This restriction is environment-specific to Cowork and can't be fixed from this tool's side. Switch to a different environment (e.g. Claude Code) and use the verification steps above instead.

---

## Notes

- This fits into the same path as an existing LM Studio + custom MCP server setup — no extra cost involved.
- On SPAs (single-page apps), the URL may not change when the screen changes. If elements can't be retrieved after a page transition, check for this.
- Cannot bypass reCAPTCHA or other anti-bot systems (out of scope by design).

---

## Note: Does the token change every time the server starts?

No. As long as `config.json` exists, the same token is reused across restarts. A new token is only generated if `config.json` is missing or deleted — which is exactly the case if you've rotated it after an accidental exposure.

---

## Issue: The AI agent can't connect to the MCP server (tools don't show up)

**Causes and fixes (check in order)**

1. **The server isn't running** — as of v0.5.0 it's a persistent process. AI agents no longer start it for you; start `node server.js` yourself (see the first item on this page).
2. **The client config is still the old stdio form** — the `command: node` + `server.js` path style no longer works. Switch to the HTTP form using the URL `http://127.0.0.1:3766/mcp` (see the user guide, step 5).
3. **Wrong URL** — the path must include `/mcp` (`http://127.0.0.1:3766` alone won't connect).

**Confirming it's resolved**
When the server terminal shows `[Rashinban] MCPセッション開始: <session ID>`, the agent-side connection is established.

---

## Note: Can multiple AI agents use it at the same time?

Yes (v0.5.0+). Each agent just connects to the same URL (`http://127.0.0.1:3766/mcp`); sessions are isolated automatically. DOM change notifications are delivered to every connected agent, and michishirube entries can be kept separate per `agent_id`. See "Multi-Agent Use" in the user guide.
