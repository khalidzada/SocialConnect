# SocialConnect — Marketplace & Commerce

## Software Requirements Specification

**Project:** SocialConnect Enterprise Platform
**Module:** Marketplace & Commerce
**Platform:** Social Network + Local/Global Commerce
**Technology Context:** .NET 8, ASP.NET Core MVC, EF Core, SQL Server, ASP.NET Core Identity, Razor, Bootstrap 5, REST APIs
**Architecture Context:** Unified Single-MVC Modular Monolith
**Status:** **CANONICAL DRAFT — READY FOR PRODUCT & ARCHITECTURAL REVIEW**
**Implementation Status:** Not Started

---

# 1. Purpose

This document defines the Software Requirements Specification (SRS) for the **Marketplace & Commerce subsystem of SocialConnect**.

The purpose of the subsystem is to establish a commerce platform that combines:

* social discovery,
* personal and community relationships,
* local commerce,
* product listings,
* professional shops,
* merchant operations,
* inventory and warehouse management,
* platform-mediated transactions,
* optional local/off-platform transactions,
* geographic discovery,
* localization,
* national and international commerce.

Marketplace & Commerce is therefore **not intended to be a standalone classified-listing system or a conventional e-commerce marketplace**.

Its defining product concept is:

> **Commerce exists inside the SocialConnect social graph rather than beside it.**

The Marketplace & Commerce subsystem must operate as a first-class part of the SocialConnect platform while preserving clear boundaries between social functionality, commerce functionality, payment functionality, moderation, notifications, media, and other platform capabilities.

---

# 2. Scope

The Marketplace & Commerce subsystem covers the complete commercial lifecycle from product discovery and listing through merchant operations, ordering, payment, fulfillment, delivery, settlement, and post-transaction lifecycle.

The scope includes:

1. Individual marketplace listings.
2. Shop-based product listings.
3. Product catalog management.
4. Shop management.
5. Inventory management.
6. Warehouse/location management.
7. Product availability.
8. Local marketplace discovery.
9. Regional/national/international discovery.
10. Social-feed product discovery.
11. Product-to-social-graph relationships.
12. Buyer and seller interactions.
13. Shopping/cart concepts where applicable.
14. Orders.
15. Payment configuration.
16. Platform-mediated protected transactions.
17. Local/external transaction arrangements.
18. Commission calculation.
19. Merchant settlement.
20. Delivery and pickup concepts.
21. Transaction lifecycle.
22. Commerce-related notifications through the canonical Notification platform.
23. Commerce-related events through the canonical Event platform.
24. Commerce-related reporting and moderation through the canonical platform.
25. Merchant subscription tiers.
26. Commerce governance and operational policies.

---

# 3. Product Vision

SocialConnect Marketplace is intended to connect **people, communities, places, shops and products** within one platform.

The product vision is based on the following relationship:

```text
Person
   ↓
Social Graph
   ↓
Community / Location / Interests
   ↓
Product Discovery
   ↓
Seller / Shop
   ↓
Inventory / Availability
   ↓
Order
   ↓
Payment
   ↓
Fulfillment
   ↓
Delivery / Pickup
   ↓
Settlement
```

The system must allow commerce to originate from multiple contexts:

* a marketplace search,
* a shop,
* a product page,
* a social post,
* a person's profile,
* a community/neighborhood context,
* a location-based discovery experience,
* recommendations within the social graph.

The commerce subsystem must therefore not assume that a product is discovered only through a conventional marketplace search.

---

# 4. Strategic Product Position

SocialConnect is not intended to become a direct clone of:

* Facebook Marketplace,
* Amazon,
* Zalando,
* Kleinanzeigen,
* Subito,
* Etsy,
* or another existing marketplace.

Existing platforms already provide substantial capabilities in their respective domains.

SocialConnect's intended distinction is the deliberate combination of:

> **Social Graph + Geographic Context + Product Graph + Shops + Merchant Operations + Flexible Transaction Models**

The platform should allow a user to move naturally between:

```text
Social interaction
      ↓
Discovery
      ↓
Product
      ↓
Seller / Shop
      ↓
Commerce
```

without requiring commerce to exist as an isolated experience.

---

# 5. Core Business Principles

The following principles are fundamental requirements.

## 5.1 Social-first commerce

Products and shops must be able to participate in the SocialConnect social environment.

Commerce must not be treated merely as an independent catalog database.

---

## 5.2 Local-first and globally capable

The platform must support both:

* nearby/local commerce,
* broader regional/national commerce,
* international commerce.

A product should not be permanently restricted to one geographic marketplace.

---

## 5.3 Individual-to-professional progression

A person must be able to start with minimal commercial commitment and progressively become a professional merchant.

The intended progression is:

```text
Individual
   ↓
Free Shop Showcase
   ↓
Professional Shop
   ↓
Subscription Plan
   ↓
Advanced Merchant Operations
```

---

## 5.4 Flexible transaction model

Not every transaction needs to be processed through SocialConnect.

The platform must support two fundamental commercial paths:

### Platform-mediated transaction

SocialConnect participates in the transaction and provides a protected payment/settlement flow.

### Local/external transaction

Buyer and seller arrange payment and fulfillment directly, particularly where they are geographically close and choose local exchange.

These models must coexist without forcing every transaction into the same commercial workflow.

---

# 6. Marketplace Participants

The system shall support the following participant categories.

## 6.1 Individual Seller

