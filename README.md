# Bobbie

Bobbie is an American organic infant formula company founded in 2018 by Laura Modi and Sarah Hardy — the only mom-founded, women-led infant formula manufacturer in the United States, selling direct to parents at [hibobbie.com](https://www.hibobbie.com/) and through [Bobbie Medical](https://medical.hibobbie.com/) for healthcare professionals.

Bobbie runs no developer program and publishes no OpenAPI. It does, however, serve a real machine-readable surface from its own domain — and it publishes an explicit agent contract at [`/agents.md`](https://www.hibobbie.com/agents.md), which is rare for a DTC brand.

## Surfaces

| Surface | Endpoint | Auth |
|---|---|---|
| Storefront GraphQL API | `https://www.hibobbie.com/api/2026-04/graphql.json` | none — introspection is anonymous |
| Storefront MCP server | `https://www.hibobbie.com/api/mcp` | none — 5 tools with full input schemas |
| UCP agentic commerce (MCP) | `https://www.hibobbie.com/api/ucp/mcp` | requires a UCP agent profile URI |
| UCP merchant profile | `https://www.hibobbie.com/.well-known/ucp` | none |
| Customer accounts (OIDC / RFC 8414) | `https://account.hibobbie.com/authentication/oauth/*` | authorization code + PKCE |
| Agent instructions | `/agents.md`, `/llms.txt` | none |

**Bobbie requires human approval on payment.** From its own `/agents.md`: *"Checkout requires human approval. Agents must not complete payment without explicit buyer consent."*

## Artifacts

- `graphql/` — the introspected Storefront SDL (416 types, 35 queries, 41 mutations) + manifest
- `mcp/` — MCP server manifest, verbatim `tools/list` response, and the tool ↔ GraphQL crosswalk
- `agentic-access/` — Bobbie's own published agent rules and flow
- `llms/` — verbatim `llms.txt` and `agents.md`
- `well-known/` — UCP profile, OIDC and OAuth authorization-server metadata
- `authentication/`, `scopes/` — the three auth postures and the customer-account scopes
- `conventions/`, `errors/`, `lifecycle/`, `conformance/`, `data-model/` — derived semantics
- `skills/` — four Agent Skills grounded in real tool names and real GraphQL fields
- `security/` — probed TLS/HSTS/DNS posture

Not found (probed 2026-08-02, all 404): OpenAPI, `/.well-known/agent-card.json`, `/.well-known/agent.json`, `/.well-known/security.txt`, `/.well-known/api-catalog`, `/.well-known/ai-plugin.json`. No status page, no bug bounty, no trust center, no first-party SDK or CLI.

- Secondary market: https://forgeglobal.com/bobbie_stock/
