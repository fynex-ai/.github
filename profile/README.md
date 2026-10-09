<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/fynex-ai/.github/main/profile/fynex-logo-dark.png">
  <img src="https://raw.githubusercontent.com/fynex-ai/.github/main/profile/fynex-logo.png" alt="Fynex" width="240">
</picture>

### The intelligence layer for finance, run by AI agents

Most finance tools execute and stop. Fynex runs the whole money chain — payments in, payouts out,
reconciliation back to your books — and reasons across it.

[![API docs](https://img.shields.io/badge/API%20docs-9EFBCD?style=for-the-badge&labelColor=9EFBCD)](https://docs.fynex.ai)
[![Payments OpenAPI](https://img.shields.io/badge/Payments%20OpenAPI-5a5570?style=for-the-badge&labelColor=5a5570)](https://docs.fynex.ai/payments-api/v2/openapi.json)
[![Billing OpenAPI](https://img.shields.io/badge/Billing%20OpenAPI-5a5570?style=for-the-badge&labelColor=5a5570)](https://docs.fynex.ai/billing-api/v1/openapi.json)
[![fynex.ai](https://img.shields.io/badge/fynex.ai-5a5570?style=for-the-badge&labelColor=5a5570)](https://fynex.ai)
[![Dashboard](https://img.shields.io/badge/Dashboard-5a5570?style=for-the-badge&labelColor=5a5570)](https://dashboard.fynex.ai)

</div>

---

## Start here

Three ways in, shortest first.

| | Route | Good for | PCI scope |
|---|---|---|---|
| **1** | **[Hosted checkout](https://docs.fynex.ai/payments-api/v2/docs/hosted-checkout)** — one `POST`, redirect to the URL we return | Getting live today | Stays with us |
| **2** | **Server-to-server** — `initialize-payment` → `finalize-payment` | Full control of the payment flow | Yours |
| **3** | **[WooCommerce plugin](https://github.com/fynex-ai/fynex-plugin-woo-commerce)** — drop-in gateway | WooCommerce stores | Stays with us |

### Hosted checkout in one call

```bash
curl -X POST https://api.fynex.ai/payments-api/v1/checkout \
  -H "Authorization: Bearer $FYNEX_SECRET_KEY" \
  -H "Idempotency-Key: order-1042" \
  -H "Content-Type: application/json" \
  -d '{
    "externalOrderRef": "order-1042",
    "amount": 49.99,
    "currencyCode": "GBP",
    "returnUrls": {
      "success": "https://example.com/thanks",
      "failure": "https://example.com/oops"
    }
  }'
```

```json
{ "sessionId": "…", "checkoutUrl": "https://…", "expiresAt": "…" }
```

Redirect the customer to `checkoutUrl`. The result arrives on your webhook, or poll the session.

> **Amounts are in major units.** `49.99` is £49.99, not 49.99 pence. Sessions are idempotent on
> `Idempotency-Key`, so a retried request returns the same session instead of charging twice.

---

## Two APIs, one key

Fynex has two public REST surfaces on `https://api.fynex.ai`. The **same seller secret key**
authenticates both, as `Authorization: Bearer sk_live_…` (or `sk_test_…`). You mint it in the
Dashboard under Integration → API keys ([how](https://docs.fynex.ai/keys)). There is no second token.

### Payments API: moves money

**61 operations** under `/payments-api/v1/`.

| Area | What you can do | Ops |
|---|---|:--:|
| **Payments & checkout** | Hosted sessions, server-to-server initialize/finalize, status, capture, refunds, allowed methods | 11 |
| **Split payments** | Create, calculate, preview, activate, clone split rules, and explain a settled payment's split | 11 |
| **Payees & payout methods** | Create payees, attach bank details, pay a payee in one call | 15 |
| **Payouts** | Create, list, get, cancel while awaiting approval | 4 |
| **Wallets** | Balances, fund a cashout wallet, transaction history | 5 |
| **Top-up invoices** | Raise, cancel, mark paid, fetch the printable document | 7 |
| **Webhooks** | Register endpoints, manage the IP allowlist | 7 |
| **Device intelligence** | Mint a Sumsub device token for risk signals | 1 |

**[Docs](https://docs.fynex.ai/payments-api/v2/docs)** ·
**[OpenAPI](https://docs.fynex.ai/payments-api/v2/openapi.json)**

### Billing API: bills for what you sell

**42 operations** under `/billing-api/v1/`. It covers contracts, subscriptions, metered usage,
invoices and credits.

| Area | What you can do | Ops |
|---|---|:--:|
| **Customers & contracts** | Create customers and contracts, amend a contract by appending a version | 4 |
| **Subscriptions** | Create, change plan, pause, resume, end trial, cancel | 8 |
| **Usage metering** | Define metrics, send events (single, batch or CSV), price them, redrive dead letters, usage webhooks | 18 |
| **Invoices** | Raise, send, preview the upcoming one, export, PDF, settlement status | 8 |
| **Credits** | Balances and top-ups | 3 |
| **Sandbox** | Test clock: advance a contract through time | 2 |

**[Docs](https://docs.fynex.ai/billing-api/v1/docs)** ·
**[OpenAPI](https://docs.fynex.ai/billing-api/v1/openapi.json)**

Both specs are generated from the running service, so they cannot drift from the code. They are
public: you don't need a key to read them.

> **The two APIs use different money units.** Billing uses integer minor units throughout: a field
> ending in `Minor` takes `4999` for €49.99. Payments takes **major** units. Read the field name
> each time; don't assume one API follows the other. Neither API rejects the wrong unit, so a
> guess can be off by a factor of a hundred.

Both APIs send webhooks through the same signed pipe: one endpoint registration and one
`X-Fynex-Signature` scheme. Deduplicate events on `eventId`.

---

## Split payments, explained rather than asserted

Marketplaces and platforms rarely keep the whole payment. Fynex treats the split as a rule you can
reason about before it runs and audit after:

- **`POST /split-rules/calculate`** — see the arithmetic on a hypothetical amount without creating anything
- **`POST /split-rules/{id}/preview-activation`** — see what activating would change before it changes
- **`GET /payments/{id}/split-decisions`** — ask a settled payment how it was split, and why

---

## Test and live

Your key decides which mode you're in. A `sk_test_…` key cannot move real money. After go-live the
test key answers `401`. **[Going live](https://docs.fynex.ai/payments-api/v2/docs/going-live)**
covers the steps between the two.

---

## Building with an AI agent?

The docs are written to be read by coding agents as well as people:

- **[`llms.txt`](https://docs.fynex.ai/llms.txt)** is the front door. It lists the mistakes this
  platform has actually seen, then links every guide as markdown.
- **[`llms-full.txt`](https://docs.fynex.ai/llms-full.txt)** has both complete references in one file.
- **[Agent skills](https://docs.fynex.ai/.well-known/skills/index.json)** is a skills index your agent can discover.

---

## Open source

| Repo | What it is |
|---|---|
| **[fynex-plugin-woo-commerce](https://github.com/fynex-ai/fynex-plugin-woo-commerce)** | Official Fynex hosted-checkout gateway for WooCommerce |

Most of the platform is private. If you are integrating and something in the docs is wrong,
ambiguous or missing, open an issue on the plugin repo or reach us
through [fynex.ai](https://fynex.ai) — a concrete report about the API is genuinely useful to us.

---

## One thing worth knowing about the intelligence layer

It recommends; it does not act on your behalf. Every recommendation opens to the reasoning behind
it — the numbers used, the rule applied, the confidence — and **anything that moves money waits for
your approval**. You can act in one click, snooze it, or dismiss it, and every transition is
written to the audit log.

<div align="center">

**[Docs](https://docs.fynex.ai)** · **[fynex.ai](https://fynex.ai)** ·
**[Dashboard](https://dashboard.fynex.ai)** · **[Get in touch](https://fynex.ai)**

</div>
