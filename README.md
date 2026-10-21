<p align="center">
  <a href="https://agents.bipa.app">
    <img src="assets/logo.png" alt="Bipa CLI" width="500" />
  </a>
</p>

<p align="center">
  <strong>Payment rails for AI agents — Pix, BRL, and beyond.</strong>
</p>

<p align="center">
  <a href="https://github.com/bipa-app/bipa-cli/releases"><img src="https://img.shields.io/github/v/release/bipa-app/bipa-cli?label=release" alt="Latest release" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-yellow.svg" alt="License: MIT" /></a>
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/MCP-server-blue" alt="MCP server" /></a>
</p>

<p align="center">
  <a href="https://agents.bipa.app">Website</a> ·
  <a href="https://agents.bipa.app/docs">Docs</a> ·
  <a href="https://github.com/bipa-app/bipa-cli/releases">Releases</a> ·
  <a href="https://agents.bipa.app/docs/seguranca">Security model</a>
</p>

<p align="center">
  <a href="https://apps.apple.com/br/app/bipa/id1516842324"><img src="https://img.shields.io/badge/Download-App%20Store-black?logo=apple&logoColor=white" alt="Download on the App Store" /></a>
  <a href="https://play.google.com/store/apps/details?id=bipa.app.bipa"><img src="https://img.shields.io/badge/Get%20it%20on-Google%20Play-414141?logo=googleplay&logoColor=white" alt="Get it on Google Play" /></a>
</p>

---

## What is Bipa CLI?

Bipa CLI gives AI agents a programmable payment account. Agents can check balances, read transaction history, and send Pix transfers — all through a secure MCP (Model Context Protocol) server that plugs into Claude, ChatGPT, Cursor, Windsurf, VS Code, and any MCP-compatible host.

### Key capabilities

| Capability | Description |
|---|---|
| **Pix transfers** | Send and receive instant BRL payments via Pix keys |
| **Balance & history** | Real-time account balance and transaction feed |
| **BR Code decode** | Parse Pix QR codes and Copia & Cola payloads |
| **MCP server** | First-class integration with AI agent hosts |
| **OAuth login** | Secure browser-based authentication flow |
| **Keyring storage** | Credentials stored in OS-native keychain |

## What can your agent do?

A few real-world workflows people build with Bipa CLI:

- **Split bills with friends and family.** *"Send R$ 35 to Maria for yesterday's lunch"* — your agent drafts the Pix, you confirm with Face ID.
- **Pay your monthly bills.** Rent, utilities, streaming subscriptions — ask any MCP-aware agent and approve in the Bipa app.
- **Build your own personal finance dashboard.** Use Replit Agent or Lovable to ship a UI with real Pix balances, transactions, and spending categories.
- **Audit and analyze your spending.** *"How much did I spend on rideshare last month?"* / *"Show all my Pix to my landlord this year."*

Every payment still requires biometric approval in the Bipa mobile app — agents request, you authorize.

## Supported AI agents

Jump straight to the setup guide for your stack:

