# Spec: settings-engine

Status: draft

## Goal

A schema-driven settings engine for the admin side so the wide range of tenant settings is not hand-built as one screen and one table column per setting.

## Requirements

- **Definitions in code:** each setting has a key, type, default, validation rules, group, label key, visibility (operator-only or merchant-editable), and whether changes need audit.
- **Values in the database:** `setting_value` rows per tenant storing JSON, with version and updated-by.
- **UI generated from definitions:** the Angular settings renderer builds forms from the definition metadata.
- **Typed access:** services read settings through typed DTOs, never raw keys.
- **Setting groups:**
  - Business profile: company name, legal notice (特定商取引法) content, contact details, invoice registration number.
  - Domains and tags: hostnames, analytics tags, conversion tags.
  - Checkout form: fields shown, required or optional, validation rules, furigana, delivery date/time slots, confirmation page skip (when skipped, the order form carries the legally required final confirmation items, which the system enforces).
  - Purchase mode: single or multiple products per offer, SKU selection mode (fixed SKU, shopper picks a variant, bundle), quantity limits per SKU and per order, minimum and maximum order amounts.
  - Upsell and cross-sell: tenant-level master switch, per-offer enablement, placement (confirm step, thanks step), mode (add to order, or replace and cancel the original order), maximum number of steps, trigger conditions, offer and template per step, behavior on decline.
  - Pricing: tax display, tax rounding rule, shipping fee rules, COD fee, free-shipping threshold.
  - Payment: enabled methods, provider credentials, 3-D Secure behavior, methods shown by default, display order, per-method conditions (amount range, one-time or subscription offers), per-method fees.
  - Orders: order number format, initial status, stock deduction timing, cancellation rules.
  - Subscriptions: allowed cycles (weeks, months, weekday of a month), delivery counts and minimum counts, first and later prices, skip/pause/cancel rules, retry policy for failed charges.
  - Mail: sender identity, templates, trigger mails, signature, unsubscribe wording.
  - Customers: required profile fields, duplicate matching rules, tags, retention and anonymization rules, shopper account options.
  - Cart templates: script-edit permission, allowed script hosts, default regions, redirect behavior when a path changes.
  - Users and security: roles, password policy, session length, IP allowlist.
  - Platform (operator only): fee plan, transaction fee rate, feature flags per tenant, limits.

## Out of scope

- The individual feature behaviors those settings control (checkout, payments, subscriptions, etc.) — this spec only defines the engine and the setting catalog.

## Acceptance criteria

- AC-1: Each setting definition carries key, type, default, validation rules, group, label key, visibility, and audit flag, all defined in code.
- AC-2: The settings renderer builds forms from definitions; no setting screen is hand-built per setting.
- AC-3: Services read settings through typed DTOs; raw key access from business code is not permitted.
- AC-4: `setting_value` stores per-tenant JSON values with a version and updated-by.
- AC-5: A change to an audit-flagged setting writes an audit entry.
- AC-6: Operator-only settings are hidden and uneditable for merchant staff.
- AC-7: Validation rules on a definition are enforced when a value is saved.

## Dependencies

- `tenancy-and-access`: per-tenant scoping, audit log integration.
- `platform-architecture`: the schema-driven settings form renderer shared library.
- `japan-compliance`: tax display and rounding, furigana, delivery date/time slots, and confirmation-skip settings.
- Data model: `setting_value`.

## Open questions

- None beyond those already listed under the individual groups; confirm the exact setting catalog with the client's back-office needs as the admin screens are unknown from the original.