A normal SocialConnect user may offer products or goods without operating a professional shop.

Typical use cases include:

* selling used goods,
* selling personal items,
* local sales,
* occasional sales,
* community exchange.

---

## 6.2 Shop Owner

A user may establish a shop presence on SocialConnect.

The shop provides:

* merchant identity,
* product showcase,
* product catalog,
* availability,
* commercial information,
* shop location,
* contact/transaction options.

---

## 6.3 Professional Merchant

A merchant may use advanced commercial capabilities through a subscription plan.

Potential capabilities include:

* larger product catalog,
* advanced inventory,
* warehouse management,
* order management,
* merchant payment configuration,
* fulfillment configuration,
* analytics,
* commercial tools.

The exact subscription capabilities are to be defined by the commercial plan configuration.

---

## 6.4 Buyer

A buyer may:

* discover products,
* inspect product information,
* inspect sellers/shops,
* communicate with sellers,
* choose local or platform-mediated transaction options where available,
* place orders,
* pay through supported mechanisms,
* receive products,
* report issues,
* participate in post-transaction workflows.

---

# 7. Commerce Levels

SocialConnect shall provide a progressive commercial model.

## 7.1 Level 1 — Individual / Free

Every eligible SocialConnect user may participate in basic marketplace activity subject to platform policies.

The individual level is intended for:

* occasional sellers,
* personal goods,
* local transactions,
* community commerce.

The free level should minimize entry barriers.

---

## 7.2 Level 2 — Free Shop Showcase

SocialConnect shall provide a free shop showcase level.

The initial product requirement is:

> **A free shop may showcase up to five products.**

The free shop level allows small sellers to establish a persistent commercial presence without immediately requiring a paid subscription.

---

## 7.3 Level 3 — Subscription Commerce

SocialConnect shall provide one or more subscription-based merchant plans.

Subscription plans may progressively unlock capabilities such as:

* larger product catalogs,
* advanced inventory management,
* warehouse management,
* merchant operations,
* advanced analytics,
* additional shop capabilities,
* advanced fulfillment options,
* other professional commerce capabilities.

The exact number, pricing and capability matrix of subscription plans shall be defined separately by the commercial strategy.

---

# 8. Product Model

A product is a first-class commerce object.

A product must support commercial information such as:

* title,
* description,
* media,
* category,
* price,
* availability,
* seller,
* shop where applicable,
* geographic context,
* inventory,
* fulfillment options,
* transaction options,
* product status.

Product information must be capable of being presented through multiple discovery surfaces.

---

# 9. Product and Social Graph Integration

The social graph is a central differentiator of SocialConnect.

Products and shops should participate in social discovery without turning the social feed into a conventional advertising catalog.

The platform should support relationships such as:

```text
User
 ├── follows User
 ├── follows Shop
 ├── interacts with Post
 ├── discovers Product
 ├── views Shop
 └── purchases Product
```

A product may also be associated with social content:

```text
Social Post
    ↓
Product
    ↓
Shop
    ↓
Seller
```

This allows a merchant to introduce a product through social content while retaining the structured product representation required for commerce.

---

# 10. Social Feed Commerce

The SocialConnect feed must be capable of supporting commerce discovery.

Commerce-related content may appear through:

* merchant posts,
* product posts,
* shop activity,
* product recommendations,
* local commerce activity,
* community activity.

The feed must remain a social experience.

Commerce integration must not require every product to behave as an advertisement.

---

# 11. Geographic Commerce

Location is a shared platform capability across SocialConnect Social and Marketplace.

The commerce engine shall support geographic context at appropriate levels.

## 11.1 Geographic hierarchy

The platform should support concepts such as:

```text
Country
   ↓
Region / State / Province
   ↓
City
   ↓
District
   ↓
Neighborhood / Local Area
```

The exact geographic data model is a platform-level concern and must be shared rather than independently reinvented by Marketplace.

---

## 11.2 Local discovery

Users must be able to discover products according to geographic relevance.

Examples include:

* products near me,
* products in my city,
* products in my neighborhood,
* products available for local pickup,
* shops nearby,
* sellers nearby.

---

## 11.3 Radius-based discovery

Where supported by the platform's geographic capabilities, users should be able to search within a defined geographic radius.

Examples:

```text
5 km
10 km
25 km
50 km
```

The exact UI and supported ranges are product decisions.

---

## 11.4 Global discovery

Products must also be discoverable outside the user's immediate geographic area where the merchant provides appropriate fulfillment.

The platform must therefore distinguish:

```text
Geographic relevance
```

from:

```text
Geographic restriction
```

A local product may remain local.

A globally shippable product may be discoverable internationally.

---

# 12. Localization

Localization is a first-class requirement.

Marketplace and Social must share platform localization capabilities.

Commerce localization may include:

* language,
* currency,
* number formatting,
* date/time formatting,
* geographic naming,
* local measurement conventions,
* tax presentation,
* shipping information,
* payment availability,
* local commercial rules.

The platform must avoid assuming that one currency, language or commercial rule applies globally.

---

# 13. Internationalization and Globalization

SocialConnect must be designed for international expansion.

Global commerce may require:

* multiple currencies,
* multiple languages,
* country-specific payment providers,
* country-specific shipping,
* tax rules,
* legal requirements,
* product restrictions,
* seller requirements,
* consumer-protection requirements.

