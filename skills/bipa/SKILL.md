---
name: bipa
description: Use Bipa CLI to make Pix payments, check balances, analyze transaction history, and decode BR codes for AI agents. MUST use this skill whenever the user mentions paying someone via Pix, checking their balance, reviewing transactions, scanning a Pix QR code, setting up Bipa CLI, or asks the agent to handle any Brazilian instant payment. Also use when the user wants to automate recurring payments, detect duplicate transactions, or compose multi-step financial workflows. Use this skill even when the user just mentions "Pix", "Bipa", "BRL transfer", "pagamento", "saldo", or wants to send money to a CPF, email, phone, or CNPJ in Brazil.
---

# Bipa CLI

Bipa CLI gives AI agents the ability to make real Pix payments in Brazil. It works as a CLI tool and as an MCP server that exposes financial tools to any MCP-compatible agent.

Pix is Brazil's instant payment system — transfers settle in seconds, 24/7, free for individuals. Bipa CLI is built on Bipa's BCB-authorized financial infrastructure.

## First: Does the User Have a Bipa Account?

Before anything else, find out whether the user already has a Bipa account. Bipa CLI requires one. Ask directly:

> "Do you have a Bipa account? Bipa CLI uses Bipa as the underlying bank, so you'll need one to send/receive Pix."

### If they DON'T have an account → help them open one

Opening a Bipa account is free, takes ~5 minutes, and gives them access to:
- **Free Pix account** — send/receive instant BRL transfers 24/7
- **Buy & sell Bitcoin** — directly from the app
- **BTC/USDT/BRL collateralized credit card** — works with Apple Pay / Google Pay
- **Crypto rails** — Polygon, Ethereum, Bitcoin, Lightning Network

There are two paths to create a Bipa account:

#### Step 1: Register via web (agent can guide this)

Direct the user to **https://bipa.app/cadastro** — Bipa's web KYC flow. Walk them through it:

1. **CPF** — Enter their CPF (Brazilian tax ID, 11 digits). This starts the registration.
2. **Terms** — Accept Bipa's terms of service and privacy policy.
3. **Email** — Enter email, then verify with a 6-digit PIN sent to that email.
4. **Phone** — Enter phone number (Brazilian +55), then verify with a 6-digit PIN via SMS.
5. **Occupation** — Select occupation from a list.
6. **Income** — Declare monthly income range.
7. **PEP** — Declare if they are a politically exposed person (most people: "No").
8. **Address** — Enter CEP (postal code), the system auto-fills street/district. Add number + complement.
9. **Document verification** — Scan a QR code on their phone to take a selfie + photo of their ID document (RG or CNH). Bipa uses automated KYC verification.
10. **Processing** — Account is reviewed (usually approved within minutes).

**Requirements:**
- Valid Brazilian CPF
- Brazilian phone number (+55)
- Valid email address
- Brazilian residential address
- ID document (RG or CNH)

#### Step 2: Wait for approval

The user receives an email when their account is approved. They can also check by revisiting https://bipa.app/cadastro — the page reflects the current status.

#### Step 3: Activate the account in the Bipa app

This step requires the mobile app — it cannot be done on the web. The user must:

1. **Download the Bipa app**
   - iOS: https://apps.apple.com/br/app/bipa/id1560985113
   - Android: https://play.google.com/store/apps/details?id=app.bipa
2. **Log in** using the same phone number or email they registered with. Bipa sends a PIN to verify.
3. **Verify their identity** via liveness check (face authentication) to activate the new device.
4. **Complete in-app onboarding:**
   - **Create a Bipa Tag** — a unique username (like `@joao`) used as their identity on Bipa and for Lightning payments.
   - **Set a security PIN** — a numeric PIN for additional transaction security.
   - **Create their first Pix key** — this registers a Pix key (CPF, email, or phone) with the Central Bank so they can receive Pix transfers.

Once onboarding is complete, the account is fully active. The user can now receive Pix, buy/sell Bitcoin, and use all Bipa features.

