# Spec: payments

Status: draft

## Goal

Payment methods, the PaymentGateway strategy, webhooks, 3-D Secure, payment management screens, method display rules, and payment records.

## Requirements

- **Strategy:** a `PaymentGateway` strategy interface with provider adapters selected per tenant.
- **MVP methods:** cash on delivery (代金引換) and one card provider using hosted fields or redirect so card numbers never touch this system.
- **Later methods:** convenience store payment, bank transfer, deferred payment (後払い), wallet methods.
- **Webhooks:** a `POST /webhooks/payments/{provider}` endpoint per provider with signature verification and idempotent handling.
- **3-D Secure:** Japanese card payment requires EMV 3-D Secure support (required in principle since end of March 2025). The provider runs the flow; subscription renewals depend on the provider's stored-credential support.
- **Payment management (admin):** a screen per tenant listing every payment method with its provider connection status, an enable switch, display order, and display conditions.
- **Method display:** the tenant setting gives the default list, and a cart URL can override which enabled methods appear and in what order. At checkout, each method's strategy also hides it automatically when it cannot serve the order (e.g. amount outside the method's allowed range, or a subscription offer when the method cannot store a token).
- **Payment records:** a per-order payment list and detail with status, provider reference, cancel and refund actions, and a log of received webhook events.
- **Fees:** per-method fees (such as the COD fee) feed the pricing rules.

## Out of scope

- Fee calculation rules themselves (owned by `settings-engine` and `catalog-and-offers`); this spec supplies the fees.

## Acceptance criteria

- AC-1: `PaymentGateway` is a strategy interface; COD and one card provider are supported at MVP, and card numbers never touch this system (hosted fields or redirect).
- AC-2: Webhook endpoints verify provider signatures and handle events idempotently.
- AC-3: Card payments support EMV 3-D Secure; subscriptions renew using the provider's stored credentials.
- AC-4: The admin payment management screen lists each method with connection status, enable switch, display order, and conditions.
- AC-5: A cart URL can override which enabled methods appear and their order; a method that cannot serve the order (amount range, subscription without token) is hidden automatically at checkout.
- AC-6: Payment records show status, provider reference, cancel/refund actions, and a webhook event log.
- AC-7: Per-method fees feed the pricing rules.

## Dependencies

- `tenancy-and-access`: per-tenant provider credentials.
- `settings-engine`: payment settings (enabled methods, 3-D Secure behavior, method conditions, fees).
- `orders`: payment records, cancel and refund actions.
- `checkout`: the payment step and stored credentials for post-order offers.
- `subscriptions`: stored-token reuse and failed-charge retry.
- `japan-compliance`: 3-D Secure requirement.
- Data model: `payment_method_config`, `cart_url_payment_method`, `payment_webhook_event`, and `payment` (orders area).

## Open questions

- **Payment provider** (source 11.5): not chosen. The decision affects the 3-D Secure flow, stored credentials for subscription renewals, and COD support.
- **Payment display rules** (source 11.8): assumed a tenant default list with a per-URL override plus automatic hiding by amount range and subscription eligibility. Confirm whether the original also varied by offer or by shopper.