These requirements must be treated as configurable/replaceable capabilities where appropriate rather than hard-coded into the generic commerce domain.

---

# 14. Neighborhood Commerce

SocialConnect should support neighborhood-level commerce.

A neighborhood context may connect:

```text
People
+
Local Community
+
Local Shops
+
Products
+
Local Events
+
Local Services
```

This creates a potential local ecosystem in which users can discover not only products but also the people and businesses behind them.

Neighborhood functionality must remain compatible with the broader country, regional and international marketplace.

---

# 15. Shop

A Shop represents a persistent commercial presence.

A Shop may contain:

* shop identity,
* merchant ownership,
* branding,
* description,
* location,
* products,
* inventory,
* availability,
* fulfillment options,
* transaction configuration,
* subscription information.

A Shop must remain distinct from an individual user profile while maintaining a relationship to its owner/operator.

---

# 16. Product Catalog

Professional commerce requires structured product catalog management.

The catalog must support:

* product creation,
* product editing,
* product publication,
* product availability,
* product media,
* categorization,
* pricing,
* inventory relationship,
* product status,
* product removal/deactivation.

Catalog functionality must support progression from small free shops to larger professional merchants.

---

# 17. Inventory Management

Inventory management is a core professional commerce requirement.

A merchant must be able to understand:

```text
Product
   ↓
Stock
   ↓
Availability
   ↓
Orders
   ↓
Fulfillment
```

Inventory should support concepts including:

* available quantity,
* reserved quantity,
* sold quantity,
* inventory status,
* stock adjustments,
* inventory availability.

Inventory changes must remain consistent with order and fulfillment state.

---

# 18. Warehouse Management

Professional shops shall be capable of managing inventory across one or more merchant-controlled warehouse or stock locations.

Warehouse management is intended to mean **inventory/location management**, not that SocialConnect must physically operate warehouses.

A merchant may have:

```text
Shop
 ├── Warehouse A
 │     ├── Product A
 │     └── Product B
 │
 ├── Warehouse B
 │     ├── Product A
 │     └── Product C
 │
 └── Local Store
       ├── Product B
       └── Product C
```

Warehouse functionality may include:

* stock locations,
* inventory quantities,
* stock allocation,
* order fulfillment source,
* availability,
* stock movement,
* inventory reconciliation.

The detailed warehouse domain shall be defined separately before implementation.

---

# 19. Product Availability

Product availability must be capable of reflecting actual merchant inventory.

The platform should distinguish between:

* listed,
* available,
* temporarily unavailable,
* out of stock,
* discontinued,
* archived.

Where inventory management is enabled, availability should be derived consistently from inventory state rather than manually maintained independently in multiple locations.

---

# 20. Order Management

An order represents a commercial commitment between buyer and seller.

The order lifecycle must support appropriate states such as:

```text
Created
   ↓
Pending Payment
   ↓
Paid / Authorized
   ↓
Processing
   ↓
Fulfillment
   ↓
Shipped / Ready for Pickup
   ↓
Delivered / Collected
   ↓
Completed
```

Additional states will be required for:

* cancellation,
* refund,
* failed payment,
* dispute,
* return,
* partial fulfillment,
* other exceptional conditions.

The final state machine must be defined in the Commerce Architecture/Domain Contract.

---

# 21. Transaction Models

SocialConnect shall support two primary transaction models.

## 21.1 Platform-Mediated Transaction

This model allows SocialConnect to participate in the commercial transaction.

Conceptually:

```text
Buyer
  ↓
Order
  ↓
Payment
  ↓
Protected Transaction
  ↓
Merchant Fulfillment
  ↓
Delivery / Pickup
  ↓
Buyer Receives Product
  ↓
Settlement
  ↓
Merchant Receives Funds
```

The intended business model is that SocialConnect may retain its commission before merchant settlement.

The actual mechanism for authorization, holding, capture, release, refunds and settlement must use supported payment-provider capabilities and comply with applicable legal and regulatory requirements.

This SRS does **not** establish that SocialConnect itself will legally act as an escrow institution.

---

# 22. Commission Model

For platform-mediated transactions, SocialConnect may charge a transaction commission.

Conceptually:

```text
Gross Transaction
      ↓
Applicable Fees / Adjustments
      ↓
SocialConnect Commission
      ↓
Merchant Settlement
```

Commission rules must be configurable by appropriate commercial policy.

Potential factors may include:

* merchant plan,
* transaction type,
* category,
* geography,
* payment method,
* promotional policy.

The exact commission structure is a commercial decision and must not be hard-coded into domain constants.

---

# 23. Local / External Transaction

SocialConnect must support situations where nearby buyers and sellers arrange the transaction independently.

Examples include:

* local pickup,
* cash payment,
* bank transfer,
* mutually agreed external payment,
* direct local exchange.

In this model:

```text
Buyer
   ↕
Seller
   ↓
External / Local Payment
   ↓
Pickup / Direct Fulfillment
```

SocialConnect does not necessarily process the payment.

Where the transaction is explicitly completed outside SocialConnect's payment flow, the platform may provide the marketplace service without charging a transaction commission.

However:

> **Free SocialConnect transaction service does not imply that the buyer or seller has no external costs.**

Third-party payment, banking, delivery or other provider fees may still apply.

---

# 24. Transaction Choice

Where both transaction models are available, users should be able to understand the distinction.

The platform should clearly communicate:

