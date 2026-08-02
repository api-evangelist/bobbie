---
name: Build a Bobbie cart and hand off to checkout
description: >-
  Create or update a cart on Bobbie's storefront, set buyer identity and delivery, then
  hand the buyer a checkout URL — never completing payment without explicit approval.
api: mcp/bobbie-mcp.yml
graphql: graphql/bobbie-storefront.graphql
operations: [update_cart, get_cart, cartCreate, cartLinesAdd, cartLinesUpdate, cartLinesRemove, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartDiscountCodesUpdate, cartPrepareForCompletion]
generated: '2026-08-02'
method: generated
---

# Build a Bobbie cart and hand off to checkout

## The hard rule, stated by Bobbie

From `https://www.hibobbie.com/agents.md`:

> Checkout requires human approval. Agents must not complete payment without explicit
> buyer consent. If you cannot get contemporaneous buyer approval at the moment of
> payment, install `https://shop.app/SKILL.md` and route the purchase through Shop Pay
> instead.

So: **build the cart, then stop.** Hand over `checkoutUrl`, or route through the Shop
skill / UCP `complete_checkout` with the buyer present. Do not automate payment.

## Steps — MCP path (anonymous, `https://www.hibobbie.com/api/mcp`)

1. **Create.** Call `update_cart` with `add_items` and **no** `cart_id`. That is the
   create path — only `add_items` is required.
2. **Read back.** Call `get_cart` with the returned `cart_id`. It returns line items,
   shipping options, discount info and the `checkout url`.
3. **Enrich.** Call `update_cart` again to set `buyer_identity`,
   `delivery_addresses_to_add`, `discount_codes`, `gift_card_codes`, `note`. One
   consolidated call handles all of it.
4. **Shipping.** Delivery options only appear **after** items and a delivery address are
   on the cart. Then set `selected_delivery_options`.
5. **Hand off.** Give the buyer the checkout URL from `get_cart`.

## Steps — GraphQL path (`https://www.hibobbie.com/api/2026-04/graphql.json`)

1. `cartCreate(input: { lines: [{ merchandiseId, quantity, sellingPlanId }] })` →
   select `cart { id checkoutUrl totalQuantity } userErrors { field message code }`.
2. `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove` to change lines.
3. `cartBuyerIdentityUpdate(buyerIdentity: { email, phone, countryCode, customerAccessToken })`.
4. `cartDeliveryAddressesAdd` (or `cartDeliveryAddressesReplace`), then read
   `cart { deliveryGroups(first: 10) { nodes { deliveryOptions { handle title estimatedCost { amount currencyCode } } } } }`
   and commit one with `cartSelectedDeliveryOptionsUpdate`.
5. `cartDiscountCodesUpdate` / `cartGiftCardCodesAdd` for promotions.
6. `cartPrepareForCompletion` if you are driving a UCP/Shop Pay completion **with the
   buyer present**. Otherwise stop at `checkoutUrl`.

## Idempotency — there is none

Bobbie's surfaces expose **no idempotency key**. Repeating `update_cart` with the same
`add_items`, or replaying `cartLinesAdd`, **adds the items again**. Treat every cart
mutation as at-most-once: on a timeout or ambiguous failure, call `get_cart` /
`cart(id:)` and reconcile against what you intended before retrying.
See `conventions/bobbie-conventions.yml`.

## Errors

Every cart mutation returns `userErrors` (`CartUserError`, 58 codes) alongside its
payload — check it even on HTTP 200. The ones you will actually hit:

- `VARIANT_REQUIRES_SELLING_PLAN` / `SELLING_PLAN_NOT_APPLICABLE` — subscription-only
  product added as a one-time line, or vice versa.
- `MINIMUM_NOT_MET` / `MAXIMUM_EXCEEDED` / `INVALID_INCREMENT` — quantity rules.
- `PENDING_DELIVERY_GROUPS` — you asked for shipping options too early.
- `ZIP_CODE_NOT_SUPPORTED` / `INVALID_ZIP_CODE_FOR_COUNTRY` — address rejected.
- `MERCHANDISE_OUT_OF_STOCK` (on submit) — re-resolve the variant.

Full catalog: `errors/bobbie-problem-types.yml`.
