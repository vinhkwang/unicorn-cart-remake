# Spec: customers-and-accounts

Status: draft

## Goal

Customer records, admin customer management, shopper account management, privacy (deletion/anonymization), and personal-data protection.

## Requirements

- **Customer creation:** records are created from orders, matched by normalized email and phone.
- **Profile:** names with furigana, addresses, order history, subscriptions, notes, tags.
- **Admin customer screens:** list with search and filters (name, email, phone, tags, status, last order date); a detail page with orders, subscriptions and payments; internal notes; tags; status (active or blocked); duplicate detection and merge; CSV export.
- **Shopper account management:** a separate screen for storefront accounts showing status (active or suspended), the linked customer, last login, and actions to send a password reset mail, force logout, and delete or anonymize on request.
- **Segments:** reuse the filter engine from mail.
- **Data deletion and anonymization:** a flow for privacy requests.
- **Personal data protection:** viewing, editing and exporting personal data is permission-gated, and edits and exports are audited. Personal data columns (name, address, phone, email) are considered for column-level encryption or at minimum encrypted storage and restricted export.

## Out of scope

- Shopper-facing account self-service flows (registration, login, password reset) — owned by `storefront`; this spec covers the admin-side management of those accounts.

## Acceptance criteria

- AC-1: A customer record is created from an order and matched to an existing record by normalized email and phone.
- AC-2: The admin customer list supports search and filters (name, email, phone, tags, status, last order date); the detail page shows orders, subscriptions, payments, notes, tags, and status.
- AC-3: Duplicate customers are detected and can be merged.
- AC-4: Shopper account management shows status, linked customer, and last login, and supports password-reset mail, force logout, and delete/anonymize on request.
- AC-5: Viewing, editing, and exporting personal data is permission-gated, and edits and exports are audited.
- AC-6: A privacy-request deletion/anonymization flow exists and is traceable.
- AC-7: Customer CSV export is supported.
- AC-8: Personal data columns are stored encrypted (or column-level encrypted) with restricted export.

## Dependencies

- `orders`: customer creation and order history.
- `subscriptions`: subscriptions on the detail page.
- `payments`: payments on the detail page.
- `mail`: segment filter engine reuse.
- `tenancy-and-access`: permission gating and audit.
- `japan-compliance`: furigana, normalization, privacy expectations.
- `storefront`: the shopper accounts this spec manages.
- Data model: `customer`, `customer_address`, `customer_note`, `customer_tag`.

## Open questions

- None beyond the general scope question (source 11.14).