* whether SocialConnect processes the payment,
* whether transaction protection is available,
* whether a commission applies,
* whether pickup is available,
* whether shipping is available,
* what happens in disputes,
* what happens if the transaction occurs externally.

The system must not represent an externally completed transaction as a SocialConnect-processed payment transaction.

---

# 25. Merchant Payment Setup

Professional merchants must be able to configure supported payment capabilities.

The payment architecture should support a provider-independent model.

Conceptually:

```text
Merchant
   ↓
Payment Configuration
   ↓
Supported Payment Provider
   ↓
Payment Capability
```

SocialConnect should not make the core commerce domain dependent upon a single payment provider.

Provider-specific behavior must be isolated behind the payment integration boundary.

Payment requirements may vary by:

* country,
* merchant,
* currency,
* payment method,
* regulatory environment.

---

# 26. Fulfillment

Commerce must distinguish payment from fulfillment.

Fulfillment may include:

* shipping,
* local delivery,
* merchant delivery,
* pickup,
* warehouse fulfillment,
* external fulfillment provider.

SocialConnect is not initially required to operate its own physical logistics network.

The system must instead support merchant-controlled and provider-integrated fulfillment models where appropriate.

---

# 27. Local Pickup

Local pickup is a first-class commerce scenario.

A product may be offered for:

* pickup at shop,
* pickup at merchant location,
* agreed local pickup point,
* other supported local arrangements.

Pickup may be especially relevant for:

* individual sellers,
* neighborhood commerce,
* used goods,
* local shops.

---

# 28. Shipping

Where a seller supports shipping, the product/order must be capable of representing shipping requirements.

Shipping may depend on:

* destination,
* product,
* merchant,
* warehouse,
* shipping provider,
* package characteristics,
* country.

Shipping capabilities must remain extensible.

---

# 29. Buyer Experience

A buyer should be able to:

1. Discover a product.
2. Understand what the product is.
3. Understand who is selling it.
4. Understand where it is located.
5. Understand availability.
6. Understand price.
7. Understand pickup/shipping options.
8. Understand transaction/payment options.
9. Purchase or arrange a local transaction.
10. Track the commercial lifecycle where applicable.
11. Receive the product.
12. Report problems where applicable.

The buyer experience must remain connected to the SocialConnect identity and social environment.

---

# 30. Seller Experience

A seller should be able to:

1. Create or manage products.
2. Publish products.
3. Manage pricing.
4. Manage inventory.
5. Manage shop information.
6. Manage warehouses/stock locations where enabled.
7. Configure fulfillment.
8. Configure supported payment capabilities.
9. Receive and process orders.
10. Track fulfillment.
11. View settlement information.
12. Manage subscription capabilities.
13. Participate in social product discovery.

---

# 31. Merchant Lifecycle

The intended merchant progression is:

```text
SocialConnect User
      ↓
Individual Seller
      ↓
Free Shop
      ↓
Growing Merchant
      ↓
Subscription Merchant
      ↓
Professional Commerce Operations
```

The platform should avoid forcing a new merchant to adopt professional infrastructure before it is necessary.

---

# 32. Product Discovery

Product discovery must operate across multiple contexts.

Possible discovery sources include:

* Marketplace search,
* category browsing,
* location search,
* neighborhood discovery,
* shop pages,
* social feed,
* social relationships,
* product recommendations,
* direct product links.

Discovery should consider relevant signals such as:

* geographic relevance,
* availability,
* product category,
* social context,
* merchant/shop relationship,
* user-selected preferences,
* supported delivery/pickup options.

The exact ranking/recommendation algorithms are outside this SRS.

---

# 33. Search

Marketplace search must support structured commercial discovery.

Search may include:

* keyword,
* category,
* price range,
* location,
* radius,
* availability,
* shop,
* seller,
* fulfillment method.

Search must support both local and broader discovery.

---

# 34. Social Trust Context

The SocialConnect identity and social graph may provide contextual trust signals.

Potential context includes:

* seller profile,
* shop identity,
* social presence,
* community relationship,
* transaction history,
* permitted ratings/reviews,
* moderation status.

The platform must distinguish:

```text
Social relationship
```

from:

```text
Commercial trust guarantee
```

A social connection must never automatically imply that a seller or transaction is safe or guaranteed.

---

# 35. Commerce Safety

Marketplace must integrate with the canonical platform safety architecture.

Commerce-related safety concerns include:

* fraudulent listings,
* prohibited products,
* misleading information,
* abusive sellers,
* payment disputes,
* counterfeit goods,
* unsafe products,
* transaction abuse,
* spam,
* manipulation.

Marketplace must use the canonical:

* Reporting,
* Moderation,
* Administration,
* Event,
* Notification

infrastructure rather than creating parallel systems.

---

# 36. Reporting and Moderation

Users must be able to report appropriate commerce targets.

Potential targets include:

* Product,
* Listing,
* Shop,
* Seller,
* Order,
* transaction-related content,
* other supported commerce entities.

Reporting must follow the platform-wide model:

```text
Report
   ↓
Moderation Review / Case
   ↓
Decision
   ↓
Action
```

Marketplace must not invent a separate moderation architecture.

---

# 37. Commerce Events

Commerce operations may generate canonical domain/integration events.

Examples include:

