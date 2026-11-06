# Spec: tenancy-and-access

Status: draft

## Goal

Multi-tenancy isolation, tenant and domain lifecycle, staff users and roles, operator impersonation, and the audit log. This spec owns the shared-database/RLS tenancy model from the architecture and the `/api/admin/**` security chain.

## Requirements

- **Multi-tenancy (decision):** shared database, shared schema, `tenant_id` on every tenant-owned table, enforced by two layers: (1) application layer — a Hibernate tenant discriminator resolved from a request-scoped tenant context; (2) database layer — PostgreSQL Row Level Security policies keyed on a session variable (`app.tenant_id`) set at the start of each transaction, protecting against a missed filter in a hand-written query.
- **Tenant resolution:** cart pages and storefront resolve the tenant by request hostname, looked up in `tenant_domain`; a shopper account belongs to exactly one tenant. Admin side resolves from the authenticated staff user. Operator users have no tenant and act across tenants through explicit, audited impersonation.
- **Tenant lifecycle:** created, active, suspended, closed.
- **Domain management:** add hostname, verify ownership by a DNS record, attach a purpose (cart pages, default).
- **Staff users:** roles (owner, manager, operator, viewer) with a permission matrix enforced in services.
- **Operator impersonation:** with an audit trail.
- **Audit log:** for every settings change, publish, refund, and export.
- **Admin security chain:** `/api/admin/**` is operator and merchant staff, role-based, with the strongest controls and an optional second factor for operators.
- Alternative considered (schema-per-tenant or database-per-tenant) is recorded but not adopted: revisit only if a client demands physical isolation.

## Out of scope

- Physical tenant isolation (schema-per-tenant or database-per-tenant) — explicitly rejected for MVP.
- Fee plans, transaction fee rates, feature flags and limits per tenant (these are the platform settings in `settings-engine`, operator-only group).

## Acceptance criteria

- AC-1: Every tenant-owned table has a `tenant_id` column and an RLS policy keyed on `current_setting('app.tenant_id')`; a query in one tenant's context cannot return another tenant's rows.
- AC-2: Cart-page and storefront requests resolve the tenant by hostname; a hostname not present in `tenant_domain` is rejected.
- AC-3: Admin requests resolve the tenant from the authenticated staff user; an operator with no tenant acts across tenants only through audited impersonation.
- AC-4: The staff role permission matrix is enforced in services, not only in the UI.
- AC-5: A hostname becomes active only after DNS ownership verification.
- AC-6: Every settings change, publish, refund, and export writes an `audit_log` entry with actor, action, target, and diff.
- AC-7: A tenant's lifecycle status gates access to its surfaces (a suspended or closed tenant is inaccessible).
- AC-8: The admin security chain offers an optional second factor for operators.

## Dependencies

- `platform-architecture`: RLS approach, security chains, `/api/admin/v1/**`.
- `settings-engine`: settings changes feed the audit log.
- `orders`: refunds and exports feed the audit log.
- `urls-and-templates`: publish events feed the audit log.
- Data model: `tenant`, `tenant_domain`, `staff_user`, `audit_log`.

## Open questions

- **Tenancy model** (source 11.4): shared schema with RLS is assumed. Per-client isolated deployments would change infrastructure and migration planning.
- **Custom domains** (source 11.6): assumed required from MVP.
- **Template JS edit permission** (source 11.11): whether merchant staff may edit template JS or only operator staff, since it runs in shoppers' browsers.
