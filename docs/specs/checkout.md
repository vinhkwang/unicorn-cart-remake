# Spec: checkout

Status: draft

## Goal

The server-side checkout state machine, offer (upsell/cross-sell) steps, the order form, and the compliance slots that render legally required confirmation and change-comparison content.

## Requirements

- **State machine:** stored in a `checkout_session` identified by an HTTP-only cookie token. States: `ENTRY → CONFIRM (skippable by setting) → OFFERS_BEFORE_PAYMENT → PAYMENT → ORDERED → OFFERS_AFTER_ORDER → COMPLETED`. Side states: `PAYMENT_PENDING` (redirect or async methods), `FAILED`, `EXPIRED`.
- **Order creation timing:** the order is created at a defined point (default: after payment authorization or COD confirmation), not at first form submit.
- **Idempotency:** submit endpoints require an idempotency key so double-clicks never create duplicate orders.
- **Offer steps:** each upsell step shows one offer. Accept adds items to the same session. Decline moves on. Steps run only when enabled in settings.
- **Server-side totals:** totals are recalculated server-side at every step. The browser never supplies prices.
- **Validation:** entry data is validated server-side with the same rules the browser uses.
- **Offer placement:** offers can appear on the confirm step before payment and on the thanks step after the order exists, both inside the same flow.
- **Add mode:** accepted items are added to the order, or to the first delivery of a subscription.
- **Replace mode:** accepted items replace the original order. The original order is canceled automatically and a new order or subscription takes its place (e.g. switching a one-time purchase to a subscription).
- **No second payment entry:** post-order offers reuse the order's stored payment credential, and the original order stays valid if the shopper leaves the page.
- **Disclosure:** an offer that changes an existing order shows a before-and-after comparison (items, prices, delivery cycle, total payable) and needs an explicit accept action. The shopper then receives an updated confirmation email.
- **Combinations:** the trigger and the offer can each be one-time or subscription, giving four combinations, each with defined behavior: add to the contract, add to the first delivery, replace by cancel and switch, or add to the order.
- **Form quality:** the order form slot includes inline validation, input assistance, and field-level error messages that keep entered values.
- **Compliance slots:** the order form step must include `{{slot:final_confirmation}}`, rendering the legally required items from the offer data; offer steps that change an existing order include `{{slot:change_comparison}}`. These cannot be hidden or restyled below minimum visibility (see `japan-compliance`).
- **Checkout endpoints:** `POST /checkout/entry`, `POST /checkout/confirm`, `POST /checkout/upsell/{step}`, `POST /checkout/complete`, `GET /thanks/{orderToken}`.

## Out of scope

- The definition of upsell/cross-sell rules attached to offers (owned by `catalog-and-offers`); this spec executes them.
- Payment processing itself (owned by `payments`).

## Acceptance criteria

- AC-1: The state machine transitions follow `ENTRY → CONFIRM (skippable) → OFFERS_BEFORE_PAYMENT → PAYMENT → ORDERED → OFFERS_AFTER_ORDER → COMPLETED`, with `PAYMENT_PENDING`, `FAILED`, and `EXPIRED` side states.
- AC-2: The order is created at the defined point, not at first form submit; each submit endpoint requires an idempotency key.
- AC-3: Totals are recalculated server-side at every step; any client-supplied price is ignored.
- AC-4: Entry data is validated server-side with the same rules the browser uses.
- AC-5: Each upsell step shows exactly one offer; accept adds to the same session and decline moves on; steps only run when enabled in settings.
- AC-6: An order-changing offer shows a before-and-after comparison (items, prices, delivery cycle, total payable) and requires explicit accept; the shopper receives an updated confirmation email.
- AC-7: Post-order offers reuse the stored payment credential, and the original order remains valid if the shopper leaves the page.
- AC-8: The four one-time/subscription trigger×offer combinations each behave as defined.
- AC-9: The order form includes `{{slot:final_confirmation}}`, and order-changing offer steps include `{{slot:change_comparison}}`; neither can be hidden or restyled below minimum visibility.
- AC-10: The order form has inline validation, input assistance, and field-level errors that preserve entered values.

## Dependencies

- `urls-and-templates`: slot tokens and flow templates.
- `catalog-and-offers`: offers and upsell/cross-sell rules.
- `payments`: the payment step and stored credentials.
- `orders`: order creation and the legal snapshot.
- `settings-engine`: confirmation-skip and upsell enablement settings.
- `japan-compliance`: final confirmation content, delivery date/time slot selection, and abandoned-entry consent.
- `subscriptions`: subscription creation when the order contains a subscription offer.
- Data model: `checkout_session`, `idempotency_key`.

## Open questions

- **Confirm skip and legal display** (source 11.10): assumed that when the confirm page is skipped, the order form carries the required final confirmation items; the client's legal team must confirm.
- **Offer placement and replace mode** (source 11.9): assumed offers on both the confirm and thanks steps with add and replace modes, based on the original's automatic cancellation after a successful upsell. Confirm exact behavior.
- **Order creation point:** default is after payment authorization or COD confirmation; confirm if it should vary by method.
