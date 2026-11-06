# Spec: subscriptions

Status: draft

## Goal

Recurring (定期) subscriptions: course settings, the order-generation scheduler, status history, and staff/shopper management actions.

## Requirements

- **Creation:** a subscription is created when an order contains a subscription offer.
- **Course settings per offer:** delivery cycle (every N weeks, every N months, or a weekday of a month), number of deliveries (unlimited or a fixed minimum count), first-delivery price and later price, and an optional final-delivery price.
- **Scheduler:** generates the next order on `next_order_date`, charges the stored payment token, and logs every run. Uses a distributed lock so it is safe to run more than one instance.
- **Status history:** transitions (active, paused, skipped, canceled, payment failed) are recorded as events with actor and reason.
- **Staff actions:** skip, pause, resume, change cycle, change next delivery date, change items, change payment method, cancel.
- **Shopper self-service:** on My Page the same actions are available but limited by the offer's rules (for example a minimum delivery count that blocks cancellation until met). The cancel path must be clear and never obstructed.
- **Failed charges:** the retry policy comes from settings. Final failure notifies the customer and flags the subscription.

## Out of scope

- Course pricing math itself (defined by `catalog-and-offers` offer pricing); this spec runs the schedule against it.
- Mail composition (owned by `mail`); this spec triggers the subscription reminder.

## Acceptance criteria

- AC-1: A subscription is created automatically when an order contains a subscription offer.
- AC-2: Course settings define the delivery cycle, delivery count, first/later/optional-final price.
- AC-3: The scheduler generates the next order on `next_order_date`, charges the stored token, and logs each run; it uses a distributed lock.
- AC-4: Status transitions are recorded as events with actor and reason.
- AC-5: Staff can skip, pause, resume, change cycle, change next delivery date, change items, change payment method, and cancel.
- AC-6: Shoppers can perform the same actions on My Page, limited by the offer's rules; a minimum delivery count blocks cancellation until met, and the cancel path is always findable.
- AC-7: Failed charges retry per settings; final failure notifies the customer and flags the subscription.

## Dependencies

- `orders`: recurring order generation.
- `payments`: stored payment token and retry.
- `catalog-and-offers`: course settings per offer.
- `settings-engine`: subscription settings (cycles, counts, skip/pause/cancel rules, retry policy).
- `mail`: subscription reminder and failure mails.
- `customers-and-accounts`: My Page surface.
- `tenancy-and-access`: per-tenant scoping.
- Data model: `subscription`, `subscription_event`, `subscription_run`.

## Open questions

- None beyond the payment provider's stored-credential support (source 11.5).