* ProductCreated,
* ProductPublished,
* ProductUpdated,
* ProductUnavailable,
* OrderCreated,
* PaymentAuthorized,
* PaymentFailed,
* PaymentCompleted,
* OrderCancelled,
* OrderShipped,
* OrderReadyForPickup,
* OrderDelivered,
* OrderCompleted,
* RefundInitiated,
* RefundCompleted,
* DisputeOpened,
* SettlementCompleted.

These names are **illustrative domain requirements**, not yet the final event contract.

Final event definitions must be established through the canonical Event & Notification architecture.

---

# 38. Commerce Notifications

Commerce notifications must use the canonical Notification platform.

Examples may include:

* new order,
* payment status,
* order status,
* shipment status,
* pickup readiness,
* delivery,
* cancellation,
* refund,
* dispute,
* settlement.

Marketplace must not create:

* MarketplaceNotificationService,
* MarketplaceNotificationPipeline,
* Marketplace-specific notification persistence,
* Marketplace-specific dispatcher.

Commerce generates events/facts; the canonical notification infrastructure determines notification handling.

---

# 39. Commerce and Media

Products and shops may use the canonical Media subsystem.

Marketplace must use the existing media architecture rather than implementing a separate upload system.

Product media should therefore follow the platform's established media lifecycle.

The commerce subsystem must not take ownership of MediaUploader internals.

---

# 40. Commerce and Geographic Platform

Geographic information must be a shared platform capability.

Marketplace must not create a separate country/city/location model merely for commerce when an appropriate platform-level geographic capability exists.

The geographic context must support:

* user location,
* shop location,
* product/listing location,
* warehouse location,
* pickup location,
* shipping destination.

---

# 41. Commerce and Identity

Commerce must integrate with SocialConnect Identity.

The system must distinguish:

```text
Identity
```

from:

```text
Business Profile
```

and:

```text
Shop / Merchant
```

Identity remains responsible for authentication/account identity.

Commerce owns commercial entities and rules.

---

# 42. Merchant Authorization

Merchant operations must be authorized according to ownership and platform authorization rules.

A user must not be able to:

* modify another merchant's products,
* modify another shop's inventory,
* access another merchant's orders,
* modify another merchant's payment configuration,
* access another merchant's settlement information.

Administrative users must also follow the platform's domain ownership rule.

Administration must not bypass commerce services and directly manipulate commerce tables.

---

# 43. Administration Integration

Marketplace & Commerce must integrate with Central Administration.

Administration may govern:

* marketplace policies,
* prohibited product categories,
* merchant policies,
* transaction policies,
* commission policies,
* subscription configuration,
* payment-channel policies,
* moderation policies,
* operational settings.

However:

> Administrative governance must not replace commerce domain ownership.

Administrative actions affecting commerce must invoke the appropriate commerce/application/domain operation.

---

# 44. Subscription Management

Subscription commerce must support merchant plan lifecycle.

Conceptually:

```text
Free
 ↓
Plan Selection
 ↓
Subscription
 ↓
Capability Activation
 ↓
Usage
 ↓
Renewal / Change / Cancellation
```

The platform must distinguish:

```text
Subscription entitlement
```

from:

```text
Commerce domain authorization
```

Subscription status controls access to commercial capabilities but must not bypass ordinary ownership and security rules.

---

# 45. Free Shop Product Limit

The initial free shop requirement is:

> **Maximum five showcased products.**

The limit must be treated as a commercial policy rather than an immutable domain constant.

Future commercial plans may define different limits.

---

# 46. Commercial Policy

The following are examples of values that should remain policy/configuration driven:

* free shop product limit,
* subscription capabilities,
* transaction commission,
* category-specific commission,
* payment availability,
* geographic availability,
* merchant limits,
* inventory limits,
* warehouse limits.

These values must remain distinct from immutable technical/domain constants.

---

# 47. Order and Payment Separation

The commerce architecture must maintain a clear distinction between:

```text
Order
Payment
Fulfillment
Settlement
```

An order must not be treated as synonymous with payment.

Payment success does not automatically mean delivery.

Delivery does not automatically mean settlement unless the applicable transaction policy defines it.

---

# 48. Protected Transaction Concept

The intended platform-mediated transaction model is based on a protected commercial lifecycle.

Conceptually:

```text
Buyer places order
       ↓
Payment authorized / collected
       ↓
Transaction enters protected state
       ↓
Seller fulfills order
       ↓
Product delivered / collected
       ↓
Applicable confirmation / policy
       ↓
Settlement
       ↓
Commission retained
       ↓
Merchant receives settlement
```

The precise mechanics must depend on the capabilities of the selected payment provider and the legal/regulatory requirements of the applicable jurisdiction.

The SRS therefore defines the **business requirement**, not a legal escrow implementation.

---

# 49. Refunds and Disputes

Platform-mediated commerce must account for:

* payment failure,
* cancellation,
* refund,
* partial refund,
* delivery failure,
* non-delivery,
* damaged product,
* buyer dispute,
* seller dispute,
* payment provider dispute,
* chargeback.

The detailed dispute state machine is a future Commerce Domain Contract requirement.

---

# 50. External Transaction Boundary

When buyer and seller complete a transaction externally, SocialConnect must maintain a clear boundary.

SocialConnect may provide:

* listing,
* discovery,
* social context,
* seller/shop information,
* communication,
* local coordination.

But it must not falsely represent:

* external payment as SocialConnect payment,
* external settlement as SocialConnect settlement,
* external fulfillment as SocialConnect fulfillment.

---

