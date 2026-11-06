# Spec: reports-and-dashboard

Status: draft

## Goal

The admin Dashboard (summaries of orders, revenue, subscriptions, and payment status) and configurable sales reports with exports.

## Requirements

- **Dashboard:** an admin-side dashboard showing summaries of orders, revenue, subscriptions, and payment status for the current tenant. Operators scope across tenants through audited impersonation.
- **Sales reports:** configurable sales reports with a selectable time range and column set, generated from order, payment, subscription, and customer data.
- **Exports:** report exports run as asynchronous jobs tracked in `export_job`, and each export writes an audit log entry.

## Out of scope

- Ad management and analytics (explicitly out of scope for the remake).
- Order and customer CSV exports themselves (owned by `orders` and `customers-and-accounts`); this spec covers sales reports and their exports.

## Acceptance criteria

- AC-1: The dashboard shows summaries of orders, revenue, subscriptions, and payment status, scoped to the tenant.
- AC-2: A sales report is configurable by time range and column set.
- AC-3: A configured report can be exported as CSV through an asynchronous `export_job` whose result is downloadable, and the export writes an audit log entry.

## Dependencies

- `orders`: order and revenue data for the dashboard and reports.
- `payments`: payment status summaries.
- `subscriptions`: subscription summaries and recurring revenue.
- `customers-and-accounts`: customer dimensions for reports.
- Data model: `export_job`; dashboard and reports read from `orders`, `payment`, `subscription`, and `customer`.

## Open questions

- The source (12.4) notes the original's exact admin screens and field lists are not public; the specific dashboard widgets and report definitions are design decisions pending the client's confirmation.
