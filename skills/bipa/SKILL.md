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
- **Crypto rails** — Bitcoin, Lightning Network, Ethereum, Polygon, Solana, XRP Ledger, Base and BNB Smart Chain, for BTC, USDT, ETH, SOL, USDC and XRP

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

# Headless OAuth — Bipa sends the user a mobile approval notification (employee pilot)
bipa login --device --phone <PHONE> --agent-name <NAME> --agent-kind <KIND>

# If the agent cannot wait for approval, resume the saved request later
bipa login --device --phone <PHONE> --no-wait --agent-name <NAME> --agent-kind <KIND>
bipa login --device --resume --agent-name <NAME> --agent-kind <KIND>
bipa login --device --cancel --agent-name <NAME> --agent-kind <KIND>
```

`--agent-kind` is the AI platform: `openclaw`, `claude`, `claude_code`, `chatgpt`, `codex`, `cursor`, `antigravity`, `grok`, `gemini`, `bipa`, or `other`.

Running `bipa login` without flags prints a usage summary.

Choosing a method:

- **Headless / chat agents (e.g. OpenClaw):** use `--device` and ask the user to approve the pairing code in the Bipa app. During the pilot, this is available to employees and employee-owned business accounts. Remote non-employee users can temporarily use `bipa login --pin --phone <PHONE> --agent-name <NAME> --agent-kind <KIND>` followed by `bipa verify <PIN>`; the user should provide only the login code, never the app PIN.
- **Agents that can surface a URL:** `--web` prints the auth URL to stdout for you to present to the user.
- **Humans at the keyboard:** `--web --open` opens the browser automatically.

### PF and PJ accounts

A person who owns a company can connect both their personal (PF) and business (PJ) accounts to the
same agent. Log in once per account and choose the account during sign-in (`bipa verify --account pf|pj`
for PIN logins). Each login is saved next to the accounts already saved; it never replaces them.

```bash
bipa accounts -f json          # saved accounts: account (user_id, account_kind, display_name),
                               # default, session_status, select_with, issues; unfiled_login
bipa pix balance --account pj  # act on a saved account: pf, pj, or a USER_ID
BIPA_ACCOUNT=pj bipa pix history  # same, from the environment
bipa accounts use pj           # make the PJ the default for commands without --account
bipa logout --account pj       # sign out of and forget only the PJ
```

`unfiled_login: true` means a login finished but Bipa could not confirm its account, so it was not
saved; run the login again. `bipa login` itself takes no `--account`: the account is the one chosen
during sign-in. A finished login reports where it was saved on stderr only; run `bipa accounts -f json` to read it.
Run one login at a time; a second login started before the first finishes
replaces its pending request. `BIPA_ACCOUNT` applies only to commands that act on an account, and
`bipa mcp install` pins an account only from an explicit `--account`.

Commands without `--account` act on the default account (the first one connected). When the user
has more than one account saved, confirm which one they mean before any payment and pass
`--account` explicitly. `whoami -f json` reports the `account` a session acts on.

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
  stored credentials). Recover it with the reported `bipa login` command.
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
session lapses between requests. Reauthenticate using the reported recovery method. Retry reads after reconnecting; for an uncertain payment submission, inspect the existing operation first using the payment-recovery rules below. `bipa whoami -f json` reports the recovery method:

```json
{ "session_status": "expired", "reauth_required": true,
  "auth_method": "oauth", "last_login_channel": null, "last_login_hint": null,
  "recommended_command": "bipa login --device --phone <PHONE> --agent-name Amy --agent-kind openclaw" }
