<p align="center">
  <img src="images/logo.jpg" width="180">
</p>

# Rashinban for AI 🧭

**Rashinban for AI** is an ultra-lightweight Chrome extension that acts as a precision bridge between web UIs and AI agents — cutting the tokens your AI spends on every page, and making its browser actions faster.

Instead of forcing AI to parse bloated DOM trees or process heavy screenshots, it extracts only the essential actionable data — CSS selectors, coordinates, and element states — and streams it over WebSocket. It turns human-centric, copy-paste-heavy web UIs into a fast, cost-efficient interface built for machines.

## ✨ Key Features

* **Massive Token & Time Savings** — No screenshots, no full DOM parsing. By passing only the "where-to-interact" skeleton, it minimizes LLM context window usage and cuts AI response times significantly.
* **Hybrid Monitoring** — The extension watches the page continuously via MutationObserver, but only pushes updates to the MCP server on request (debounced). You get real-time accuracy without flooding the connection.
* **Resilient WebSocket Connection** — Automatic reconnection with exponential backoff (1s → 2s → 4s...) keeps the bridge alive through sleep, server restarts, or network drops — silently, with no missed notifications.
* **Defense in Depth** — Three independent security layers: localhost-only binding, Chrome extension ID verification, and token authentication.

## 🛠️ Tech Stack & Architecture
* **Frontend:** Chrome Extension (Manifest V3, JavaScript / MutationObserver)
* **Backend:** Local MCP Server (WebSocket)
* **Supported AI Environments:** Works smoothly with local LLM setups (e.g., LM Studio) and custom MCP-compatible AI agents.

## ⚠️ Current Limitations
* Does not support file upload fields due to browser security restrictions.
* Cannot bypass reCAPTCHA/anti-bot systems.

---

## 📦 Repository

[https://github.com/fukyou-jpn/rashinban-for-ai](https://github.com/fukyou-jpn/rashinban-for-ai)

## 📄 License

This project is licensed under the **GNU General Public License v3.0 (GPL-3.0)**.
See the [LICENSE](LICENSE) file for details.

## Related Projects

- Rashinban MCP Server