| Agent | Transport | Setup guide |
|---|---|---|
| Claude (Desktop & Claude.ai) | Local + Remote | [docs/claude](https://agents.bipa.app/docs/claude) |
| Claude Code | Local + Remote | [docs/claude-code](https://agents.bipa.app/docs/claude-code) |
| ChatGPT | Remote | [docs/chatgpt](https://agents.bipa.app/docs/chatgpt) |
| Cursor | Local + Remote | [docs/cursor](https://agents.bipa.app/docs/cursor) |
| Gemini CLI | Local | [docs/gemini](https://agents.bipa.app/docs/gemini) |
| Codex | Local | [docs/codex](https://agents.bipa.app/docs/codex) |
| OpenClaw | Local | [docs/openclaw](https://agents.bipa.app/docs/openclaw) |
| n8n | Remote | [docs/n8n](https://agents.bipa.app/docs/n8n) |
| Antigravity | Local + Remote | [docs/antigravity](https://agents.bipa.app/docs/antigravity) |
| Grok | Remote | [docs/grok](https://agents.bipa.app/docs/grok) |
| Replit | Remote | [docs/replit](https://agents.bipa.app/docs/replit) |
| Lovable | Remote | [docs/lovable](https://agents.bipa.app/docs/lovable) |

Don't see your host? Any MCP-compatible client works with the remote server at `https://mcp.bipa.app/mcp`.

## Agent Skill

Install the Bipa CLI skill to give your AI agent full knowledge of how to use Bipa CLI — including account setup, Pix payments, and all available tools:

```bash
npx skills add bipa-app/bipa-cli
```

The skill teaches your agent how to:
- Help users open a Bipa account and complete onboarding
- Install and authenticate the Bipa CLI
- Make Pix payments, check balances, decode QR codes
- Analyze transaction history and detect patterns

You can also view the skill reference directly from the CLI:

```bash
bipa skill
```

## Before you start

You need a **Bipa account** and the **Bipa mobile app** to use the CLI — the app is where you approve every Pix payment with biometrics.

1. **Create a free Bipa account** at [bipa.app](https://bipa.app) (under 2 minutes, no fees).
2. **Install the mobile app:**
   - [App Store (iOS)](https://apps.apple.com/br/app/bipa/id1516842324)
   - [Google Play (Android)](https://play.google.com/store/apps/details?id=bipa.app.bipa)
3. Complete onboarding in the app — you'll need this to authorize agent requests.

> Bipa is currently available for residents of Brazil 🇧🇷.

## Install

```bash
curl -fsSL https://agents.bipa.app/install.sh | sh
```

The managed installer places a launcher in `~/.local/bin/bipa` and keeps versioned binaries under `~/.bipa/`.

Optional flags:

```bash
# Custom binary directory
curl -fsSL https://agents.bipa.app/install.sh | sh -s -- --bin-dir /usr/local/bin

# Pin a specific version
curl -fsSL https://agents.bipa.app/install.sh | sh -s -- --version v0.1.2
```

## Quick start

```bash
# Authenticate (prints URL for agents, --open to launch browser)
bipa login --web

# Install the MCP server for Claude Desktop
bipa mcp install --client claude

# Check your balance
bipa pix balance

# Send a Pix payment
bipa pix pay --key alice@example.com --amount 25,00 --note "coffee" --agent-message "Paying Alice for coffee"

# View recent transactions
bipa pix history --limit 5
```

## MCP tools

Once installed, Claude Desktop (or any MCP host) can use these tools:

| Tool | Description |
|---|---|
| `bipa_whoami` | Session status |
| `bipa_account` | Account profile and metadata |
| `bipa_balance` | Available balance in cents |
| `bipa_history` | Transaction history (list or detail) |
| `bipa_deposit` | Pix keys for receiving deposits |
| `bipa_pix_keys` | Configured Pix keys |
| `bipa_limits` | Transfer risk limits |
| `bipa_pix_brcode_decode` | Decode Pix BR Code payloads |
| `bipa_pix_pay_key` | Create a Pix transfer |

A remote MCP server is also available at `https://mcp.bipa.app/mcp` with automatic OAuth 2.1 authentication — no local install required.

## Cursor & Grok Bot plugin

This repo is also a plugin for the Cursor and Grok Bot marketplaces. The manifest lives at [`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json) and bundles:

- the remote MCP server (`https://mcp.bipa.app/mcp`, OAuth 2.1), declared in [`mcp.json`](mcp.json)
- the [Bipa skill](skills/bipa/SKILL.md)
- the square logo at [`assets/icon.png`](assets/icon.png)

Once installed, the host opens the OAuth flow on first use and you authorize access in the Bipa app. No token or secret is needed.

## Security & approvals

Bipa CLI is designed so that **agents request, humans authorize**:

- Every Pix payment requires biometric approval in the Bipa mobile app — the CLI never moves money on its own.
- Spending limits are configurable per day, per transaction, and per recipient.
- Credentials are stored in your OS-native keychain (Keychain on macOS, Secret Service on Linux, Credential Manager on Windows).
- The remote MCP server uses OAuth 2.1 with PKCE — short-lived tokens, no shared secrets.

Full architecture: [agents.bipa.app/docs/seguranca](https://agents.bipa.app/docs/seguranca).

## Releases

This repository hosts pre-built binaries for every release. The managed installer downloads from the [Releases](https://github.com/bipa-app/bipa-cli/releases) page.

| Platform | Architecture | Asset |
|---|---|---|
| macOS | Apple Silicon | `bipa-vX.Y.Z-darwin-arm64.tar.gz` |
| macOS | Intel | `bipa-vX.Y.Z-darwin-x64.tar.gz` |
| Linux | x86_64 | `bipa-vX.Y.Z-linux-x64.tar.gz` |
| Linux | ARM64 | `bipa-vX.Y.Z-linux-arm64.tar.gz` |

## Links

- **Website**: [agents.bipa.app](https://agents.bipa.app)
- **Docs**: [agents.bipa.app/docs](https://agents.bipa.app/docs)
- **Security model**: [agents.bipa.app/docs/seguranca](https://agents.bipa.app/docs/seguranca)
- **Open an account**: [bipa.app](https://bipa.app)
- **Mobile app — iOS**: [App Store](https://apps.apple.com/br/app/bipa/id1516842324)
- **Mobile app — Android**: [Google Play](https://play.google.com/store/apps/details?id=bipa.app.bipa)
- **Releases**: [github.com/bipa-app/bipa-cli/releases](https://github.com/bipa-app/bipa-cli/releases)
- **Bug reports & feedback**: [GitHub Issues](https://github.com/bipa-app/bipa-cli/issues)

## License

MIT — see [LICENSE](LICENSE) for details.