```

1. Read `session_status` / `reauth_required`. If `reauth_required` is true, run the
   `recommended_command` — it preserves the recorded login flow. A first connection uses public browser OAuth; eligible headless users can choose device OAuth, and existing PIN connections retain their previous channel.
2. **OAuth** (`auth_method: oauth`): follow the reported `--device` or `--web` command. For `--device`, get the user's Bipa phone, replace `<PHONE>`, and start the request. Ask the user to open Bipa's notification and review the connection; present the code or verification URL only when the notification is unavailable. Never request the app PIN.
3. **Existing PIN connection** (`auth_method: pin`): follow the reported legacy command using the previous email or phone channel. Confirm the full identifier with the user if needed.
4. Re-run failed reads. For payment submissions, inspect the existing operation before deciding whether another submission is appropriate; reconnecting does not establish whether the earlier payment was accepted.

The app PIN stays with the human and is entered only in Bipa.

## Set Up MCP for Claude Desktop

```bash
bipa mcp install --client claude
# Restart Claude Desktop to pick up the change
```

The MCP server reuses saved credentials. No tokens in config files.

To give one agent both the PF and the PJ, install one local server per saved account:

```bash
bipa mcp install --client claude --account pf   # server mcpsrv_bipa_pf
bipa mcp install --client claude --account pj   # server mcpsrv_bipa_pj
```

Each server is pinned to its account by user id, states that account in its instructions, and
reports it in `bipa_whoami`. Use the server for the account the user means. A hosted connector
connection acts on the single account chosen when it was authorized.

Remote MCP is also available at `https://mcp.bipa.app/mcp` with automatic OAuth 2.1 (for Claude.ai, ChatGPT, Cursor, VS Code, and other remote-capable clients).

## MCP Tools

### Pix payments

| Tool | Description |
|---|---|
| `bipa_pay` | Submit a Pix payment request by `key` (+ `amount_cents`, `agent_message`); inspect its status rather than assuming completion. Optional `schedule` object (`date`, `frequency` = once/daily/weekly/monthly, `count`) for future or recurring payments. For other destinations use `bipa_pix_pay_recipient` / `_tag` / `_trusted_contact` / `_brcode`. |
| `bipa_pix_pay_key` | Send a Pix payment using a supplied key (lookup + transfer in one call). |
| `bipa_pix_recipient_suggestions` | List top 10 saved recipients with `account:read`. Use when the recipient must be discovered; supplied IDs or PIX keys do not require this lookup. |
| `bipa_pix_pay_recipient` | Pay a saved recipient using a known `pix_payment_recipient_id` supplied by the user or prior authorized context. Discovery through `bipa_pix_recipient_suggestions` requires `account:read`. |
| `bipa_pix_trusted_contacts` | List pre-approved trusted contacts with `account:read` (payments skip biometric approval) |
| `bipa_pix_pay_trusted_contact` | Pay a trusted contact using a known `pix_trusted_contact_id` supplied by the user or prior authorized context. Discovery through `bipa_pix_trusted_contacts` requires `account:read`. Uses existing delegation without fresh approval; inspect the returned status, including manual review or failure. |
| `bipa_pix_payment_status` | Read a PIX outflow request using `request_id` set to the returned `outflow_request_id`. Supports pending approval; schedule IDs, boleto/DDA approval IDs and history transaction IDs are different identifiers. |
| `bipa_pix_tag_preview` | Resolve a BIPA user by tag (e.g. `$bipatag`) and create a server-side preparation record. Does not submit an outgoing payment. |
| `bipa_pix_pay_tag` | Pay a BIPA user by tag |
| `bipa_pix_brcode_decode` | Decode a Pix QR / Copia e Cola string (local, no auth) |
| `bipa_pix_brcode_preview` | Resolve a BR Code and persist a server-side preview record with recipient, amount and payment id. Does not submit an outgoing payment. |
| `bipa_pix_pay_brcode` | Pay a BR Code (preview + transfer in one call). For `fixed` codes omit `amount_cents`; for `custom` codes pass it. |
| `bipa_pix_brcode_encode` | Generate a static Pix Copia e Cola payload from a registered key (+ optional fixed amount). Registers an additive receivable record and reconciles incoming payments back to the agent. Currently authorized through the `account:read` capability; this does not grant outgoing-payment authority. Each call mints a fresh idempotency key — calling twice with the same args creates two distinct BR Codes; cache the response instead of retrying. |
| `bipa_pix_brcode_encode_widget` | Render the QR widget for a BR Code that was just generated. Pass the full output of `bipa_pix_brcode_encode`. |

