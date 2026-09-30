<div align="center">

<img src="https://raw.githubusercontent.com/fynex-ai/.github/main/profile/fynex-logo.png" alt="Fynex" width="240">

### The intelligence layer for finance, run by AI agents

Most finance tools execute and stop. Fynex runs the whole money chain — payments in, payouts out,
reconciliation back to your books — and reasons across it.

[![API docs](https://img.shields.io/badge/API_docs-live_OpenAPI-2c2740?style=for-the-badge)](https://api.fynex.ai/payments-api/v2/docs)
[![OpenAPI](https://img.shields.io/badge/openapi.json-3.1-3a3550?style=for-the-badge)](https://api.fynex.ai/payments-api/v2/openapi.json)
[![Website](https://img.shields.io/badge/fynex.ai-website-5a5570?style=for-the-badge)](https://fynex.ai)
[![Dashboard](https://img.shields.io/badge/dashboard-sign_in-8b8796?style=for-the-badge)](https://dashboard.fynex.ai)

</div>

---

## Start here

Three ways in, shortest first.

| | Route | Good for | PCI scope |
|---|---|---|---|
| **1** | **[Hosted checkout](https://api.fynex.ai/payments-api/v2/docs)** — one `POST`, redirect to the URL we return | Getting live today | Stays with us |
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

## What the API covers

**60 operations**, all under `https://api.fynex.ai/payments-api/v1/`, authenticated with
`Authorization: Bearer sk_live_…` (or `sk_test_…`).

| Area | What you can do | Ops |
|---|---|:--:|
| **Payments & checkout** | Hosted sessions, server-to-server initialize/finalize, status, capture, refund, allowed methods | 11 |
| **Split payments** | Create, calculate, preview, activate, clone and explain split rules | 10 |
| **Payees & payout methods** | Create payees, attach bank details, pay a payee in one call | 15 |
| **Payouts** | Create, list, get, cancel while awaiting approval | 4 |
| **Wallets** | Balances, fund a cashout wallet, transaction history | 5 |
| **Top-up invoices** | Raise, cancel, mark paid, fetch the printable document | 7 |
| **Webhooks** | Register endpoints, manage the IP allowlist | 7 |
| **Device intelligence** | Mint a Sumsub device token for risk signals | 1 |

Full request and response schemas are in the
**[live OpenAPI docs](https://api.fynex.ai/payments-api/v2/docs)** — generated from the running
service, so they cannot drift from the code. The spec itself is at
[`/payments-api/v2/openapi.json`](https://api.fynex.ai/payments-api/v2/openapi.json), and both are
public: no key needed to read them.

---

## Split payments, explained rather than asserted

Marketplaces and platforms rarely keep the whole payment. Fynex treats the split as a rule you can
reason about before it runs and audit after:

- **`POST /split-rules/calculate`** — see the arithmetic on a hypothetical amount without creating anything
- **`POST /split-rules/{id}/preview-activation`** — see what activating would change before it changes
- **`GET /payments/{id}/split-decisions`** — ask a settled payment how it was split, and why

---

## Test and live

Your key decides which it is: the spec documents seller tokens as `sk_test_…` or `sk_live_…`, and a
test key cannot move real money. The OpenAPI document also lists `https://staging-api.fynex.ai`
alongside production, with the same paths and the same auth header — talk to us about access before
you point an integration at it.

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

**[Docs](https://api.fynex.ai/payments-api/v2/docs)** · **[fynex.ai](https://fynex.ai)** ·
**[Dashboard](https://dashboard.fynex.ai)** · **[Get in touch](https://fynex.ai)**

</div>
