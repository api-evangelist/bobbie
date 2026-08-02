---
name: Answer a shopper's Bobbie policy question
description: >-
  Answer returns, shipping, subscription and contact questions from Bobbie's own
  published policies rather than from model memory.
api: mcp/bobbie-mcp.yml
graphql: graphql/bobbie-storefront.graphql
operations: [search_shop_policies_and_faqs, shop, page, pageByHandle, article]
generated: '2026-08-02'
method: generated
---

# Answer a shopper's Bobbie policy question

## Steps

1. **Ask the store, not the model.** Call `search_shop_policies_and_faqs` on
   `https://www.hibobbie.com/api/mcp` with the shopper's question as `query` and any
   buyer `context`. It answers questions like "what is your return policy", "what is your
   shipping policy", "what is your phone number", "what are your hours".
2. **Or read the policy documents directly** over GraphQL:
   `shop { refundPolicy { title body url } shippingPolicy { title body url } privacyPolicy { title body url } termsOfService { title body url } subscriptionPolicy { title body url } contactInformation { title body url } }`
3. **Fall back to store pages** for anything not modelled as a policy:
   `pageByHandle(handle: "help-center")`, `pages(first: 20)`, or `articles` on the
   editorial blog.
4. Cite the canonical human URL in your answer:
   - Refund: `https://www.hibobbie.com/policies/refund-policy`
   - Shipping: `https://www.hibobbie.com/policies/shipping-policy`
   - Terms: `https://www.hibobbie.com/policies/terms-of-service`
   - Privacy: `https://www.hibobbie.com/policies/privacy-policy`
   - Help centre: `https://www.hibobbie.com/pages/help-center`

## Rules

- **Never paraphrase a policy from memory.** Refund windows, subscription cancellation
  terms and shipping cut-offs change; quote what the live response says and link it.
- If the tool returns nothing relevant, say so and point at the help centre or
  `https://www.hibobbie.com/pages/contact` — do not invent a policy.
- `search_shop_policies_and_faqs` is a server-side retrieval composite, not a field
  selection: it may summarise. When the exact wording matters (a refund dispute, a legal
  term), read the `ShopPolicy.body` over GraphQL instead.
- Infant feeding, allergy, and formula-transition questions are **not** policy questions.
  Route them to `medical.hibobbie.com` and to the parent's pediatrician.
