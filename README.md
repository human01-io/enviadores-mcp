<p align="center">
  <img src="https://api.enviadores.com.mx/api/v1/assets/mcp-icon-256.png" width="96" alt="Enviadores" />
</p>

# Enviadores MCP Server

**The first shipping MCP server in Mexico.** Quote, create, track and cancel shipments with Estafeta, DHL, FedEx, Paquetexpress and more — plus pickups, an address book, webhooks and balance — from Claude, ChatGPT, Cursor or any MCP client.

**Español:** el primer servidor MCP de paquetería en México. Cotiza, crea, rastrea y cancela envíos multi-paquetería, solicita recolecciones, administra tu libreta de direcciones y recibe webhooks desde cualquier cliente MCP.

- **Endpoint:** `https://api.enviadores.com.mx/api/v1/mcp` (Streamable HTTP, remote — nothing to install)
- **Registry:** [`mx.com.enviadores/shipping`](https://registry.modelcontextprotocol.io/v0.1/servers?search=mx.com.enviadores) on the official MCP Registry
- **Docs:** https://enviadores.com.mx/desarrolladores · **Landing:** https://enviadores.com.mx/mcp

## Authentication

Two options:

1. **OAuth 2.1** — clients that speak MCP OAuth (claude.ai connectors, ChatGPT developer mode) discover it automatically: RFC 8414/9728 metadata, dynamic client registration (RFC 7591) and CIMD. Every grant mints a scoped key tied to your Enviadores account.
2. **API key** — mint an `ek_` key in the [dashboard](https://app.enviadores.com.mx) and send it as a bearer token. `ek_test_` keys run in a full sandbox with no real money.

## Connect

**Claude Code**

```bash
claude mcp add --transport http enviadores https://api.enviadores.com.mx/api/v1/mcp \
  --header "Authorization: Bearer ek_..."
```

**claude.ai** — Settings → Connectors → *Add custom connector* → `https://api.enviadores.com.mx/api/v1/mcp` (OAuth flow, no key needed).

**Cursor / JSON config**

```json
{
  "mcpServers": {
    "enviadores": {
      "url": "https://api.enviadores.com.mx/api/v1/mcp",
      "headers": { "Authorization": "Bearer ek_..." }
    }
  }
}
```

## Tools (23)

Every tool is a thin projection of one public REST endpoint. Quotes come back **grouped by service** (carrier + service level, shown once with its price range and bookable options); a `rate_id` is valid for 30 minutes.

**Quote and ship**

| Tool | What it does |
|---|---|
| `validate_postal_code` | SEPOMEX lookup (state, municipality, colonias) — optional pre-flight |
| `get_shipping_rates` | Live multi-carrier rates between two Mexican postal codes; one box (`package`) or 2–10 boxes as one shipment (`packages`); optional `insurance.insured_value` |
| `create_shipment` | Buy a label from a quoted `rate_id` — idempotent (`idempotency_key`), same boxes as quoted, optional `declared_value` |
| `get_shipment` | One shipment with its current status |
| `list_shipments` | Shipments on the account, filterable by status, tracking number and date |
| `get_label` | Label PDF link for a shipment |
| `track_shipment` | Live tracking by tracking number (guía) |
| `cancel_shipment` | Request cancellation (refund follows the carrier's outcome) |

**Pickups** (request-only: our team confirms with the carrier and emails the result)

| Tool | What it does |
|---|---|
| `schedule_pickup` | Request a carrier pickup at the sender's door for a local time window |
| `get_pickup` / `list_pickups` | Status of pickup requests |

**Address book**

| Tool | What it does |
|---|---|
| `list_senders` / `create_sender` | Saved senders |
| `list_recipients` / `create_recipient` | Saved recipients |
| `import_contacts` | Bulk import (rows or raw CSV) with a dry-run mode |

**Webhooks** (no polling: `shipment.*` and `pickup.resolved` events, HMAC-signed, retried with backoff)

| Tool | What it does |
|---|---|
| `register_webhook` | Register an `https://` endpoint for a set of events (secret shown once) |
| `list_webhooks` / `delete_webhook` | Manage the account's webhooks (5 active per mode) |
| `test_webhook` | Deliver a signed `ping` synchronously |

**Account**

| Tool | What it does |
|---|---|
| `whoami` | Account, key scopes, mode (sandbox/live) and capabilities of this connection |
| `get_balance` | Prepaid balance, holds and available amount |
| `list_transactions` | Balance movements (charges, refunds, top-ups) |

Every tool ships `title`, `annotations` (`readOnlyHint` / `destructiveHint` / `openWorldHint`) and an `outputSchema` derived from the [OpenAPI spec](https://api.enviadores.com.mx/api/v1/openapi.json), so structured results are typed. Pickup tools appear only when the feature is open on the account.

## Sandbox

`ek_test_` keys (or choosing *Sandbox* in the OAuth consent screen) run every tool against a sandbox with **no real money**: quotes list only services with a real sandbox behind them (a FedEx test lane plus one simulated offer), labels are durable signed test PDFs, and webhooks receive `shipment.created` / `shipment.cancelled` plus `ping`. Reconnect choosing *Producción* to ship for real.

## Spend safety

Everything charges a **prepaid balance** with server-side enforcement: `create_shipment` can never spend more than the available balance nor exceed the key's daily limit. `create_shipment` accepts an `idempotency_key` (one is generated and returned if omitted) so retries never double-charge. The MCP adapter is a pure protocol translator over the [public REST API](https://enviadores.com.mx/desarrolladores) — an agent can never do anything a plain REST caller couldn't.

## Support

- Docs: https://enviadores.com.mx/desarrolladores
- Email: contacto@enviadores.com.mx

---

This repository hosts the public documentation for the hosted server; the server itself is operated by [Enviadores](https://enviadores.com.mx).
