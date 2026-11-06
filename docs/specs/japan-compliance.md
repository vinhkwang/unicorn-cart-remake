# Spec: japan-compliance

Status: draft

## Goal

Capture the Japan-specific legal and localization requirements (source section 8) that cut across features: name and address input, tax display, the specified commercial transactions law (特定商取引法) final confirmation screen, 3-D Secure, and information security expectations. These notes summarize public information and are not legal advice; a lawyer must confirm them before launch.

## Requirements

- **Names:** name fields carry furigana, auto-suggested from the kanji input where feasible.
- **Address input:** postal code to address lookup; full-width and half-width normalization for phone, postal code, and email.
- **Delivery:** delivery date and time slot selection.
- **Tax:** tax-inclusive price display, per-rate tax breakdown (10% and reduced 8%), configurable rounding, and invoice-ready receipts.
- **Required links:** a specified commercial transactions law (特定商取引法) page and a privacy policy link are required on every cart page.
- **Market payment methods:** COD, convenience store, deferred payment, and card with 3-D Secure.
- **Consent:** consent handling when capturing abandoned entry data.
- **Final confirmation screen (特定商取引法 Article 12-6):** since June 2022 the screen where the shopper presses the order button must clearly show quantity, price, payment timing and method, delivery timing, application period and withdrawal/cancellation terms, without misleading display. For subscriptions this includes each delivery's quantity, the prices from the second delivery on, and the billing and shipping dates. Skipping the confirm page is safe only if the order form screen itself carries these items beside the order button — enforced by the compliance slot in `checkout`.
- **Pending changes:** a Consumer Affairs Agency study group interim report (2 September 2026) proposes showing the total payment amount on the final confirmation screen, banning cancellation terms placed away from the main display, requiring before-and-after disclosure when an order is changed right after it is placed, and requiring contract details to be sent by email. These are not yet law; the offer comparison slot, the update email, and the legal snapshot in `orders` anticipate them.
- **Misleading design (dark patterns):** the same report proposes broader rules that may later cover claims such as countdowns, stock and review displays in templates. A template editor warning for these is a later candidate.
- **3-D Secure:** EMV 3-D Secure has been required in principle for all EC merchants since the end of March 2025. The payment provider normally runs the flow; subscription renewals depend on the provider's stored-credential support.
- **Information security:** clients may ask for ISMS and PrivacyMark-style controls.

## Out of scope

- Legal advice: these notes summarize public information; a lawyer must confirm them before launch.
- Implementation of any not-yet-law pending changes as hard requirements (they are anticipated, not enforced).

## Acceptance criteria

- AC-1: Every cart page renders both the 特定商取引法 page link and the privacy policy link, and neither can be removed by a template author.
- AC-2: Prices are displayed tax-inclusive with a per-rate (10%/8%) breakdown and a configurable rounding rule.
- AC-3: Phone, postal code, and email inputs normalize full-width characters to half-width before validation and storage.
- AC-4: Postal code input can resolve an address.
- AC-5: The final confirmation step (or the order form when confirm is skipped) shows quantity, price, payment timing and method, delivery timing, application period, and withdrawal/cancellation terms; for subscriptions it also shows each delivery's quantity, second-and-later prices, and billing/shipping dates.
- AC-6: Card payments support EMV 3-D Secure via the provider.
- AC-7: Name fields include furigana with auto-suggestion from kanji where feasible.
- AC-8: The checkout order form supports delivery date and time slot selection.
- AC-9: Capturing abandoned entry data is gated by consent handling.

## Dependencies

- Referenced by `checkout` (final confirmation and comparison slots), `payments` (3-D Secure), `catalog-and-offers` and `settings-engine` (tax display and rounding), `customers-and-accounts` (furigana, normalization, privacy), `urls-and-templates` (required links on cart pages), `storefront` (furigana names, normalization, tax-inclusive display), and `orders` (legal snapshot).

## Open questions

- **Confirm-skip legal display** (source 11.10): whether the final confirmation items on the order form satisfy Article 12-6 when the confirm page is skipped needs the client's legal team to confirm.
- **Pending law changes:** the total-amount display, cancellation-term placement, after-order change disclosure, and email contract delivery are not yet law; watch the next ordinary Diet session and adjust before launch.
- **Dark-pattern rules:** broader rules may later restrict countdown/stock/review displays in templates; a template editor warning is a later candidate.
