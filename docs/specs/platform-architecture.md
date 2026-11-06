# Spec: platform-architecture

Status: draft

## Goal

Define the platform-wide architecture, layering, frontend strategy, security chains, API surface, and non-functional requirements that every feature spec must follow. The original product's internals (tech stack, schema, deployment) are not public, so everything here is a proposed design for this remake, not a replication.

## Requirements

- **Deployable shape:** a modular monolith — one Spring Boot deployable split by feature packages with strict internal boundaries — plus three Angular applications, one of which (cart-app) is rendered by a Node SSR server. A monolith fits a solo developer, keeps order/payment transactions simple, and can be split later along module lines.
- **Backend:** Java 21, Spring Boot 3.x, Spring Web, Spring Security, Spring Data JPA with Hibernate 6, Flyway, MapStruct, Bean Validation, Micrometer.
- **Frontend:** Angular with standalone components and signals, lazy-loaded feature routes, reactive forms, an OpenAPI-generated API client, and one workspace containing the admin app, the client app, and the cart app plus shared libraries for the generated API client, auth, and the schema-driven settings form renderer. UI components are shared only between the admin and client apps.
- **Cart renderer:** a Node process running Angular SSR for cart-app. It holds no business data and calls Spring Boot for everything.
- **Database and services:** PostgreSQL 16+; Redis for sessions and caching (optional at MVP); an S3-compatible object store for images and exports; an SMTP or API mail provider.
- **Edge:** a reverse proxy (Caddy or Nginx) doing host-based routing and TLS for merchant custom domains. It sends API paths to Spring Boot and cart URLs to the Node renderer.
- **Layering (strict Clean Architecture):** Controller → Service → Repository, no layer skipping. Controllers are HTTP-only: accept request DTOs, validate, call exactly one service method, return response DTOs. Services hold business rules and transaction boundaries and never touch HTTP types. Repositories are interfaces owned by the domain with JPA implementations behind them. DTOs are immutable records at every boundary; MapStruct converts; entities never leave the repository and service layers. Constructor injection only. Collaborators that vary (payment gateways, tax rounding, mail providers) are injected as interfaces. ArchUnit tests enforce all of this.
- **Design patterns:** Strategy (payment gateways, tax rounding, shipping fee calculation, order number generation, mail providers, upsell eligibility rules); Repository (all persistence); DTO (all API and SSR view-model boundaries); Dependency Injection (including strategy selection via a registry keyed by tenant settings); State machine (checkout session, order status, subscription status); Factory/Registry (setting definitions, template slot components and variables).
- **Frontend specifics:** admin settings screens are generated from setting definitions served by the backend, not hand-built per setting. The template editor (in admin-app) has code editors for HTML/CSS/JS per region and a live preview that loads the real cart renderer in an iframe. Cookie-based session auth with Angular's built-in XSRF handling. UI library for admin and client apps: Angular Material or PrimeNG (data-table-heavy back office favors PrimeNG); cart-app uses none.
- **Security chains:** three separate Spring Security filter chains — `/api/admin/**` (operator and merchant staff, role-based, strongest controls, optional second factor for operators), `/api/store/**` (shoppers; public reads for catalog and articles; authenticated routes for account, cart and orders; CSRF protected; login throttled), and cart pages plus `/checkout/**` (anonymous or logged-in consumers, CSRF protected, rate limited, bot-protected).
- **API surface:** REST with OpenAPI as the contract; Angular clients are generated from it. `/api/admin/v1/**` for the admin side (operator and merchant staff), `/api/store/v1/**` for the storefront (catalog, articles, shopper auth, cart, orders), each with its own DTOs. Cart pages are served by the Node renderer on the cart URL path; checkout calls go to Spring Boot (`POST /checkout/entry`, `POST /checkout/confirm`, `POST /checkout/upsell/{step}`, `POST /checkout/complete`, `GET /thanks/{orderToken}`). The renderer reads its render model from `GET /internal/render/v1/page` (reachable only from the renderer), and payment providers post events to `POST /webhooks/payments/{provider}`. On a shared hostname, storefront and cart pages are separated by path prefix. Pagination, filtering and sorting follow one shared convention; errors use a single problem-details format.
- **Data model conventions:** `bigint` identity keys, `tenant_id` on tenant-owned tables, `timestamptz` everywhere, money as `bigint` yen (no decimals), JSONB for flexible configuration, status columns as `varchar` with check constraints, Flyway-managed migrations, soft delete only where business rules need it.
- **Non-functional requirements:** cart pages cached and light with a bundle budget targeting about 120 KB gzipped initial JS and a fast first paint on mobile; stateless application nodes with sessions in Redis or the database, health checks, and zero-downtime deploys; no card data stored; secrets outside the repo; dependency scanning; structured logs with tenant and request identifiers, Micrometer metrics, and tracing on checkout and payment calls; daily PostgreSQL backups with tested restore and point-in-time recovery for production; Docker images for the Spring Boot service and Node renderer, Docker Compose for local development, and CI running build and tests on every push.
- **Testing strategy:** unit tests for services; Testcontainers PostgreSQL integration tests for repositories and RLS; contract tests for payment adapters; ArchUnit architecture tests; Playwright end-to-end tests covering the checkout.
- **Repository layout (assumed):** Spring Boot backend at the repo root (`pom.xml`, `mvnw`, `src/`), Angular workspace under `frontend/`, Playwright end-to-end tests under `e2e/`. The frontend workspace must define `lint` and `typecheck` npm scripts (Phase 0 scaffold).
- **Delivery phasing (section 10):** Phase 0 foundations (repo layout, backend module skeleton, Flyway, Testcontainers, ArchUnit, Angular workspace with three apps and shared libs, Docker Compose, CI); Phase 1 tenancy/auth/settings; Phase 2 catalog and offers; Phase 3 storefront; Phase 4 URLs/templates and Angular SSR cart pages; Phase 5 checkout/orders/payments; Phase 6 upsell and cross-sell; Phase 7 subscriptions and mail; Phase 8 customers and reports; Phase 9 candidates. Recommended MVP is Phases 0–5.