# 51. Merchant Operations

Professional commerce should provide an operational layer beyond simple listings.

The merchant operating environment should ultimately cover:

```text
Shop
 ├── Catalog
 ├── Products
 ├── Inventory
 ├── Warehouses
 ├── Orders
 ├── Fulfillment
 ├── Payments
 ├── Settlement
 ├── Customers
 └── Commercial Settings
```

This is the foundation for evolving SocialConnect from simple marketplace participation into professional commerce.

---

# 52. Commerce Dashboard

Professional merchants should have a dedicated commerce operating experience.

Potential dashboard information includes:

* active products,
* stock status,
* orders,
* fulfillment status,
* payment status,
* settlement status,
* shop performance,
* subscription status.

The dashboard must use query services/view models and must not contain business logic.

---

# 53. Analytics

The platform may eventually provide merchant analytics.

Potential metrics include:

* product views,
* product interactions,
* shop visits,
* inquiries,
* orders,
* conversion,
* revenue,
* transaction commission,
* inventory movement.

Analytics requirements should remain separate from the core transactional domain.

---

# 54. Business Model

The initial SocialConnect commerce model is:

| Commercial Level           | Intended User                         | Cost Model                                          | Core Purpose                         |
| -------------------------- | ------------------------------------- | --------------------------------------------------- | ------------------------------------ |
| Individual                 | Normal users / occasional sellers     | Free                                                | Local and personal commerce          |
| Free Shop                  | Small sellers / micro-businesses      | Free                                                | Shop showcase, up to 5 products      |
| Subscription               | Professional merchants                | Paid subscription                                   | Advanced commerce operations         |
| Platform Transaction       | Merchants using SocialConnect payment | Transaction commission                              | Protected/platform-mediated commerce |
| Local External Transaction | Nearby buyer/seller                   | Potentially no SocialConnect transaction commission | Local direct commerce                |

The exact prices, percentages and plan names are commercial decisions and are not fixed by this SRS.

---

# 55. Value Creation

SocialConnect creates value through multiple layers.

## For individuals

* simple marketplace participation,
* local discovery,
* social context,
* low entry barrier.

## For small sellers

* free shop presence,
* product showcase,
* local customer discovery,
* social distribution.

## For professional merchants

* structured catalog,
* inventory,
* warehouse management,
* orders,
* payments,
* fulfillment,
* subscription capabilities.

## For buyers

* product discovery,
* local and global options,
* seller/shop context,
* social discovery,
* transaction choices.

---

# 56. Marketplace vs Social Network

Marketplace must enhance the social network without replacing it.

The platform should support:

```text
SocialConnect
├── Social
│   ├── People
│   ├── Posts
│   ├── Comments
│   ├── Reactions
│   └── Communities
│
└── Commerce
    ├── Shops
    ├── Products
    ├── Inventory
    ├── Orders
    ├── Payments
    └── Fulfillment
```

These domains interact through explicit contracts.

Neither domain should absorb the responsibilities of the other.

---

# 57. Marketplace vs Traditional E-Commerce

SocialConnect does not attempt to reproduce the entire operational scale of global e-commerce platforms.

Its distinctive product center is:

```text
Social Discovery
        +
Local Commerce
        +
Professional Commerce
```

The platform therefore supports both lightweight and professional commerce without requiring every seller to operate as a large-scale e-commerce business.

---

# 58. Marketplace vs Classifieds

SocialConnect supports classified-style local transactions but adds structured commerce capabilities.

The intended progression is:

```text
Simple Listing
       ↓
Social Discovery
       ↓
Shop
       ↓
Catalog
       ↓
Inventory
       ↓
Order
       ↓
Payment
       ↓
Fulfillment
```

This allows a user to remain at the simple-listing level or progress into professional commerce.

---

# 59. Non-Goals

The initial Marketplace & Commerce subsystem is **not** intended to:

1. Become a physical warehouse operator.
2. Operate a global delivery fleet.
3. Become a bank.
4. Become a payment processor.
5. Become a logistics provider by default.
6. Replace third-party payment providers.
7. Replace all external shipping providers.
8. Force every seller into a paid subscription.
9. Force every local transaction through SocialConnect payment.
10. Turn the social feed into a pure product advertising catalog.
11. Duplicate the canonical Notification infrastructure.
12. Duplicate the canonical Event infrastructure.
13. Duplicate the canonical Media infrastructure.
14. bypass the canonical Moderation/Administration infrastructure.

---

# 60. Architectural Boundary Requirements

Although this document is a product SRS rather than an implementation contract, the following boundaries are mandatory.

Marketplace & Commerce must not:

* directly own Identity authentication logic,
* directly manipulate unrelated domain tables,
* implement notification infrastructure,
* implement a second event bus,
* implement a second outbox,
* implement a second media uploader,
* implement a second moderation platform,
* implement a separate administration system.

Commerce-specific business logic belongs within Commerce application/domain boundaries.

Shared platform capabilities must remain shared.

---

# 61. Integration with Canonical Event Architecture

Commerce must use the finalized SocialConnect Event & Notification architecture.

The expected conceptual flow is:

```text
Commerce Business Operation
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Dispatcher
        ↓
Event Handler
        ↓
Recipient Resolution
        ↓
Notification Orchestration
        ↓
Notification Persistence / Delivery Planning
        ↓
Channel Delivery
```

Commerce must not introduce a parallel event-processing architecture.

---