### Bank slips & bills (boletos)

| Tool | Description |
|---|---|
| `bipa_bank_slip_preview` | Preview a bank slip (boleto) from its digitable line or barcode: recipient, amount, kind (static/dynamic), due date. Dynamic slips return `min_amount`/`max_amount`. Persists server-side decoding/preparation records; does not submit an outgoing payment. |
| `bipa_pay_bank_slip` | Request a boleto payment using `input`, `agent_message` and a retained `idempotency_key`. Accepted requests return `awaiting_user_approval` and `approval_id` for in-app approval. For `custom` slips pass `amount_cents` within min/max; for `fixed` slips omit it. |
| `bipa_dda_list` | With `financial:read`, list registered DDA bills with `id`, `status`, `recipient`, exact `amount_cents`, `amount` and `due_date`. Read `subscription_state` separately; an empty list alone does not establish enrollment. |
| `bipa_dda_pay` | Request a bill payment using a known `id` supplied by the user or prior authorized context, optional `amount_cents`, `agent_message` and optional `idempotency_key`. Returns an approval ID when supported; unavailable server RPCs return `capability_unavailable`. |

### Account, balance & history

| Tool | Description |
|---|---|
| `bipa_balance` | BRL, BTC and USDT wallet balances plus separate BRL savings ("cofrinho"/"caixinha"), with exact units and display strings; nonzero ETH, SOL, USDC and XRP holdings in `other_assets` |
| `bipa_limits` | Transfer risk limits (daily and nightly) |
| `bipa_deposit` | Where to deposit: the Pix keys (default, or asset `brl`), or a crypto address on one network. btc on bitcoin; usdt on ethereum or polygon; eth on ethereum; sol on solana; usdc on ethereum, polygon, solana, base or bsc; xrp on xrp with a `destination_tag`. `network` is required for usdt and usdc |
| `bipa_pix_keys` | List configured Pix keys |
| `bipa_account` | Account profile and metadata |
| `bipa_whoami` | Session status (includes auth method and expiry) |
| `bipa_history` | Pix transaction list (`limit?`, default 20, max 50) or detail by `id` |
| `bipa_transactions` | Multi-asset compact transaction list (BRL, BTC, USDT). Most recent up to `limit` (default 20, max 50). |
| `bipa_transaction_detail` | Detailed info for a specific multi-asset transaction by `id` |
| `bipa_timeline` | Unified activity timeline across all assets and event layers. Supports search, layer filtering, cursor pagination. |

CLI `bipa pix history --agent` and MCP `bipa_history` list/detail outputs include `amount_cents_semantics: "magnitude_with_direction"`. The expected server contract is a nonnegative magnitude: use `direction` to subtract debits or add credits. An unexpected negative raw amount violates that contract; preserve it for diagnosis, do not apply a direction sign again, and report that affected cash-flow calculations are unavailable. An unknown direction likewise has no inferred sign and cannot contribute to a reliable net calculation. Check `status` before treating a row as settled cash flow, and follow Pix pagination/coverage before summarizing a period; pending or failed rows and partial history do not establish a net account balance.

### Prices & portfolio

| Tool | Description |
|---|---|
| `bipa_tickers` | Current bid/ask prices for BTC, USDT, ETH, SOL, USDC and XRP against BRL, and BTC/USDT in USDT |
| `bipa_prices` | Historical price series for one asset (btc, usdt, eth, sol, usdc or xrp); with `candle_hours` (1–168), also buy-price candles over that window |
| `bipa_btc_prices` | BTC/BRL historical price series (the same as `bipa_prices` with asset btc) |
| `bipa_usdt_prices` | USDT/BRL historical price series (the same as `bipa_prices` with asset usdt) |
| `bipa_portfolio` | Portfolio summary (P&L, trade stats, balance history) for one asset (btc, usdt, eth, sol, usdc or xrp) and a period |
| `bipa_portfolio_overview` | The whole crypto wallet's value in reais over a period, now and per bucket. Bipa opens it account by account (`code: portfolio_overview_unavailable` otherwise) |