## Out of scope

- Ad management and ad analysis, including AD codes per channel/agency and real-time CVR.
- Chatbot-style entry form (PGS) and exit-prevention popups (SGS).
- Phase 9 candidates (to be reviewed after MVP): coupons, trial offers, step mail, CRM extraction of subscribers, analytics on continuation, upselling from a trial product to a subscription.
- Replication of the original product's internal tech stack, schema, or deployment, which are not public.

## Acceptance criteria

- AC-1: ArchUnit tests fail the build if a controller imports a repository, a service imports a controller, or a module reaches into another module's internals.
- AC-2: No controller contains business logic or direct repository access; each calls exactly one service method and maps errors to problem-details responses.
- AC-3: Three Spring Security filter chains exist for `/api/admin/**`, `/api/store/**`, and cart/checkout paths, each enforcing the controls listed in Requirements.
- AC-4: The cart-app initial JavaScript bundle is at or under the ~120 KB gzipped budget and lazy-loads slot components.
- AC-5: The internal render API is reachable only from the renderer, never from the public internet.
- AC-6: The Angular workspace builds all three apps; the backend is a single deployable with feature-package modules.
- AC-7: The OpenAPI contract generates the Angular API client, and both API prefixes (`/api/admin/v1/**`, `/api/store/v1/**`) have separate DTOs.
- AC-8: Every persisted table follows the data model conventions (bigint identity, `tenant_id` on tenant-owned tables, `timestamptz`, yen `bigint`, JSONB, check-constrained status, Flyway migrations).
- AC-9: Testcontainers integration tests, ArchUnit tests, and Playwright e2e tests run in CI on every push.

## Dependencies

- Referenced by every feature spec. Depends on no other spec; it is foundational.
- External system components: PostgreSQL 16+, Redis (optional MVP), an S3-compatible object store, an SMTP/API mail provider, and a reverse proxy (Caddy or Nginx).
- Multi-tenancy mechanics are specified in `tenancy-and-access`; cart SSR mechanics in `urls-and-templates`; compliance in `japan-compliance`.

## Open questions

- **Repo layout:** whether the Spring Boot backend lives at the repo root (assumed here, so `./mvnw` works from root) or under a `backend/` directory. Confirm before Phase 0.
- **UI library:** PrimeNG (favored for the data-table-heavy back office) vs Angular Material.
- **Hosting** (source 11.13): undecided; affects TLS automation for custom domains and managed Redis/object-store choices.
- **Angular SSR fit** (source 11.12): templates are raw HTML/CSS/JS strings, so Angular is a host for server-rendered strings plus slot components. The Phase 4 performance gate decides whether to keep Angular SSR or fall back to a JVM-side renderer using the same render API.
- **Scope of the original work** (source 11.14): which modules were personally built determines where to match experience and where to simplify.