# 62. Integration with Canonical Notification Architecture

Commerce notifications are consumers of the platform notification system.

Examples:

```text
Order Created
    ↓
Notification

Payment Status Changed
    ↓
Notification

Order Shipped
    ↓
Notification

Order Delivered
    ↓
Notification
```

The exact notification policy must be defined after the canonical Event & Notification platform is finalized.

---

# 63. Integration with Canonical Alerting

Commerce may later provide operational alert sources through the canonical Alerting architecture.

The generic Alert platform must remain independent of Commerce.

Commerce conditions should plug into the established alert source/policy boundary.

Commerce must not create a hard-coded central switch containing every possible commerce alert condition.

---

# 64. Security Requirements

Marketplace must provide:

* authenticated commerce operations where required,
* ownership validation,
* authorization,
* anti-forgery protection,
* payment security,
* sensitive-data protection,
* merchant isolation,
* order ownership enforcement,
* payment ownership enforcement,
* concurrency protection,
* auditability for privileged operations.

Payment credentials and provider secrets must never be stored as ordinary commerce data.

---

# 65. Concurrency Requirements

Commerce operations must be concurrency-safe.

Particularly sensitive operations include:

* inventory reservation,
* stock reduction,
* order creation,
* payment state transitions,
* cancellation,
* refund,
* settlement,
* subscription capability changes.

The system must prevent inconsistent inventory and duplicate financial operations.

---

# 66. Audit Requirements

Important commercial operations must be auditable.

Potential audited operations include:

* merchant configuration changes,
* payment configuration changes,
* inventory corrections,
* order administrative actions,
* refunds,
* disputes,
* settlement changes,
* subscription changes,
* privileged moderation actions.

Audit must follow the canonical SocialConnect audit architecture.

---

# 67. Data Ownership Principle

Commerce entities must have clear ownership.

Examples include:

```text
Shop → Merchant
Product → Shop / Seller
Inventory → Merchant / Stock Location
Order → Buyer + Seller
Payment → Transaction
Settlement → Merchant
Warehouse → Merchant
```

Administrative access does not eliminate domain ownership.

---

# 68. Extensibility Requirements

Commerce must be designed for future expansion.

Potential future capabilities include:

* product variants,
* bundles,
* wholesale,
* subscriptions/products,
* promotions,
* coupons,
* returns,
* advanced shipping,
* multi-vendor orders,
* marketplace commissions,
* tax calculation,
* international trade,
* additional payment providers,
* fulfillment providers,
* merchant analytics.

Future features must extend established boundaries rather than redesigning the core commerce model unnecessarily.

---

# 69. Commerce Lifecycle Overview

The complete intended platform flow is:

```text
User / Merchant
      ↓
Create Product / Shop
      ↓
Publish
      ↓
Social + Marketplace Discovery
      ↓
Buyer Interaction
      ↓
Transaction Choice
   ┌───────────────┴────────────────┐
   ↓                                ↓
Platform Transaction          Local / External
   ↓                                ↓
Order                          Buyer ↔ Seller
   ↓
Payment
   ↓
Protected Transaction
   ↓
Fulfillment
   ↓
Delivery / Pickup
   ↓
Completion
   ↓
Settlement
   ↓
Commission
```

---

# 70. End-to-End Commerce Principle

The complete commercial lifecycle must ultimately be understandable as:

> **Discover → Evaluate → Choose Transaction → Order → Pay → Fulfill → Receive → Complete → Settle**

For local external transactions:

> **Discover → Evaluate → Contact → Agree → Exchange Locally**

Both are legitimate SocialConnect commerce experiences.

---

# 71. Product Success Criteria

The Marketplace & Commerce subsystem should ultimately demonstrate that:

1. A normal SocialConnect user can participate in local commerce without becoming a professional merchant.
2. A small seller can establish a free shop.
3. A free shop can showcase up to five products.
4. A growing merchant can transition to a subscription plan.
5. A professional merchant can manage structured products.
6. Professional merchants can manage inventory.
7. Professional merchants can manage warehouse/stock locations.
8. Buyers can discover products through both marketplace and social contexts.
9. Geographic context works across local and broader discovery.
10. Localization supports international expansion.
11. Merchants can configure supported payment capabilities.
12. Platform-mediated transactions can support protected payment/settlement policies.
13. SocialConnect can retain a configurable transaction commission where applicable.
14. Nearby buyers and sellers can complete transactions externally.
15. External local transactions do not have to incur SocialConnect transaction commission.
16. Commerce uses the canonical Event platform.
17. Commerce uses the canonical Notification platform.
18. Commerce uses the canonical Media platform.
19. Commerce uses the canonical Moderation and Administration platform.
20. Commerce remains extensible without becoming a monolithic service or domain.

---

# 72. Future Commerce Architecture Contracts

This SRS intentionally does not define implementation details.

Before implementation, the following contracts should be finalized:

### 72.1 Commerce Domain Model Contract

Defines:

* entities,
* value objects,
* relationships,
* ownership,
* lifecycle states,
* invariants.

### 72.2 Product & Catalog Contract

Defines:

* products,
* categories,
* variants,
* pricing,
* catalog lifecycle.

### 72.3 Shop & Merchant Contract

Defines:

* shop ownership,
* merchant lifecycle,
* shop settings,
* subscription entitlements.

### 72.4 Inventory & Warehouse Contract

Defines:

