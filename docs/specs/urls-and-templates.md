# Spec: urls-and-templates

Status: draft

## Goal

Cart URLs, templates and versioning, the template editor, and the Angular SSR cart renderer (cart pages). This spec owns the renderer mechanics and the template-as-data security model from the architecture.

## Requirements

- **Cart URL:** a hostname and path bound to an offer and a flow. Statuses: draft, published, archived. Unique per tenant, hostname and path.
- **Flow binding:** each URL names a template for every step: entry, confirm (optional), each upsell and cross-sell step, and thanks. The thanks step can also host post-order offers.
- **Template:** a named set of regions. Default regions are header, body and footer; a template can add more named regions (for example extra head tags). Each region stores HTML, CSS and JS.
- **Slots and variables:** regions embed platform components through slot tokens — `{{slot:order_form}}`, `{{slot:upsell_offer}}`, `{{slot:cart_summary}}`, `{{slot:order_summary}}`, plus the compliance slots `{{slot:final_confirmation}}` and `{{slot:change_comparison}}` — and a fixed set of variables, both documented in the editor. Variables such as `{{product.name}}` and `{{offer.price}}` come from a fixed allowlist and are substituted on the Spring Boot side by plain text replacement; there are no loops, conditions or expressions.
- **Versioning:** every save creates a template version. One version is published. Rollback is a republish of an older version. Publishing updates every URL that uses the template.
- **Template library:** platform starter templates owned by the operator, which merchants can duplicate, and tenant templates.
- **Editor:** code editors for HTML, CSS and JS per region, a live preview through the real renderer, warnings for unknown slots or variables, and a diff between versions.
- **URL management:** list, search, filter by status, duplicate, change template, publish and unpublish, and redirects when a path changes.
- **Permissions:** editing template JS needs a separate permission from editing HTML and CSS.
- **Request flow (renderer):** the proxy sends cart URLs to the Node renderer. The renderer reads the Host header and path, calls the internal render API (`GET /internal/render/v1/page`) on Spring Boot, and receives a render model (resolved template regions, offer, checkout step, settings). It then renders the page with hydration.
- **Templates as data:** regions are stored as HTML, CSS and JS strings. They are never compiled as Angular templates and never evaluated on the server. The renderer inserts the HTML as markup, adds the CSS to the page head, and runs the JS in the browser after load.
- **Slots (rendering):** the renderer splits region HTML at slot tokens and renders platform-owned Angular components in their place, so form logic, validation and totals stay trusted and server-driven.
- **Hydration:** only the slot components hydrate. Template HTML regions are skipped during hydration, and event replay covers clicks that happen before hydration finishes.
- **Performance:** the cart app uses no UI component library, loads slot components lazily, and has a bundle budget of about 120 KB gzipped initial JS. Rendered output is cached by tenant, path and template version and invalidated on publish. Each template region has a size limit and a content hash, the hash serving as the render cache key.
- **Security:** template JS is the trust boundary. Editing JS needs a dedicated permission; every save is versioned and audited; each merchant has its own hostname so scripts cannot touch another tenant's cookies. Without the JS permission, HTML goes through an allowlist sanitizer that strips scripts. A per-tenant CSP lists approved script hosts. Because the server never evaluates template content, there is no server-side template injection.
- **CSRF:** checkout calls go to Spring Boot on the same hostname using the XSRF cookie and header. Rendered HTML never contains a token, so it stays cacheable.
- **Preview:** unpublished versions render only with a short-lived signed token.
- **Domains:** each merchant maps one or more hostnames. The proxy terminates TLS and forwards the original host header.
- **Fallback:** templates are plain strings and slots are tokens, so a JVM-side renderer could replace the Node renderer using the same render API (the escape hatch if the Phase 4 performance gate fails).
- **Compliance slots:** the order form step must include `{{slot:final_confirmation}}`, which renders the legally required items from the offer data, and offer steps that change an existing order include `{{slot:change_comparison}}`. Template authors cannot hide these or restyle them below a minimum visibility, and a template without them cannot be published.

## Out of scope

- The checkout state machine and order form logic behind the slots (owned by `checkout`).
- Ad management and analytics.

## Acceptance criteria

- AC-1: A cart URL is unique per (tenant, hostname, path) and bound to an offer and a flow, with statuses draft/published/archived.
- AC-2: Each flow step names a template; publishing a template version updates every URL that uses it; rollback republishes an older version.
- AC-3: Template regions store HTML, CSS and JS as strings and are never evaluated on the server.
- AC-4: Slot tokens render platform-owned components; variables substitute only from the fixed allowlist with no loops, conditions or expressions.
- AC-5: A template missing `{{slot:final_confirmation}}` (or `{{slot:change_comparison}}` where required) cannot be published, and the compliance slots cannot be hidden or restyled below minimum visibility.
- AC-6: HTML saved without the JS permission is sanitized to strip scripts; editing JS requires a dedicated permission; every save is versioned and audited.
- AC-7: A per-tenant CSP is applied to cart pages, and merchant hostnames isolate cookies.
- AC-8: Unpublished versions render only with a short-lived signed preview token.
- AC-9: Rendered output is cached by (tenant, path, template version) and invalidated on publish; each region has a size limit and a content hash used as the cache key.
- AC-10: The internal render API (`GET /internal/render/v1/page`) returns the render model and is reachable only from the renderer.
- AC-11: A path change on a URL creates a redirect.
- AC-12: The cart-app initial JS bundle is at or under ~120 KB gzipped and lazy-loads slot components.
- AC-13: The template library exposes operator-owned starter templates that a merchant can duplicate into tenant templates.
- AC-14: The template editor provides a live preview through the real renderer, warns on unknown slots or variables, and shows a diff between versions.
- AC-15: URL management supports list, search, and filter by status, and a URL can be duplicated or have its template changed.
- AC-16: Checkout calls use the XSRF cookie and header on the same hostname, and rendered HTML never contains a CSRF token.
- AC-17: Only slot components hydrate; template HTML regions are skipped during hydration, and clicks that occur before hydration finishes are replayed.

## Dependencies

- `tenancy-and-access`: hostname resolution, publish audit, permissions.
- `catalog-and-offers`: offers referenced by cart URLs and flow steps.
- `checkout`: slot components (order form, upsell, summaries) and flow steps.
- `japan-compliance`: required links and compliance slots on cart pages.
- `platform-architecture`: renderer, proxy routing, bundle budget.
- Data model: `cart_url`, `url_redirect`, `cart_template`, `cart_template_version`, `cart_template_region`.

## Open questions

- **Angular SSR fit** (source 11.12): the Phase 4 performance gate decides whether to keep Angular SSR or fall back to a JVM-side renderer using the same render API.
- **Template authoring rights** (source 11.11): confirm who may edit JS (operator only, or merchant staff too).
- **Custom domains** (source 11.6): assumed required from MVP with storefront and cart pages on the same hostname; otherwise Phase 4 simplifies to path or subdomain routing.