> Most data tools above have a `_widget` counterpart (e.g. `bipa_balance_widget`). In hosts that render MCP Apps, such as claude.ai and ChatGPT, show the data result to the person by passing it unchanged to the matching widget tool. In text-only hosts, answer from the data tool. Widget tools never fetch new data or submit payments.

## Bipa Business (business API key)

A company on Bipa Business can give an agent a **business API key** instead of a `bipa login`. It
reaches one business account through the public Business API v1. It is separate from the PF/PJ
session: different commands, a different MCP server, and its own permissions.

```bash
bipa business login <BUSINESS_ACCOUNT_ID>   # the user types the key at a hidden prompt; or --key-stdin
bipa business whoami -f json                # environment, account, key hint, financial:read check
bipa mcp install --business                 # MCP server mcpsrv_bipa_business with bipa_business_* tools
```

- A member creates the key in Bipa Business (Configurações › Desenvolvedores › Chaves) and picks its
  permissions there. Never ask for the key in chat, never pass it as an argument, never echo it.
- The API does not report a key's permissions. A `403 permission_denied` means the key lacks the
  permission **or** the business account id is wrong; say both.
- **A key never approves a payment.** `bipa business requests create` /
  `bipa_business_payment_request_create` only asks for one. Always tell the user it waits for a
  member's approval in Bipa Business; no money leaves before that.
- Pay in three steps: preview (`preview pix|boleto`, `bipa_business_pix_preview` /
  `bipa_business_boleto_preview`), confirm recipient and amount with the user, create with the
  `preview_id`. Follow with `requests get|wait` (`bipa_business_payment_request[_wait]`); cancel with
  `requests cancel` while it is pending.
- Keep the `idempotency_key` a write returns. After a timeout or 5xx, retry the same write with the
  same key (`--idempotency-key`, or the `idempotency_key` input); never with changed details.
- MCP amounts are BRL cents in, `{min_units, decimal_scale}` out. Decide on the error `code`.
- **Conversions, accepting a target order and savings movements run at once, with no approval.**
  Payments still wait for an approver; these do not. The key's creator chose its permissions:
  `conversions:execute` allows converting and accepting target orders, `savings:manage` allows
  moving savings (cofrinho). Without them the tools answer `permission_denied`.
  - Convert: `bipa_business_conversion_quote`, show the person what they pay, receive and the fee,
    then `bipa_business_conversion_create` with the quote's `conversion_idempotency_key`, the
    quote's asset, side, amount and recurrence (pass the recurrence to the quote too), its
    `counter_amount` min_units as `counter_limit_min_units`, and `confirmed_amount` equal to the
    amount sent.
  - Target order: `bipa_business_target_order_create` (it reserves nothing), confirm the terms, then
    `bipa_business_target_order_accept` with the `order_id`, the same details, the
    `idempotency_key` create returned and `confirmed_amount`. Accepting reserves the funds. If the
    key is refused as not recorded or too old, call create again with the same details and the same
    key, then accept; never make a new order for it.
  - Savings: `bipa_business_savings` returns a `deposit_idempotency_key` and a
    `withdraw_idempotency_key`; confirm the amount, then `bipa_business_savings_deposit` or
    `_withdraw` with the matching key and `confirmed_amount {brl_cents}`.
  - Each key works only for the terms it was issued for, in this session, for 30 minutes. If a
    move timed out, retry it with the same key; if the key was refused as unknown, check
    `bipa_business_statement_entries` and `bipa_business_orders` before quoting again.
  - Send `confirmed_amount` only after the person confirmed that exact amount. A mismatch is refused
    and nothing is sent. In the CLI (`convert create`, `target-order create --accept`,
    `savings deposit|withdraw`) the person confirms with y/N in their terminal; never pass `--yes`
    for them.
- Receiving needs no approval and sends nothing: `links create|deactivate`
  (`bipa_business_payment_link_create` / `_deactivate`), `deposit-address`
  (`bipa_business_deposit_address_request`). Confirm the title, methods and amount first. Cancel a
  recurring or target order with `cancel-order` (`bipa_business_order_cancel`).