* inventory,
* stock locations,
* warehouses,
* reservations,
* adjustments,
* stock movements.

### 72.5 Order & Fulfillment Contract

Defines:

* order state machine,
* fulfillment,
* pickup,
* shipping,
* cancellation,
* returns.

### 72.6 Payment & Settlement Contract

Defines:

* payment abstraction,
* provider integration,
* authorization,
* capture,
* protected transaction,
* commission,
* settlement,
* refunds,
* disputes.

### 72.7 Marketplace Search & Discovery Contract

Defines:

* product search,
* location search,
* filtering,
* ranking,
* availability,
* social discovery integration.

### 72.8 Commerce Event Integration Contract

Defines:

* commerce domain events,
* integration events,
* payload contracts,
* versions,
* idempotency,
* event ownership.

### 72.9 Commerce Notification Policy Contract

Defines:

* notification triggers,
* recipient rules,
* channels,
* preferences,
* templates.

### 72.10 Commerce Moderation & Governance Contract

Defines:

* prohibited products,
* reportable commerce targets,
* moderation policies,
* administrative actions.

---

# 73. Open Product Decisions

The following must be finalized before implementation of the corresponding functionality:

1. Exact free individual listing limits, if any.
2. Exact free shop capabilities.
3. Subscription plan names and pricing.
4. Subscription capability matrix.
5. Commission percentages and calculation rules.
6. Supported payment providers by geography.
7. Protected-payment mechanics supported by each provider.
8. Delivery confirmation policy.
9. Dispute window.
10. Refund policy.
11. Return policy.
12. Chargeback handling.
13. Shipping provider strategy.
14. Local pickup verification requirements.
15. Seller verification requirements.
16. Merchant/business verification requirements.
17. Tax responsibilities.
18. Country-specific commercial requirements.
19. Product-category restrictions.
20. Product variant requirements.
21. Warehouse feature depth.
22. Multi-warehouse allocation rules.
23. Product reservation behavior.
24. Search ranking policy.
25. Social recommendation policy.
26. Product review/rating requirements.
27. Merchant analytics scope.
28. International marketplace rollout strategy.

These decisions must be documented before their respective implementation contracts are locked.

---

# 74. Architectural Dependency Order

Marketplace & Commerce implementation should follow the broader SocialConnect architectural dependency order.

The recommended dependency chain is:

```text
Platform Foundations
        ↓
Identity / Profile
        ↓
Geographic Foundation
        ↓
Media
        ↓
Social Graph
        ↓
Post / Feed
        ↓
Event & Notification
        ↓
Alerting
        ↓
Marketplace & Commerce
        ↓
Administration / Governance Integration
```

The exact implementation sequence may be refined during architecture planning, but Marketplace must consume finalized shared platform capabilities rather than recreate them.

---

# 75. Implementation Readiness Rule

This SRS is a **product-level contract**.

It must not be interpreted as permission to begin implementation immediately.

Before implementation:

1. Requirements must be reviewed.
2. Domain boundaries must be locked.
3. Data model must be locked.
4. Commerce state machines must be locked.
5. Payment/settlement model must be locked.
6. Geographic dependencies must be confirmed.
7. Event integration must be finalized.
8. Notification policies must be finalized.
9. Administration integration must be finalized.
10. Security/authorization boundaries must be finalized.
11. API contracts must be finalized.
12. Testing strategy must be finalized.

No implementation should silently introduce architectural decisions that contradict this SRS.

---

# 76. Strategic Product Statement

SocialConnect Marketplace & Commerce is designed around a simple product proposition:

> **People should be able to discover, discuss, buy, sell and operate commerce within the same social environment in which they already interact with people and communities.**

The platform therefore combines:

```text
Social Graph
      +
Geographic Context
      +
Product Discovery
      +
Shops
      +
Inventory
      +
Warehouse Operations
      +
Orders
      +
Payment Options
      +
Fulfillment
      +
Settlement
```

while preserving the ability to support both:

```text
Local / Direct Commerce
```

and:

```text
Platform-Mediated Commerce
```

The objective is not to reproduce one existing marketplace model.

The objective is to establish a platform in which **social relationships, geographic relevance and structured commerce are first-class parts of the same product ecosystem**.

---

# 77. Canonical Boundary

The following statement is the governing boundary for this SRS:

> **Marketplace & Commerce owns commercial products, shops, merchants, inventory, warehouses, orders, fulfillment and commerce policies. It consumes shared SocialConnect capabilities for identity, geography, media, events, notifications, alerting, moderation and administration. It must not duplicate those platform capabilities or bypass their established contracts.**

---

# 78. Final Requirement

SocialConnect Marketplace & Commerce must provide a progression from:

> **“I want to sell something nearby.”**

to:

> **“I want to operate a professional shop with products, inventory, warehouses, orders, payments and fulfillment.”**

without requiring the user to leave the SocialConnect ecosystem.

The system must support the entire progression while preserving:

* social context,
* geographic relevance,
* merchant ownership,
* transaction flexibility,
* professional commerce operations,
* security,
* governance,
* extensibility,
* internationalization,
* and clear architectural boundaries.

---

**Document Status:** CANONICAL DRAFT — READY FOR PRODUCT & ARCHITECTURAL REVIEW
**Implementation Status:** NOT STARTED
**Next Step:** Review and lock Marketplace & Commerce requirements before creating the Commerce Domain/Data/Architecture contracts.
