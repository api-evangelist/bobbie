---
name: Find and compare Bobbie infant formula
description: >-
  Search Bobbie's catalog for the right infant formula and resolve a specific purchasable
  variant, using either the anonymous Storefront MCP server or the Storefront GraphQL API.
api: mcp/bobbie-mcp.yml
graphql: graphql/bobbie-storefront.graphql
operations: [search_catalog, get_product_details, search, products, product, productByHandle]
generated: '2026-08-02'
method: generated
---

# Find and compare Bobbie infant formula

Bobbie sells a small catalog (organic infant formula, gentle formula, whole milk formula,
grass-fed whole milk formula, plus accessories) with multiple sizes and a subscription
option per product. Your job is to get from a parent's intent to one concrete, in-stock
**variant id**.

## Which surface to use

Both are anonymous — no key, no token.

| Need | Surface |
|---|---|
| Natural-language intent ("gentle formula for a gassy newborn") | MCP `search_catalog` at `https://www.hibobbie.com/api/mcp` |
| Exact, structured queries and full field control | GraphQL at `https://www.hibobbie.com/api/2026-04/graphql.json` |

## Steps — MCP path

1. Call `search_catalog`. Supply a natural-language `query`, structured `filters`, or
   both — the tool requires at least one. Pass buyer context (`address_country`,
   `currency`) so pricing and availability are correct; Bobbie asks for this explicitly
   in `/agents.md`.
2. Results are paginated with a deliberately small first page. Page forward rather than
   asking for a huge `first`.
3. Call `get_product_details` with the `product_id` you picked. Pass `options` to select
   a specific variant (size, one-time vs subscription); without `options` you get the
   first available variant, which may not be the one the parent wants. `country` and
   `language` shape the price and copy.

## Steps — GraphQL path

1. `products(first:, query:, sortKey:)` or `search(query:, types: [PRODUCT], productFilters:)`
   to list candidates. Select `id handle title productType tags availableForSale
   priceRange { minVariantPrice { amount currencyCode } } requiresSellingPlan`.
2. `product(id:)` or `productByHandle(handle:)` for the detail. Select
   `variants(first: 25) { nodes { id title sku availableForSale quantityAvailable price { amount currencyCode } selectedOptions { name value } } }`.
3. Use `variantBySelectedOptions(selectedOptions: [...])` when you already know the option
   values — it resolves straight to one variant id.
4. Wrap the query in `@inContext(country: US, language: EN)` to localize.

## Rules

- **Check `availableForSale` and `quantityAvailable` before you promise anything.** Infant
  formula goes out of stock; a variant that lists is not a variant that ships.
- **`requiresSellingPlan: true` means the product cannot be bought one-time.** Route to
  the subscription skill (`bobbie-subscribe-formula.md`) instead of adding it as a plain
  line item, or the cart mutation fails with `VARIANT_REQUIRES_SELLING_PLAN`.
- Never quote a price you did not read from the response — prices vary by country and by
  selling plan.
- Bobbie is an infant nutrition brand. Do not give feeding, medical, or dosing advice.
  Route clinical questions to `medical.hibobbie.com` and the parent's pediatrician.

## Errors

GraphQL returns HTTP 200 with an `errors[]` array; check it before reading `data`. Every
response carries `extensions.cost` — the API is query-cost throttled, not header
rate-limited. The MCP endpoint is rate-limited per IP; back off on 429.
See `errors/bobbie-problem-types.yml`.