- There is no sandbox for business keys yet: `--sandbox` fails. Every call is production.
- The key is not tied to a person and outlives the member who made it. When someone leaves, a member
  revokes the key in Bipa Business; `bipa business logout` only forgets it on this computer.

Reads: `balances`, `address`, `statement list|get|export`, `monthly list|request|file`,
`income-report`, `orders`, `links list|get|payment`, `pix-keys`, `counterparties`, `banks`,
`trusted-counterparties`, `requests list`, `assistant enrollment|boletos|boleto`, `savings`,
`limits` — all under `bipa business`, all with `-f json`. Files are CLI-only. Guide:
`docs/BUSINESS_API.md`.

## CLI Commands

Before each new payment, generate and retain a UUIDv7. Pass it as `--idempotency-key <UUIDV7>` to any `pix pay`, `bank-slip pay` or `dda pay` command below.

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
bipa pix history [--limit N] [--cursor CURSOR] [TRANSACTION_ID]
bipa pix status <OUTFLOW_REQUEST_ID>
bipa pix wait <OUTFLOW_REQUEST_ID> [--timeout-seconds 60]
bipa pix keys
bipa pix deposit                              # Pix keys
bipa pix deposit --asset usdc --network base  # crypto address on one network
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
bipa accounts                # saved PF/PJ accounts; `bipa accounts use <pf|pj|USER_ID>`
bipa <command> --account pj  # any command on a saved account other than the default
bipa logout [--account pj]
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

1. **Discover recipients only when needed and permitted.** Use a supplied recipient ID, trusted-contact ID, or PIX key directly with the corresponding payment tool and its required payment permission. When the target must be discovered and `account:read` is granted, use the appropriate recipient/contact list. Without discovery permission, ask for a known ID or PIX key; a catalog read is not a prerequisite for submitting that payment.

2. **`bipa_pix_pay_key` includes recipient lookup.** `bipa_pix_pay_key` resolves the recipient automatically. Never try to look up a Pix key separately before paying.

3. **`agent_message` is required for every payment.** This text is shown to the human in the Bipa app during the approval step. Write a clear, honest explanation of why the agent is making this payment (e.g., "User asked me to pay João R$20 for lunch"). Payments without `agent_message` are rejected.

4. **Submission is not settlement.** An accepted ordinary PIX request can return `awaiting_approval` and an `outflow_request_id`; tell the user to approve it in Bipa. Query `bipa_pix_payment_status` with `request_id: <outflow_request_id>` to follow pending approval and later outcomes. Trusted contacts use existing delegation without fresh approval, but may still fail or require manual review. Keep `schedule_approval_id`, `schedule_id` and boleto/DDA `approval_id` distinct: follow those in the Bipa app, not the PIX status RPC. History is transaction data, never a substitute for looking up a pending approval.

5. **Amounts.** MCP takes `amount_cents` (integer, must be > 0). CLI takes `--amount` in BRL (e.g., `12.34` or `12,34`) or `--amount-cents`. "R$50" = `amount_cents: 5000`.

6. **Retain payment identity before submission.** Generate a fresh UUIDv7 for each distinct payment and pass `idempotency_key` to MCP payment tools, or `--idempotency-key` to `pix pay`, `bank-slip pay` and `dda pay`. Omitting it generates a new key for compatibility. Retain the effective key and returned operation/approval IDs. After an uncertain response, inspect that existing operation before retrying; if its ID is unavailable, reconcile through the Bipa app or support. Never reuse a key with changed account, recipient, amount or schedule, or create a replacement merely because a response was lost. Durable server payload-bound replay is not guaranteed: the existing PIX account+UUID lookup can return the original operation for changed details, and approval UUID conflicts do not necessarily recover the original response.

7. **Never search Pix keys without payment intent.** Only resolve keys when the user explicitly wants to pay.

## Workflows

