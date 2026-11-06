# Spec: orders

Status: draft

## Goal

Order lifecycle, order detail, order operations, CSV export, per-tenant order number generation, and the legal snapshot that records what the shopper saw.

## Requirements

- **Statuses:** placed, payment pending, paid, preparing, shipped, delivered, canceled, refunded.
- **Order detail:** items, customer, addresses, payments, shipments, the cart URL and template version, and an event timeline.
- **Operations:** edit shipping info before shipment, cancel, partial refund, resend mail, add internal notes.
- **CSV export:** with configurable column sets, including shipping-company formats.
- **Order number generation:** per-tenant, through a strategy backed by a locked counter row.
- **Legal snapshot:** each order stores a snapshot of what the shopper saw (rendered HTML, terms, template version, capture time) to anticipate the pending before-and-after disclosure and contract-detail requirements.
- **Constraints:** unique `(tenant_id, order_no)`; an index on `(tenant_id, placed_at)` for reporting.

## Out of scope

- Payment processing (owned by `payments`); this spec records payments and refunds.
- Subscription generation (owned by `subscriptions`).

## Acceptance criteria

- AC-1: An order transitions through statuses from placed to paid/preparing/shipped/delivered, or to canceled/refunded, and each transition is recorded in an event timeline.
- AC-2: Order detail shows items, customer, addresses, payments, shipments, the cart URL and template version, and the event timeline.
- AC-3: Shipping info can be edited before shipment; cancel and partial refund are supported; internal notes and mail resend are available.
- AC-4: CSV export supports configurable column sets, including shipping-company formats.
- AC-5: Order numbers are unique per (tenant, order_no) and generated through a strategy backed by a locked counter row.
- AC-6: Each order captures a legal snapshot with rendered HTML, terms JSON, template version, and capture time.
- AC-7: An index on `(tenant_id, placed_at)` supports reporting queries.

## Dependencies

- `checkout`: order creation timing and the order link (replace mode).
- `payments`: payment records, status, cancel and refund actions.
- `catalog-and-offers`: items and pricing captured on the order.
- `tenancy-and-access`: per-tenant order numbers and export audit.
- `settings-engine`: order number format, initial status, stock deduction timing, cancellation rules.
- `mail`: order received/shipped mails.
- `japan-compliance`: the legal snapshot that records what the shopper saw.
- Data model: `orders`, `order_item`, `payment`, `shipment`, `order_event`, `order_link`, `order_legal_snapshot`, `export_job`.

## Open questions

- None beyond those already listed in `checkout` and `payments`.
