# Spec: mail

Status: draft

## Goal

Transactional mail, segment mailing, the mail template editor, and delivery handling (bounces, unsubscribes, suppression).

## Requirements

- **Transactional mails:** order received, payment failure, shipped, subscription reminder.
- **Segment mailing:** segment definitions stored as structured filters (purchase history, product, subscription status), evaluated into recipient lists.
- **Template editor:** with variables, test send, scheduled send, delivery log, bounce and unsubscribe handling, and a suppression list.
- **Sending:** goes through a queue and a `MailProvider` strategy.

## Out of scope

- Mail content and sender identity settings (owned by `settings-engine` mail group).
- The subscription scheduler that triggers reminders (owned by `subscriptions`).

## Acceptance criteria

- AC-1: The four transactional mails (order received, payment failure, shipped, subscription reminder) are sent on their corresponding events.
- AC-2: Segments are stored as structured filters and evaluate into recipient lists.
- AC-3: The mail template editor supports variables, test send, and scheduled send.
- AC-4: A delivery log records sends; bounces and unsubscribes are handled; the suppression list is enforced.
- AC-5: Sending goes through a queue and a `MailProvider` strategy.

## Dependencies

- `orders`: order received/shipped events.
- `payments`: payment failure events.
- `subscriptions`: reminder triggers.
- `customers-and-accounts`: segment filter engine reuse.
- `settings-engine`: sender identity, templates, trigger mails, signature, unsubscribe wording.
- `tenancy-and-access`: per-tenant mail scoping.
- Data model: `mail_template`, `mail_segment`, `mail_campaign`, `mail_delivery`, `mail_suppression`.

## Open questions

- None.