Payment keys in these examples are illustrative. Generate a fresh UUIDv7 for each real payment, record it before submission, and keep it with the returned IDs.

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
    "agent_message": "User asked me to pay João R$20 for lunch",
    "idempotency_key": "0198fa10-0000-7000-8000-000000000042"
  }
}
```

Example `data` from the response envelope:
```json
{
  "idempotency_key": "0198fa10-0000-7000-8000-000000000042",
  "outflow_request_id": 42,
  "status": "awaiting_approval",
  "key": "joao@email.com",
  "owner_name": "João Silva",
  "amount_brl": "R$ 20,00",
  "note": "lunch",
  "message": "Payment submitted. The user must approve this operation in the Bipa app. Use bipa_pix_payment_status with outflow_request_id to check the request status."
}
```

Tell the user: *"I've submitted the R$20 payment to João Silva. Please approve it in the Bipa app."* Then query `bipa_pix_payment_status` with `{"request_id":42}` when checking this request. A history transaction ID is a different identifier.

**CLI:**
```bash
bipa pix pay --key joao@email.com --amount 20 --note "lunch" --agent-message "Paying João for lunch" \
  --idempotency-key 0198fa10-0000-7000-8000-000000000042
bipa pix status 42 --agent
```

### Pay a QR Code / Copia e Cola

**Preferred: use `bipa_pix_pay_brcode` (preview + transfer in one call):**
```json
{
  "name": "bipa_pix_pay_brcode",
  "arguments": {
    "brcode": "<COPIA_E_COLA>",
    "agent_message": "Paying invoice from QR code",
    "idempotency_key": "0198fa10-0000-7000-8000-000000000043"
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

Trusted contacts use the existing server delegation without a fresh approval and require `payments:execute_trusted`. With a supplied trusted-contact ID, call the payment tool directly. The discovery sequence below applies only when the ID must be found and `account:read` is granted. A submission may still be pending, fail or require manual review; use the returned `outflow_request_id` to query the PIX status.

1. `bipa_pix_trusted_contacts` → list all enabled contacts with `id`, `name`, `document_masked`, `limit_cents`, and bank details
2. Find the right contact and confirm with the user
3. `bipa_pix_pay_trusted_contact` with `pix_trusted_contact_id`, `amount_cents`, and `agent_message`

**MCP discovery (when needed, with `account:read`):**
```json
{
  "name": "bipa_pix_trusted_contacts",
  "arguments": {}
}
```

**MCP payment with a supplied ID:**
```json
{
  "name": "bipa_pix_pay_trusted_contact",
  "arguments": {
    "pix_trusted_contact_id": 42,
    "amount_cents": 5000,
    "agent_message": "User asked to pay Maria R$50 — trusted contact",
    "idempotency_key": "0198fa10-0000-7000-8000-000000000044"
  }
}
```

### Pay a Saved Recipient (Preferred for Repeat Payments)

A supplied recipient ID or PIX key can be used directly with the corresponding payment-request tool and `payments:request`. The discovery sequence below applies only when the target must be found and `account:read` is granted.

1. `bipa_pix_recipient_suggestions` → returns the user's top 10 most frequent/recent PIX recipients
2. Match by `name` or `bank_name` — confirm with the user ("Is this João at Nubank?")
3. `bipa_pix_pay_recipient` with the recipient's `id` (as `pix_payment_recipient_id`), `amount_cents`, and `agent_message`

If no match is found in the suggestions list, fall back to `bipa_pix_pay_key` with the user's Pix key.

**MCP discovery (when needed, with `account:read`):**
```json
{
  "name": "bipa_pix_recipient_suggestions",
  "arguments": {}
}
```

**MCP payment with a supplied ID:**
```json
{
  "name": "bipa_pix_pay_recipient",
  "arguments": {
    "pix_payment_recipient_id": 42,
    "amount_cents": 2000,
    "agent_message": "User asked to pay João R$20 for lunch — saved recipient",
    "idempotency_key": "0198fa10-0000-7000-8000-000000000045"
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
    "agent_message": "User asked to send R$10 to $bipatag",
    "idempotency_key": "0198fa10-0000-7000-8000-000000000046"
  }
}
```

### Pay a Bank Slip (Boleto)

User shares a digitable line or barcode.

1. `bipa_bank_slip_preview` with the line/barcode in `input` → returns recipient, amount, `kind` (static/dynamic), due date. Dynamic slips also return `min_amount`/`max_amount`.
2. Show the user recipient + amount + due date.
3. `bipa_pay_bank_slip` with `input`, `agent_message`, a retained `idempotency_key`, and `amount_cents` only for `custom`/dynamic slips. An accepted request returns `status: awaiting_user_approval` and an `approval_id`. Ask the user to approve it in Bipa; retain this ID for reconciliation. Boleto approval IDs cannot be queried with `bipa_pix_payment_status`, and history is not a pending-approval lookup.

**CLI:**
```bash
bipa bank-slip preview "<DIGITABLE_LINE>"
bipa bank-slip pay "<DIGITABLE_LINE>"                 # static (fixed) slip
bipa bank-slip pay "<DIGITABLE_LINE>" --amount-cents 5000   # dynamic slip, partial amount
```

### Review Registered Bills (DDA)

DDA (Débito Direto Autorizado) delivers a subscribed user's boletos into Bipa so they can be reviewed without typing barcodes. Inventory review requires `financial:read`. A supplied bill ID can be passed directly to `bipa_dda_pay` with `payments:request` without listing the inventory.

1. `bipa_dda_list` → inspect bills and the separate `subscription_state`; an empty list does not establish enrollment.
2. Summarize what is due or overdue. If requested, call `bipa_dda_pay` with the bill `id`, `agent_message` and a retained `idempotency_key`; accepted requests return an `approval_id` for the Bipa app. A missing server capability is not authorization to substitute another payment path.

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
    "idempotency_key": "0198fa10-0000-7000-8000-000000000047",
    "schedule": { "date": "2026-07-01", "frequency": "monthly", "count": 12 }
  }
}
```

`frequency` is one of `once`, `daily`, `weekly`, `monthly`; `count` is the number of executions (default 1). CLI: `--schedule-date`, `--schedule-frequency`, `--schedule-count`. A `schedule_approval_id` still awaits in-app approval; a `schedule_id` identifies a created schedule. Follow schedules in Bipa. Only an actual `outflow_request_id` can be queried with `bipa_pix_payment_status`.

### Crypto deposits

- `bipa_deposit` with `asset` and `network` → the address that receives that asset on that network. Repeat the asset and network to the person: another asset, or the same asset on another network, can be lost.
- XRP: always give both `address` and `destination_tag`. Bipa shares the address across accounts, so a deposit without the tag is not credited.
- An error with `code: app_session_only` means Bipa doesn't serve deposit addresses to CLI and agent sessions yet. The session is fine: say the Bipa app shows the address, and don't ask the person to sign in again. `code: deposit_network_unavailable` means that network isn't taking deposits right now; offer another network the asset arrives on. `code: business_account` means a company's crypto deposit addresses aren't served here (its Pix keys still are); `code: not_open_to_account` means that asset's deposits aren't open to this account yet. `code: account_unavailable` means the account can't receive deposits: point the person to Bipa support, not to another network or a new sign-in.

**CLI:** `bipa pix deposit --asset <asset> [--network <network>]`

### Prices & Portfolio

- `bipa_tickers` → current bid/ask for every listed pair (`tickers[].pair`, such as `btc_brl`, `eth_brl` or `btc_usdt`).
- `bipa_prices` → historical price series for charts, for any of the six assets. Pass `candle_hours` for candles: they need a signed-in session, and when this session can't read them `candles.status` is `unavailable` with a `reason` (`app_session_only` or `sign_in_required`) while the series still answer. `bipa_btc_prices` and `bipa_usdt_prices` are the same for BTC and USDT.
- `bipa_portfolio` → P&L, trade stats, and balance history for an asset and period. Read quantities from `asset_balance_amount` and `asset_quantity_amount`; the integer `asset_balance` and `asset_quantity` are smallest units for BTC and USDT only, and zero for ETH, SOL, USDC and XRP.

- `bipa_portfolio_overview` → the whole wallet in reais over a period. When it answers `code: portfolio_overview_unavailable`, it isn't open to this account yet: say so, and offer `bipa_portfolio` for each asset instead.

**CLI:** `bipa prices tickers`, `bipa prices history <asset> [--candle-hours N]`, `bipa prices btc`, `bipa prices usdt`, `bipa portfolio --asset <asset>`, `bipa portfolio --overview [--period <period>]`

### Financial Snapshot

1. `bipa_balance` → `brl.total`, `btc.total`, `usdt.total` for wallet balances, and `brl.savings` for separate cofrinho savings. Use `brl.total_cents`, `btc.total_sats` and `usdt.total_usdtmicros` for exact calculations. Other holdings (ETH, SOL, USDC, XRP) appear in `other_assets` with `total` and the exact `total_min_units`/`decimal_scale`; they carry no BRL value unless `total_brl` is present.
2. `bipa_history` (limit: 20) → recent transactions with `direction` (credit/debit), amounts, counterparty names, timestamps (BRT)

Present wallet balances by asset, cofrinho savings separately, and recent transactions grouped by direction. The `available` fields preserve legacy card-related server values and can be zero for accounts without cards. Payment endpoints determine funds, limits and authorization; a balance snapshot alone does not establish payment eligibility.

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
| `rate limit exceeded` | Respect `retry_after_seconds`; inspect an uncertain payment outcome before resubmitting |
| `no active Bipa CLI session` | Reauthenticate, then retry reads; reconcile uncertain payments — see [Reauthenticate mid-session](#reauthenticate-mid-session) |
| `session expired` | Reauthenticate, then retry reads; reconcile uncertain payments — see [Reauthenticate mid-session](#reauthenticate-mid-session) |
| `No saved Bipa account matches --account …` (a pinned MCP server fails to start) | Run `bipa accounts` to list and identify saved accounts, then log in to the missing one or reinstall the server with the right `--account` |
| `Your saved accounts changed while this command started` | Another command changed the default account; retry |
| `… is saved twice` / `should hold Bipa user …` | A credential file was copied or edited by hand; follow the message to delete the extra file |

## Transaction Statuses

- `awaiting_approval` — Pix payment waiting for user approval in Bipa app
- `awaiting_user_approval` — pending in-app approval in PIX status or boleto/DDA request responses
- `approved` — approval recorded; PIX processing remains pending
- `denied_by_user` — user denied the PIX request
- `manual_review` — support review required; not proof of funds sent
- `unknown` — absent or unrecognized status; no successful outcome may be inferred
- `sent` — server-reported PIX send outcome; not a receipt lookup
- `scheduled` — schedule created, not proof of an executed payment; `schedule_approval_id` means it still awaits approval
- `succeeded` / `confirmed` — settled
- `pending` — in progress
- `failed` — transfer failed
- `created` — initial state

## Key Facts

- All amounts in BRL. MCP uses cents (integer). CLI accepts BRL decimals and cents.
- Transactions show `credit` (in) and `debit` (out) directions.
- Timestamps in BRT (UTC-3).
- Rate limits per server instance: payment submissions (Pix, boleto and DDA) share a 5/min bucket; BR Code decode/encode/preview, tag preview and bank slip preview each have their own 20/min bucket.
- Credentials use the OS keychain by default on macOS/Windows; Linux defaults to a file. `BIPA_CREDENTIALS_PATH` selects an explicit file store for unattended agents. Failed keychain reads or writes preserve the selected storage and report an error; `bipa doctor --agent` reports `storage_kind` without exposing tokens or opening the keychain.
- Transaction details include available structured `sections`. Complete receipt linkage from every payment request remains unavailable.
- Payments via Bipa CLI are Pix and bank slips/boletos (including DDA-registered bills). It also exposes read-only multi-asset views: BTC/USDT balances, prices, portfolio, and a unified timeline. Crypto swaps/sends are not yet available via MCP.
