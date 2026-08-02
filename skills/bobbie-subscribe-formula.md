---
name: Put Bobbie formula on a subscription selling plan
description: >-
  Resolve a product's selling plans and add a variant to the cart on a recurring plan
  rather than as a one-time purchase.
api: graphql/bobbie-storefront.graphql
operations: [product, productByHandle, sellingPlanGroups, sellingPlanAllocations, cartCreate, cartLinesAdd, cartLinesUpdate, update_cart]
generated: '2026-08-02'
method: generated
---

# Put Bobbie formula on a subscription selling plan

Bobbie's core business is a recurring formula subscription. In the contract that shows up
as Shopify **selling plans**, and getting it wrong is the single most common failure mode
on this store.

## Steps

1. **Discover the plans on the product.**
   `product(id:)` → `sellingPlanGroups(first: 10) { nodes { appName name options { name values } sellingPlans(first: 10) { nodes { id name description recurringDeliveries options { name value } } } } }`
2. **Get the allocation on the exact variant.** Plans price per variant:
   `variants(first: 25) { nodes { id title sellingPlanAllocations(first: 10) { nodes { sellingPlan { id name } priceAdjustments { price { amount currencyCode } compareAtPrice { amount currencyCode } } } } } }`
   The `sellingPlan.id` from the allocation is what you pass to the cart — not the group id.
3. **Add it to the cart on that plan.**
   `cartLinesAdd(cartId:, lines: [{ merchandiseId: <variantId>, quantity: 1, sellingPlanId: <sellingPlanId> }])`
   or, on the MCP surface, `update_cart` with the equivalent add-item payload.
4. **Switch an existing line** between one-time and subscription with `cartLinesUpdate`,
   setting or clearing `sellingPlanId` on the line.
5. Read back `cart { lines { nodes { id quantity sellingPlanAllocation { sellingPlan { name } } cost { totalAmount { amount currencyCode } } } } }`
   and confirm the recurring price with the buyer before handing off `checkoutUrl`.

## Rules

- **`Product.requiresSellingPlan: true` means one-time purchase is impossible.** Adding
  that variant with no `sellingPlanId` fails with `VARIANT_REQUIRES_SELLING_PLAN`.
- A `sellingPlanId` that does not belong to that variant's allocations fails with
  `SELLING_PLAN_NOT_APPLICABLE`. Always take the id from the variant's
  `sellingPlanAllocations`, never from another variant or from memory.
- Subscription price ≠ list price. Read `priceAdjustments` — quoting the one-time price
  for a subscription line misleads the buyer.
- **A subscription is a recurring financial commitment.** Confirm cadence and price
  explicitly with the buyer, and never complete the purchase yourself — see
  `bobbie-build-cart-and-checkout.md` and `agentic-access/bobbie-agentic-access.yml`.
- Managing an existing subscription (skip, pause, cancel) is a **customer-account**
  operation. It requires an OIDC-delegated session with
  `customer-account-api:full` — it is not on the anonymous storefront surface. See
  `scopes/bobbie-scopes.yml`.
