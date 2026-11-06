# Spec: catalog-and-offers

Status: draft

## Goal

Products, variants, SKUs, offers with purchase modes, stock policy, and the upsell/cross-sell rules that attach to an offer.

## Requirements

- **Products and variants:** a product has variants with SKU, price in yen, tax category, and stock.
- **Offers:** an offer is what the page actually sells — items and quantities, one-time or subscription pricing, a shipping rule, and an optional discount.
- **Purchase mode per offer:** a single product, or multiple products. Multiple products can be a fixed bundle or items the shopper chooses with quantities. SKU selection is either a fixed SKU, a variant the shopper picks on the page, or a bundle of several SKUs. Quantity limits apply per SKU and per order.
- **Stock policy per variant:** unlimited, tracked, or tracked with oversell block.
- **Upsell and cross-sell rules:** attach to an offer as ordered steps with a target offer, an eligibility condition, and accept and decline behavior. They run only when the enabling settings are on: a tenant-level switch, a per-offer switch, and a maximum step count. When off, the checkout skips those steps.
- **Strategies:** shipping fee calculation and tax rounding are strategy interfaces selected per tenant.

## Out of scope

- The execution of upsell/cross-sell steps inside checkout (owned by `checkout`); this spec defines the rules attached to offers.
- Ad management and analytics.

## Acceptance criteria

- AC-1: A product has variants, each with an SKU, price in yen, tax category, and stock.
- AC-2: An offer defines items and quantities with one-time or subscription pricing, a shipping rule, and optional discount.
- AC-3: Purchase mode is single product or multiple products, where multiple products is either a fixed bundle or shopper-chosen quantities; the three SKU selection modes (fixed SKU, shopper-picks-variant, bundle of SKUs) are also supported.
- AC-4: Quantity limits are enforced per SKU and per order at checkout.
- AC-5: Each stock policy (unlimited, tracked, tracked with oversell block) is honored when adding to cart and placing an order.
- AC-6: Upsell/cross-sell steps are skipped unless the tenant-level switch, the per-offer switch, and the maximum step count all allow them.
- AC-7: Shipping fee calculation and tax rounding are resolved through strategy interfaces selected per tenant settings.

## Dependencies

- `settings-engine`: purchase mode, pricing, and upsell/cross-sell enablement settings.
- `tenancy-and-access`: per-tenant catalog scoping.
- `checkout`: consumes offers and upsell steps.
- `payments`: per-method fees feed pricing rules.
- `japan-compliance`: tax-inclusive display and rounding.
- Data model: `product`, `product_variant`, `offer`, `offer_item`, `upsell_step`, `shipping_rule`.

## Open questions

- **Purchase modes** (source 11.7): assumed single product, or multiple products as a fixed bundle or shopper-chosen items, with on-page SKU selection. Confirm what "multiple products" meant in the original.
