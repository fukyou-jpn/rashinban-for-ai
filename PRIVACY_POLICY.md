# Privacy Policy (Rashinban for AI)

Last updated: June 21, 2026

## About This Extension

Rashinban for AI ("this extension") is a Chrome extension that extracts actionable UI elements (CSS selectors, coordinates, and states) from web pages and passes them to a local MCP server running on your own computer. It's designed to help AI agents (e.g. LM Studio) interact with your browser.

## What We Collect and Why

Everything this extension handles stays **entirely on your own computer**. Nothing is sent to any third-party server.

| Data | Stored where | Purpose |
|---|---|---|
| Auth token | Browser extension storage (`chrome.storage.local`) and a local file (`config.json`) | Authenticates communication between the extension and the local MCP server |
| Extension ID | Local file (`config.json`) | Confirms the connection is coming from the legitimate extension |
| Current tab's URL and title | Transmitted only, never stored | Lets the local MCP server identify which tab an action applies to |
| Interactive page elements (CSS selectors, coordinates, element states) | Transmitted only, never stored | Provided to the AI agent via the local MCP server so it can interact with the page |

## Third-Party Sharing

This extension does not share any data with advertising, analytics, or any other third-party services. It does not communicate with any external API.

## Scope of Communication

All communication between this extension and the MCP server is limited to `ws://127.0.0.1:3765` (localhost). No data travels over the internet.

## Deleting Your Data

- The auth token can be changed or removed anytime from the extension's options page.
- The locally stored `config.json` can be reset by deleting the file directly.

## Permissions Used

| Permission | Purpose |
|---|---|
| `scripting` | Runs a content script to extract interactive elements from the page |
| `tabs` | Reads information about the currently active tab |
| `clipboardWrite` | Powers the "copy as JSON" feature for extracted elements |
| `storage` | Stores the auth token |
| `host_permissions` (`ws://127.0.0.1/*`) | WebSocket communication with the local MCP server |

## Contact

Questions about this policy can be sent to rashinbanfeedback@gmail.com.

## Changes to This Policy

This policy may be updated as features are added. Significant changes will be noted here and in release notes.
