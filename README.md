<p align="center">
  <img src="https://api.enviadores.com.mx/api/v1/assets/mcp-icon-256.png" width="96" alt="Enviadores" />
</p>

# Enviadores MCP Server

**The first shipping MCP server in Mexico.** Quote, create, track and cancel shipments with Estafeta, DHL, FedEx, Paquetexpress and more — from Claude, ChatGPT, Cursor or any MCP client.

**Español:** el primer servidor MCP de paquetería en México. Cotiza, crea, rastrea y cancela envíos multi-paquetería desde cualquier cliente MCP.

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

## Tools (12)

| Tool | What it does |
|---|---|
| `get_shipping_rates` | Live multi-carrier rates for a package between two Mexican postal codes |
| `create_shipment` | Buy a label from a quoted `rate_id` (idempotent — safe to retry) |
| `get_label` | Fetch the label PDF for a shipment |
| `track_shipment` | Live tracking by shipment id |
| `list_shipments` | List shipments on the account |
| `cancel_shipment` | Request cancellation |
| `get_balance` | Prepaid balance and daily spend limit |
| `validate_postal_code` | SEPOMEX postal-code lookup (city, state, colonias) |
| `list_senders` / `create_sender` | Address book: saved senders |
| `list_recipients` / `create_recipient` | Address book: saved recipients |

## Spend safety

Everything charges a **prepaid balance** with server-side enforcement: `create_shipment` can never spend more than the available balance nor exceed the key's daily limit. `create_shipment` accepts an `idempotency_key` (one is generated and returned if omitted) so retries never double-charge. The MCP adapter is a pure protocol translator over the [public REST API](https://enviadores.com.mx/desarrolladores) — an agent can never do anything a plain REST caller couldn't.

## Support

- Docs: https://enviadores.com.mx/desarrolladores
- Email: contacto@enviadores.com.mx

---

This repository hosts the public documentation for the hosted server; the server itself is operated by [Enviadores](https://enviadores.com.mx).
