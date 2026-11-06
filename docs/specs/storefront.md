# Spec: storefront

Status: draft

## Goal

The shopper-facing storefront (client side). Shoppers are not staff. Covers catalog browsing, articles, shopper accounts, the cart, order history, and the hand-off to the SSR checkout.

## Requirements

- **Catalog browsing:** product list with category filter, search, sort and pagination. Product detail with variants, tax-inclusive price, stock state, and the linked offer.
- **Articles:** merchant-authored content (news, guides, product stories) with categories, scheduled publishing, SEO fields, and optional links to products. Authored in the admin side.
- **Article authoring (admin side):** articles are authored in the admin side with categories, scheduled publishing, SEO fields, and optional product links.
- **Shopper accounts:** register, log in, log out, reset password, edit profile and addresses. The session cookie is scoped to the merchant's hostname.
- **Cart:** items and quantities with prices recalculated on the server. Stored per account for logged-in shoppers and in a cookie for guests, merged on login.
- **Order history:** logged-in shoppers see past orders and statuses, and can use the subscription actions from `subscriptions`.
- **Checkout hand-off:** the cart sends the shopper to the SSR checkout template with a signed cart token, and the checkout state machine takes over. Landing pages enter the same state machine directly.
- **Access mode:** whether browsing requires login is a tenant setting, public by default.
- **Indexability:** public catalog and article pages must be indexable, so the storefront uses Angular SSR or prerendering for those routes.

## Out of scope

- Ad management and analytics, PGS chatbot entry form, and SGS exit popups.

## Acceptance criteria

- AC-1: An unauthenticated shopper can browse the product list and detail and read articles when the tenant's access mode is public.
- AC-2: Product detail shows variants, the tax-inclusive price, stock state, and the linked offer.
- AC-3: A shopper can register, log in, log out, reset a password, and edit profile and addresses; the session cookie is scoped to the merchant's hostname.
- AC-4: Cart prices are recalculated server-side; a guest cart merges into the account cart on login.
- AC-5: A logged-in shopper sees order history and can perform the subscription actions allowed by the offer's rules.
- AC-6: The storefront cart hands off to the SSR checkout with a signed cart token; landing pages enter the same state machine directly.
- AC-7: Public catalog and article routes are server-rendered or prerendered so they are indexable.
- AC-8: When the tenant sets browsing to require login, catalog and article routes require authentication.
- AC-9: Staff can author an article in the admin side with categories, SEO fields, optional product links, and a scheduled publication time; the storefront shows it only once published and not before its scheduled time.

## Dependencies

- `tenancy-and-access`: shopper auth, tenant resolution by hostname.
- `catalog-and-offers`: products, variants, offers for catalog and cart.
- `checkout`: the hand-off token and shared state machine.
- `subscriptions`: shopper self-service actions surfaced on order history.
- `settings-engine`: access mode and any storefront settings.
- `platform-architecture`: storefront SSR/prerendering and the `/api/store/v1/**` surface.
- `japan-compliance`: furigana names, address/phone/email normalization, and tax-inclusive price display.
- Data model: `article`, `article_category`, `article_product_link`, `shopper_account`, `shopper_cart`, `shopper_cart_item`, `password_reset_token`.

## Open questions

- **Cart hand-off** (source 11.1): assumed the storefront cart hands the shopper to the SSR checkout template and landing pages enter the same checkout directly. Confirm or correct.
- **Login requirement** (source 11.2): assumed browsing is public and login is needed for account, saved cart, and order history. A members-only shop changes storefront routes and the SEO plan.
- **Articles** (source 11.3): assumed merchant-authored content with optional product links. Confirm whether they also serve as SEO landing content.
- **Custom domains** (source 11.6): assumed required from MVP with storefront and cart pages on the same hostname.