#### Step 4: Connect to Bipa CLI

Now the user can log in to Bipa CLI and connect their agent. After this, they only need to open the Bipa app to approve or reject payments requested by their agent — everything else is automated.

### If they DO have an account → proceed to install & login

## Install Bipa CLI

```bash
# Managed installer (recommended for users and agents)
curl -fsSL https://agents.bipa.app/install.sh | sh
```

## Authenticate

Login requires an explicit method flag and an agent identity. `--agent-name` is **required** (ask the user for your name if you don't know it); `--agent-kind` defaults to `other`.

```bash
# OAuth login — prints URL for agent-friendly workflows
bipa login --web --agent-name <NAME> --agent-kind <KIND>

# OAuth login — opens browser automatically (human-friendly)
bipa login --web --open --agent-name <NAME> --agent-kind <KIND>

# PIN-based login via email (two-step)
bipa login --pin --email user@example.com --agent-name <NAME> --agent-kind <KIND>
bipa verify <PIN>

# PIN-based login via phone
bipa login --pin --phone "+55 11 99999 0000" --agent-name <NAME> --agent-kind <KIND>
bipa verify <PIN>
```

`--agent-kind` is the AI platform: `openclaw`, `claude`, `claude_code`, `chatgpt`, `codex`, `cursor`, `antigravity`, `grok`, `gemini`, `bipa`, or `other`.

Running `bipa login` without flags prints a usage summary.

Choosing a method:

- **Headless / chat agents (e.g. OpenClaw):** use PIN — `bipa login --pin --email …` or `--phone …`, then relay the PIN via `bipa verify <PIN>`. You can't complete a browser flow, but you can relay a code the user reads back to you.
- **Agents that can surface a URL:** `--web` prints the auth URL to stdout for you to present to the user.
- **Humans at the keyboard:** `--web --open` opens the browser automatically.

### Check session status

```bash
bipa whoami          # human-readable
bipa whoami -f json  # session_status, reauth_required, recommended_command, auth_method,
                     # last_login_channel, last_login_hint, expires_at, expires_in_seconds
```

Prefer `session_status` (`active` / `expired` / `none`) and `reauth_required` over reasoning about
`expires_in_seconds` yourself. When `reauth_required` is true, `recommended_command` is the exact
command to run.

### Auth boundaries — CLI session vs. app connector

There are **two independent sessions**, and reauthenticating one does **not** refresh the other:

- **CLI session** — used by `bipa …` commands and the local `bipa mcp` server (they share the same
  stored credentials). Recover it with `bipa login` / `bipa verify` as below.
- **Hosted app connector** — a remote MCP connection managed by the client app (e.g. Claude/OpenClaw
  connectors). When it errors with something like *"This app connection requires reauthentication"*,
  that is the connector's own session. `bipa login`/`bipa verify` will **not** fix it — the user
  must reauthenticate the connector inside the app that owns it.

If you're unsure which one failed: `no active Bipa CLI session` / `Bipa CLI session expired` are the
CLI session. Anything phrased as "app connection" / "connector" is the hosted connector.

If `whoami` reports `reauth_required: true` with **no** `recommended_command`, the session came from
the `BIPA_JWT` environment (not `bipa login`) and can't be recovered with a CLI command — it must be
refreshed wherever that env var is set (the deployment/host), then the MCP server restarted.

### Reauthenticate mid-session

A tool call can fail with `no active Bipa CLI session` or `Bipa CLI session expired` when the CLI
session lapses between requests. This is recoverable — don't abandon the task; reauthenticate and
retry the original call. Don't guess the flow — `bipa whoami -f json` tells you exactly what to do:

```json
{ "session_status": "expired", "reauth_required": true,
  "auth_method": "pin", "last_login_channel": "phone", "last_login_hint": "+••••0000",
  "recommended_command": "bipa login --pin --phone +••••0000 --agent-name Amy --agent-kind openclaw && bipa verify <PIN>" }
```

1. Read `session_status` / `reauth_required`. If `reauth_required` is true, run the
   `recommended_command` — it already encodes the right method (`--web` vs `--pin`) and, for PIN, the
   **same channel the user last used** (`last_login_channel`), so you never guess email vs. phone.
2. **OAuth** (`auth_method: oauth`): the command is `bipa login --web …` — present the printed URL to
   the user and wait for them to finish in the browser.
3. **PIN** (`auth_method: pin`): the PIN is delivered out-of-band to the user's `last_login_channel`.
   The masked `last_login_hint` is a placeholder — confirm the full email/phone with the user if
   needed. **Ask the user to read the PIN back to you**, then run `bipa verify <PIN>` within ~60s.
4. Re-run the tool that originally failed.

The PIN always goes to the human, never to you; your job is to trigger it and relay it via `bipa
verify`.

## Set Up MCP for Claude Desktop

```bash
bipa mcp install --client claude
# Restart Claude Desktop to pick up the change
```

The MCP server reuses saved credentials. No tokens in config files.

Remote MCP is also available at `https://mcp.bipa.app/mcp` with automatic OAuth 2.1 (for Claude.ai, ChatGPT, Cursor, VS Code, and other remote-capable clients).

## MCP Tools

### Pix payments

| Tool | Description |
|---|---|
| `bipa_pay` | Create a Pix transfer by `key` (+ `amount_cents`, `agent_message`). Optional `schedule` object (`date`, `frequency` = once/daily/weekly/monthly, `count`) for future or recurring payments. `bipa_pix_send` is an alias. For other destinations use `bipa_pix_pay_recipient` / `_tag` / `_trusted_contact` / `_brcode`. |
| `bipa_pix_pay_key` | Send a Pix payment by key (lookup + transfer in one call). Use as fallback when recipient is not in saved list. |
| `bipa_pix_recipient_suggestions` | List top 10 saved recipients. **Call this first** when the user wants to pay someone by name. |
| `bipa_pix_pay_recipient` | Pay a saved recipient by `pix_payment_recipient_id` (from `bipa_pix_recipient_suggestions`). Skips Pix key lookup. |
| `bipa_pix_trusted_contacts` | List pre-approved trusted contacts (payments skip biometric approval) |
| `bipa_pix_pay_trusted_contact` | Pay a trusted contact by `pix_trusted_contact_id` (from `bipa_pix_trusted_contacts`). Instant settlement. |
| `bipa_pix_tag_preview` | Look up a BIPA user by tag (e.g. `$bipatag`) — confirm before paying |
| `bipa_pix_pay_tag` | Pay a BIPA user by tag |
| `bipa_pix_brcode_decode` | Decode a Pix QR / Copia e Cola string (local, no auth) |
| `bipa_pix_brcode_preview` | Preview a BR Code via server: returns recipient name, document, amount, and payment id |
| `bipa_pix_pay_brcode` | Pay a BR Code (preview + transfer in one call). For `fixed` codes omit `amount_cents`; for `custom` codes pass it. |
| `bipa_pix_brcode_encode` | Generate a static Pix Copia e Cola payload from a registered key (+ optional fixed amount). Server-issued, reconciles incoming payments back to the agent. Each call mints a fresh idempotency key — calling twice with the same args creates two distinct BR Codes; cache the response instead of retrying. |
| `bipa_pix_brcode_encode_widget` | Render the QR widget for a BR Code that was just generated. Pass the full output of `bipa_pix_brcode_encode`. |

### Bank slips & bills (boletos)

| Tool | Description |
|---|---|
| `bipa_bank_slip_preview` | Preview a bank slip (boleto) from its digitable line or barcode: recipient, amount, kind (static/dynamic), due date. Dynamic slips return `min_amount`/`max_amount`. Read-only; does NOT pay. |
| `bipa_pay_bank_slip` | Submit a bank slip payment **request**. Does NOT pay directly — creates a pending approval (`status: awaiting_user_approval`, `approval_id`) the user must authorize in the Bipa app. Pass the line/barcode in `input`; for `custom` slips pass `amount_cents` within min/max, for `fixed` slips omit it. |
| `bipa_dda_list` | List the DDA (Débito Direto Autorizado) bank slips registered to the user — boletos delivered into Bipa so they can be reviewed without typing the barcode. Each item has `id`, `status` (overdue/due/paid/canceled), `recipient`, `amount`, `due_date`. Read-only; empty when the user isn't subscribed to DDA. |

### Account, balance & history

| Tool | Description |
|---|---|
| `bipa_balance` | Current available balance and savings ("cofrinho"/"caixinha") balance (formatted BRL strings) |
| `bipa_limits` | Transfer risk limits (daily and nightly) |
| `bipa_deposit` | List deposit Pix keys |
| `bipa_pix_keys` | List configured Pix keys |
| `bipa_account` | Account profile and metadata |
| `bipa_whoami` | Session status (includes auth method and expiry) |
| `bipa_history` | Pix transaction list (`limit?`, default 20, max 50) or detail by `id` |
| `bipa_transactions` | Multi-asset compact transaction list (BRL, BTC, USDT). Most recent up to `limit` (default 20, max 50). |
| `bipa_transaction_detail` | Detailed info for a specific multi-asset transaction by `id` |
| `bipa_timeline` | Unified activity timeline across all assets and event layers. Supports search, layer filtering, cursor pagination. |

### Prices & portfolio

| Tool | Description |
|---|---|
| `bipa_tickers` | Current BTC/BRL, USDT/BRL, and BTC/USDT bid/ask prices |
| `bipa_btc_prices` | BTC/BRL historical price series |
| `bipa_usdt_prices` | USDT/BRL historical price series |
| `bipa_portfolio` | Portfolio summary (P&L, trade stats, balance history) for a given asset and period |

> Most data tools above have a `_widget` counterpart (e.g. `bipa_balance_widget`). Widget tools are hidden from the model and exist only for app-side rendering — call the plain data tool; the host app calls the widget with that tool's output when it needs to render UI.

## CLI Commands

```
# Pix payments (one destination flag: --key | --brcode | --tag | --trusted-contact | --recipient)
bipa pix pay --key <PIX_KEY> --amount <BRL> --agent-message "reason" [--note "memo"]
bipa pix pay --key <PIX_KEY> --amount-cents <CENTS> --agent-message "reason"
bipa pix pay --brcode <COPIA_E_COLA> [--amount <BRL>] --agent-message "reason"
bipa pix pay --tag <BIPA_TAG> --amount <BRL> --agent-message "reason"
bipa pix pay --trusted-contact <ID> --amount <BRL> --agent-message "reason"
bipa pix pay --recipient <ID> --amount <BRL> --agent-message "reason"
# Scheduled / recurring (add to any pix pay)
bipa pix pay --key <PIX_KEY> --amount <BRL> --agent-message "reason" \
  --schedule-date 2026-07-01 --schedule-frequency monthly --schedule-count 12

# Read state
bipa pix balance
bipa pix history [--limit N] [TRANSACTION_ID]
bipa pix keys
bipa pix deposit
bipa pix limits
bipa pix recipient-suggestions
bipa pix trusted-contacts
bipa pix account
bipa pix brcode decode <BRCODE_STRING>
bipa pix brcode encode --key <PIX_KEY> [--amount <BRL>]

# Bank slips / boletos (alias: bipa boleto ...)
bipa bank-slip preview <DIGITABLE_LINE_OR_BARCODE>
bipa bank-slip pay <DIGITABLE_LINE_OR_BARCODE> [--amount-cents <CENTS>]

# DDA (registered boletos)
bipa dda list

# Prices, portfolio & multi-asset views
bipa prices tickers          # alias: bipa cotacao tickers
bipa prices btc
bipa prices usdt
bipa portfolio               # alias: bipa carteira
bipa timeline                # alias: bipa extrato-geral
bipa transactions            # alias: bipa transacoes
bipa transaction <ID>        # alias: bipa transacao

# Session & misc
bipa whoami
bipa login --web [--open] --agent-name <NAME> [--agent-kind <KIND>]
bipa logout
bipa skill
```

## Pix Key Types

Recipients are identified by Pix keys. The tool normalizes these automatically:

- **CPF**: `123.456.789-00` → `12345678900`
- **CNPJ**: `12.345.678/0001-99` → digits only
- **Phone**: `+55 (11) 99999-9999` → `+5511999999999`
- **Email**: `alice@example.com` → as-is
- **EVP (random key)**: UUID like `550e8400-e29b-41d4-a716-446655440000`

Pass the key in any common format — normalization is handled internally.

## Critical Rules

1. **Check saved recipients first.** When the user asks to pay someone by name, always call `bipa_pix_recipient_suggestions` first. If the person is in the list, use `bipa_pix_pay_recipient` — it's faster and avoids Pix key lookup. Only fall back to `bipa_pix_pay_key` (with a Pix key) when no saved recipient matches.

2. **`bipa_pix_pay_key` includes recipient lookup.** `bipa_pix_pay_key` resolves the recipient automatically. Never try to look up a Pix key separately before paying.

3. **`agent_message` is required for every payment.** This text is shown to the human in the Bipa app during the approval step. Write a clear, honest explanation of why the agent is making this payment (e.g., "User asked me to pay João R$20 for lunch"). Payments without `agent_message` are rejected.

4. **Payments require human approval.** After calling `bipa_pix_pay_key` or `bipa_pix_pay_recipient`, the payment enters `awaiting_approval` status. The user must approve it in the Bipa mobile app via biometric (Face ID / Touch ID). Exception: trusted contact payments (`bipa_pix_pay_trusted_contact`) skip approval and settle immediately. Always inform the user and suggest checking `bipa_history` later.

5. **Amounts.** MCP takes `amount_cents` (integer, must be > 0). CLI takes `--amount` in BRL (e.g., `12.34` or `12,34`) or `--amount-cents`. "R$50" = `amount_cents: 5000`.

6. **Idempotency is automatic.** The system generates unique keys internally. No need to pass `request_id` or `idempotency_key`.

7. **Never search Pix keys without payment intent.** Only resolve keys when the user explicitly wants to pay.

## Workflows

### Pay Someone by Pix Key

User: "Pay joao@email.com R$20 for lunch"

**MCP:**
```json
{
  "name": "bipa_pix_pay_key",
  "arguments": {
    "key": "joao@email.com",
    "amount_cents": 2000,
    "note": "lunch",
    "agent_message": "User asked me to pay João R$20 for lunch"
  }
}
```

Response:
```json
{
  "status": "awaiting_approval",
  "key": "joao@email.com",
  "owner_name": "João Silva",
  "amount_brl": "R$ 20.00",
  "note": "lunch",
  "message": "Payment submitted. The user must approve this operation in the Bipa app. Use bipa_history to check the transaction status."
}
```

Tell the user: *"I've submitted the R$20 payment to João Silva. Please approve it in the Bipa app."*

**CLI:**
```bash
bipa pix pay --key joao@email.com --amount 20 --note "lunch" --agent-message "Paying João for lunch"
```

### Pay a QR Code / Copia e Cola

**Preferred: use `bipa_pix_pay_brcode` (preview + transfer in one call):**
```json
{
  "name": "bipa_pix_pay_brcode",
  "arguments": {
    "brcode": "<COPIA_E_COLA>",
    "agent_message": "Paying invoice from QR code"
  }
}
```
For fixed-amount codes, `amount_cents` is optional (taken from the code). For custom-amount codes, pass `amount_cents`.

**Preview first (when you need to show details before paying):**
1. `bipa_pix_brcode_preview` with the raw string → returns `name`, `document_masked`, `total_value_cents`, `kind` (dynamic/static), `amount_type` (fixed/custom)
2. Show the user: recipient name, masked document, and amount
3. `bipa_pix_pay_brcode` with the same brcode and `agent_message`

**CLI:**
```bash
bipa pix pay --brcode "<COPIA_E_COLA>" --agent-message "Paying invoice from QR code"
bipa pix pay --brcode "<COPIA_E_COLA>" --amount 50 --agent-message "Custom amount payment"
```

**Offline decode (no auth needed):**
- `bipa_pix_brcode_decode` parses the TLV structure locally without server contact

### Pay a Trusted Contact

Trusted contacts are pre-approved recipients — no Pix key lookup needed, and payment skips biometric approval.

1. `bipa_pix_trusted_contacts` → list all enabled contacts with `id`, `name`, `document_masked`, `limit_cents`, and bank details
2. Find the right contact and confirm with the user
3. `bipa_pix_pay_trusted_contact` with `pix_trusted_contact_id`, `amount_cents`, and `agent_message`

**MCP:**
```json
{
  "name": "bipa_pix_trusted_contacts",
  "arguments": {}
}
```

```json
{
  "name": "bipa_pix_pay_trusted_contact",
  "arguments": {
    "pix_trusted_contact_id": 42,
    "amount_cents": 5000,
    "agent_message": "User asked to pay Maria R$50 — trusted contact"
  }
}
```

### Pay a Saved Recipient (Preferred for Repeat Payments)

When the user asks to pay someone by name (not by Pix key), check their saved recipients first. This is faster than a Pix key lookup and covers most "pay X" requests.

1. `bipa_pix_recipient_suggestions` → returns the user's top 10 most frequent/recent PIX recipients
2. Match by `name` or `bank_name` — confirm with the user ("Is this João at Nubank?")
3. `bipa_pix_pay_recipient` with the recipient's `id` (as `pix_payment_recipient_id`), `amount_cents`, and `agent_message`

If no match is found in the suggestions list, fall back to `bipa_pix_pay_key` with the user's Pix key.

**MCP:**
```json
{
  "name": "bipa_pix_recipient_suggestions",
  "arguments": {}
}
```

```json
{
  "name": "bipa_pix_pay_recipient",
  "arguments": {
    "pix_payment_recipient_id": 42,
    "amount_cents": 2000,
    "agent_message": "User asked to pay João R$20 for lunch — saved recipient"
  }
}
```

**CLI:**
```bash
bipa pix recipient-suggestions
bipa pix pay --recipient 42 --amount 20 --agent-message "Paying João for lunch"
```

### Pay by BIPA Tag

BIPA tags are user nicknames starting with `$` (e.g. `$bipatag`).

1. `bipa_pix_tag_preview` → confirm recipient name, masked CPF
2. `bipa_pix_pay_tag` with `tag`, `amount_cents`, and `agent_message`

**MCP:**
```json
{
  "name": "bipa_pix_pay_tag",
  "arguments": {
    "tag": "$bipatag",
    "amount_cents": 1000,
    "agent_message": "User asked to send R$10 to $bipatag"
  }
}
```

### Pay a Bank Slip (Boleto)

User shares a digitable line or barcode.

1. `bipa_bank_slip_preview` with the line/barcode in `input` → returns recipient, amount, `kind` (static/dynamic), due date. Dynamic slips also return `min_amount`/`max_amount`.
2. Show the user recipient + amount + due date.
3. `bipa_pay_bank_slip` with `input` (and `amount_cents` only for `custom`/dynamic slips). This creates a **payment request** — the response is `status: awaiting_user_approval` with an `approval_id`. Tell the user to open the Bipa app and approve it.

**CLI:**
```bash
bipa bank-slip preview "<DIGITABLE_LINE>"
bipa bank-slip pay "<DIGITABLE_LINE>"                 # static (fixed) slip
bipa bank-slip pay "<DIGITABLE_LINE>" --amount-cents 5000   # dynamic slip, partial amount
```

### Review Registered Bills (DDA)

DDA (Débito Direto Autorizado) delivers a subscribed user's boletos into Bipa so they can be reviewed without typing barcodes.

1. `bipa_dda_list` → each bill has `id`, `status` (overdue/due/paid/canceled), `recipient`, `amount`, `due_date`. Returns an empty list when the user isn't subscribed to DDA.
2. Summarize what's due/overdue. To pay one, use the bill's barcode with the bank-slip flow above.

**CLI:** `bipa dda list`

### Schedule a Payment

Any `bipa_pay` call accepts a `schedule` object for future or recurring transfers.

```json
{
  "name": "bipa_pay",
  "arguments": {
    "key": "joao@email.com",
    "amount_cents": 5000,
    "agent_message": "User asked to pay rent on the 1st each month",
    "schedule": { "date": "2026-07-01", "frequency": "monthly", "count": 12 }
  }
}
```

`frequency` is one of `once`, `daily`, `weekly`, `monthly`; `count` is the number of executions (default 1). CLI: `--schedule-date`, `--schedule-frequency`, `--schedule-count`.

### Prices & Portfolio

- `bipa_tickers` → current BTC/BRL, USDT/BRL, BTC/USDT bid/ask.
- `bipa_btc_prices` / `bipa_usdt_prices` → historical price series for charts.
- `bipa_portfolio` → P&L, trade stats, and balance history for an asset and period.

**CLI:** `bipa prices tickers`, `bipa prices btc`, `bipa prices usdt`, `bipa portfolio`

### Financial Snapshot

1. `bipa_balance` → `available` and `savings` (cofrinho), formatted BRL
2. `bipa_history` (limit: 20) → recent transactions with `direction` (credit/debit), amounts, counterparty names, timestamps (BRT)

Present a clean summary: available and cofrinho balances in R$, last transactions grouped by direction.

### Detect Recurring / Duplicate Transactions

1. `bipa_history` with `limit: 50` (maximum)
2. Group by counterparty name and direction
3. Flag same-counterparty + same-amount entries

### Check Limits Before Large Transfers

`bipa_limits` returns `daily_limit_cents`, `nightly_limit_cents`, and reset timeframes. Present these before big payments so the user knows if it'll go through.

## Error Handling

| Error | What to do |
|---|---|
| `recipient key was not found` | Ask user to verify the key |
| `recipient key is flagged as fraudulent` | Do not retry. Inform user. |
| `recipient key lookup was rate-limited` | Wait and retry |
| `amount_cents must be greater than zero` | Check amount conversion |
| `agent_message is required` | Always include why the agent is paying |
| `rate limit exceeded` | Wait `retry_after_seconds` then retry |
| `no active Bipa CLI session` | Reauthenticate, then retry — see [Reauthenticate mid-session](#reauthenticate-mid-session) |
| `session expired` | Reauthenticate, then retry — see [Reauthenticate mid-session](#reauthenticate-mid-session) |

## Transaction Statuses

- `awaiting_approval` — Pix payment waiting for user approval in Bipa app
- `awaiting_user_approval` — bank slip (boleto) payment request waiting for in-app approval
- `scheduled` — approved, queued for settlement
- `succeeded` / `confirmed` — settled
- `pending` — in progress
- `failed` — transfer failed
- `created` — initial state

## Key Facts

- All amounts in BRL. MCP uses cents (integer). CLI accepts BRL decimals and cents.
- Transactions show `credit` (in) and `debit` (out) directions.
- Timestamps in BRT (UTC-3).
- Rate limits: payment tools (Pix pay, BR Code pay, bank slip pay) share a 5/min bucket; preview/decode tools (BR Code decode/encode/preview, bank slip preview) each have their own 20/min bucket.
- Credentials stored in OS keychain (macOS Keychain, Windows Credential Manager).
- Transaction details include structured `sections` with labeled fields for full receipt info.
- Payments via Bipa CLI are Pix and bank slips/boletos (including DDA-registered bills). It also exposes read-only multi-asset views: BTC/USDT balances, prices, portfolio, and a unified timeline. Crypto swaps/sends are not yet available via MCP.
