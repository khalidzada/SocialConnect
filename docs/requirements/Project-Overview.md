# SocialConnect

## Hybrid Social Network & Marketplace Platform

**Document Status:** Canonical
**Document Version:** 2.0
**Project:** SocialConnect
**Repository:** `khalidzada/SocialConnect`

---

## 1. Project Overview

SocialConnect is a modular web platform designed to combine **social networking, community interaction, local discovery, and marketplace commerce** within a single ecosystem.

The platform is intended to bridge the gap between traditional social networks and online marketplaces by allowing users not only to communicate and share content, but also to discover, promote, and eventually trade products through seller and marketplace capabilities.

The long-term vision is positioned between several established interaction models:

* social networking and community interaction;
* local marketplace/classified discovery;
* structured online marketplace commerce;
* seller-oriented storefront and product discovery.

SocialConnect therefore treats **social interaction and commerce as connected experiences rather than completely separate systems**.

The platform is being developed incrementally. Some capabilities are already implemented, some architectural contracts are locked and undergoing implementation, and other capabilities remain planned for future development.

---

# 2. Vision

The core vision of SocialConnect is:

> **Connect people, content, places, and commerce in one platform where social interaction can naturally lead to local and global discovery and marketplace activity.**

The platform is intended to support three connected dimensions:

### Social

Users can build profiles, publish content, interact with other users, react to content, comment, share content, and receive notifications.

### Marketplace

Users can eventually participate as sellers through shops, products, listings, and commerce workflows.

### Discovery

Users can discover social content and marketplace content according to relevant context, including geographic and eventually regional/global context.

---

# 3. Product Direction

SocialConnect is not intended to be only a social network with a shop attached to it.

Its long-term product direction is:

```text
                 SOCIALCONNECT

       Social Interaction & Community
                    │
                    ▼
              Content & Feed
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Social Discovery     Product Discovery
          │                   │
          └─────────┬─────────┘
                    ▼
              Marketplace
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Local Commerce      Global Commerce
```

This direction allows the platform to evolve from social interaction into location-aware discovery and eventually broader marketplace participation.

---

# 4. Target Users

SocialConnect is intended to support several user categories.

## 4.1 Social Users

Users primarily interested in:

* profiles;
* social posts;
* comments;
* reactions;
* sharing;
* discovering content;
* interacting with other users;
* receiving notifications.

## 4.2 Sellers

Users who eventually use SocialConnect as a marketplace channel for:

* shops;
* product listings;
* product media;
* seller identity;
* product discovery;
* local and broader commerce.

## 4.3 Buyers

Users who discover products through:

* social content;
* marketplace discovery;
* seller profiles;
* shops;
* local discovery;
* broader marketplace search.

## 4.4 Advertisers and Businesses

A future business capability may support:

* sponsored content;
* promoted products;
* business visibility;
* advertising campaigns;
* analytics.

These capabilities are part of the product direction and are not to be interpreted as current implemented functionality unless explicitly documented elsewhere as implemented.

## 4.5 Platform Administrators and Moderators

Privileged users responsible for:

* governance;
* moderation;
* reporting;
* platform configuration;
* operational management;
* compliance-related workflows;
* administration.

---

# 5. Core Product Domains

SocialConnect is being developed as a modular platform with clearly separated ownership boundaries.

Major domains include:

### Identity & Profiles

Responsible for identity-related platform capabilities and user profile functionality.

### Social Content

Responsible for posts and associated social interactions.

### Comments

Responsible for comment creation, replies, editing, lifecycle, and associated UI behaviour.

### Reactions

Responsible for user reactions against supported targets.

### Media

Responsible for media upload, temporary storage, finalization, assignment, and media lifecycle.

### Feed & Discovery

Responsible for assembling and presenting social and eventually marketplace content.

### Sharing

Responsible for internal and external sharing semantics.

### Events

Responsible for domain/integration event infrastructure and transactional event publication.

### Notifications

Responsible for notification persistence, querying, command operations, delivery planning, and notification presentation integration.

### Alerting

Responsible for the generic operational alerting platform.

### Administration

Responsible for privileged governance and administration capabilities.

### Marketplace

The marketplace bounded context is a major future domain and will progressively introduce shops, products, listings, commerce workflows, and marketplace governance.

---

# 6. Social and Marketplace Relationship

SocialConnect uses a conceptual relationship between the Social and Marketplace contexts.

```text
┌─────────────────────┐             ┌─────────────────────────┐
│   Social Context    │             │   Marketplace Context   │
│                     │             │                         │
│ Profiles            │             │ Shops                   │
│ Posts               │             │ Products                │
│ Comments            │             │ Listings                │
│ Reactions           │             │ Seller capabilities      │
│ Sharing             │             │ Commerce workflows       │
│ Feed                │             │ Marketplace discovery    │
└──────────┬──────────┘             └────────────┬────────────┘
           │                                     │
           └──────────── Shared Kernel ──────────┘
                         Identity / Geo
```

The shared concepts must not become an excuse for merging bounded-context responsibilities.

Each domain owns its own business behaviour.

---

# 7. Location as a Platform Capability

Location is a significant part of the long-term SocialConnect product direction.

Location is not treated merely as a GPS coordinate attached to a request.

It can provide context for:

* local content discovery;
* nearby products;
* seller discovery;
* marketplace proximity;
* regional experiences;
* country-aware behaviour;
* future localization and globalization.

The platform therefore distinguishes conceptually between:

### Geographic Location

Where a person, product, shop, or piece of content is geographically relevant.

### Localization

How the platform adapts to a user's region or language, including future support for:

* language;
* currency;
* date/time formats;
* number formats;
* regional presentation.

### Globalization

The ability for the platform to support users, sellers, products, and commerce across multiple countries and regions.

These concepts will be documented more deeply as the marketplace and localization capabilities are implemented.

---

# 8. Social Features

The social platform is intended to support:

* user profiles;
* profile information;
* posts;
* text content;
* media;
* comments;
* nested replies;
* reactions;
* internal sharing;
* external sharing;
* feed timelines;
* notifications;
* future social graph capabilities.

Social functionality is implemented through reusable, independently owned components and services.

---

# 9. Marketplace Direction

The marketplace direction combines characteristics of:

* local marketplace/classified platforms;
* structured online marketplaces;
* seller-oriented storefront platforms.

The long-term marketplace can support:

* seller identity;
* virtual shops;
* product listings;
* product media;
* categories;
* product discovery;
* local discovery;
* regional marketplaces;
* global marketplace participation;
* future commerce and payment workflows.

The marketplace must be implemented as an independently governed domain rather than being embedded inside the social domain.

---

# 10. Feed and Discovery

The long-term feed experience is intended to bring together relevant platform content.

Conceptually:

```text
Social Content ───────┐
                      ├──► Discovery / Feed ───► User
Marketplace Content ──┘
```

The feed architecture must remain extensible so that future discovery strategies can be introduced without coupling individual content domains to the feed implementation.

Future recommendation or machine-learning capabilities are considered an extension of the discovery platform, not a foundational dependency of the current architecture.

---

# 11. Current Technology Foundation

The current application is built around:

* ASP.NET Core MVC
* .NET 8
* C#
* Entity Framework Core
* SQL Server
* ASP.NET Identity
* Razor Views
* Bootstrap 5
* JavaScript
* REST-style APIs where appropriate
* FluentValidation
* Generic Repository and Unit of Work infrastructure
* DTO-based application contracts
* modular layered application architecture

Manual mapping is used.

**AutoMapper is not part of the current architecture.**

---

# 12. Architectural Style

SocialConnect is implemented as a **modular monolith with strong architectural boundaries**.

The architecture intentionally avoids prematurely converting the application into distributed microservices.

The current direction is:

```text
                    SocialConnect
                         │
              Modular Monolithic App
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
   Presentation      Application        Domain
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                   Infrastructure
                         │
                      SQL Server
```

The architecture is designed so that well-defined module boundaries can evolve independently without requiring premature physical service separation.

---

# 13. Core Architecture Rules

The following principles are canonical.

## Controller

Controllers:

* receive requests;
* perform boundary-level request handling;
* invoke application services/orchestrators;
* return responses.

Controllers do not own business workflows.

---

## Service

Services contain business rules and application behaviour appropriate to their responsibility.

Large services must be decomposed.

A large workflow should become:

```text
Orchestrator
   │
   ├── Context Builder
   ├── Loader
   ├── Resolver
   ├── Factory / Policy
   └── Result / ViewModel construction
```

---

## Repository

Repositories contain persistence operations only.

Repositories do not own business rules.

---

## DbContext

`DbContext` is an infrastructure concern.

Business services must not treat the DbContext as their business API.

---

## DTO

DTOs represent service/application transport contracts.

---

## ViewModel

ViewModels exist for UI rendering.

---

## Entity

Entities represent domain concepts and domain state.

---

# 14. Reusable Component Architecture

Reusable UI components follow a strict ownership model.

Each component owns:

* state;
* lifecycle;
* validation relevant to its UI responsibility;
* UI behaviour;
* events;
* internal implementation.

Consumers:

* configure the component;
* call its public API;
* listen to its events;
* react to results.

Consumers must never control internal component implementation.

---

# 15. Component Communication

Components communicate only through public contracts.

For example:

```text
CommentManager
      │
      ▼
MediaUploader Public API
      │
      ▼
MediaUploader Events
      │
      ▼
Result
```

A consumer must never depend on:

* internal JavaScript objects;
* private functions;
* XHR instances;
* internal file collections;
* internal object URLs;
* implementation-specific state.

---

# 16. Component State

Reusable components own their own state machines.

For example:

```text
MediaUploader

Idle
  ↓
Files Selected
  ↓
Uploading
  ↓
Uploaded
  ↓
Awaiting Consumer
  ↓
Reset
  ↓
Idle
```

CommentManager independently owns its own lifecycle.

State machines must never be merged between components.

---

# 17. Server / Client Boundary

The server owns:

* validation;
* authorization;
* persistence;
* workflows;
* business rules;
* domain state.

The client owns:

* interaction;
* rendering enhancements;
* client-side component state;
* UI coordination;
* public component event handling.

JavaScript must not duplicate server business rules.

---

# 18. ViewComponent Architecture

Reusable UI is implemented as ViewComponents.

The canonical structure is:

```text
ViewComponent
      ↓
ViewModel
      ↓
Default.cshtml
      ↓
Optional Partials
      ↓
JavaScript
      ↓
CSS
```

Builders configure reusable components.

Defaults provide centralized default configuration.

---

# 19. JavaScript Architecture

Feature JavaScript uses the application module infrastructure.

Reusable JavaScript components follow:

* `App.Modules.register`;
* isolated module state;
* `WeakMap` where appropriate;
* immutable configuration;
* explicit initialization;
* binding;
* destruction;
* public APIs;
* domain/business events.

`site.js` remains foundation-only.

Feature-specific API and business logic does not belong in `site.js`.

---

# 20. Media Architecture

The current media architecture follows the locked pipeline:

```text
MediaUploader
      ↓
/api/media/upload-async
      ↓
MediaUploadService
      ↓
Temporary Storage
uploads/temp/
      ↓
Media Domain
      ↓
Media IDs
      ↓
Media Finalization / Assignment
```

The responsibilities are deliberately separated.

### Detection

Discovers facts.

### Validation

Verifies facts.

### Storage

Persists facts.

The rule is:

> **Detection discovers facts. Validation verifies facts. Storage persists facts.**

No validator may discover information.

No detector may enforce business rules.

No storage service may perform validation.

---

# 21. Post Creation

Post creation follows the canonical workflow:

```text
Post Composer
      ↓
CreatePostDto
      ↓
PostCreationContext
      ↓
PostCreationService
      ↓
Create Post
      ↓
Finalize Media
      ↓
Assign Media
      ↓
Publish Feed
      ↓
PostCreationResult
```

PostService does not become the owner of the complete post creation workflow.

---

# 22. Sharing

SocialConnect supports distinct sharing concepts.

### Internal Share

An internal share creates a new Post referencing the original post.

The original content/media is not duplicated.

`SharedPostId` represents the relationship.

The relationship uses restricted deletion semantics.

### External Share

Represents sharing outside the SocialConnect internal post graph.

---

# 23. Events and Notifications

SocialConnect distinguishes:

* Domain Events;
* Integration Events;
* Outbox Events.

Events are persisted transactionally where required through the transactional outbox architecture.

The broader lifecycle is:

```text
Business Operation
      ↓
Event Raised
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
Persistence / Delivery Planning
      ↓
Delivery
      ↓
Tests
```

The notification database is the persistence source of truth for notifications.

Real-time delivery mechanisms such as SignalR are delivery mechanisms, not the notification source of truth.

---

# 24. Recipient Resolution

Recipient resolution is a separate architectural concern.

Candidate sources include:

* Post Owner;
* Comment Owner;
* Target Owner;
* Parent Comment Author.

Recipient candidate sources are resolved independently and then composed through the candidate-source architecture.

Recipient resolution must not become embedded inside individual event handlers.

---

# 25. Alerting

SocialConnect contains a generic Alerting architectural direction.

The Alerting platform must remain generic.

The Alert Source Catalog must **not become a giant hard-coded switch** containing every operational condition in the platform.

Instead:

```text
Operational Condition
        ↓
Alert Source / Policy Boundary
        ↓
Generic Alerting Platform
```

The initial operational evidence domain is the Event/Notification platform.

The generic Alerting backend is completed and verified independently before additional alert sources are introduced.

Future sources are introduced one vertical slice at a time.

---

# 26. External Integration Boundary

Every external provider or API integration terminates at an Infrastructure Adapter Boundary.

Application and domain services consume internal contracts.

They must not depend directly on:

* provider SDKs;
* provider endpoints;
* provider authentication mechanisms;
* provider payload formats;
* provider-specific response models.

This allows external integrations to change without contaminating the application/domain architecture.

---

# 27. Administration

Administration is a privileged subsystem within the same SocialConnect application.

It is not a separate application.

ASP.NET Core Areas are not used for Administration.

Administration follows the same application architecture and domain ownership principles.

The current role model is:

* Admin;
* Moderator;
* Vendor;
* User.

There is no SuperAdmin role in the current architecture.

Administration is responsible for governance and privileged operations while respecting domain ownership.

Administrators must not bypass domain/application services to manipulate domain tables directly.

---

# 28. Development Philosophy

SocialConnect is being developed incrementally.

The project prioritizes:

* explicit architectural contracts;
* clear ownership;
* small services;
* separation of concerns;
* reusable components;
* server-owned business logic;
* deterministic workflows;
* testable boundaries;
* documented decisions;
* controlled extensibility.

The project does not introduce architectural complexity simply because a technology is available.

---

# 29. Implementation Status

SocialConnect documentation distinguishes implementation status.

### Implemented

Functionality exists in the current codebase.

### Locked

The architecture or contract has been finalized and should not be redesigned unless a genuine contradiction is discovered.

### In Development

The architecture is defined and implementation is underway.

### Planned

Accepted future capability that is not yet implemented.

### Exploratory

A potential future product capability that has not yet become a committed implementation contract.

### Historical

An earlier design or implementation assumption retained for project history.

---

# 30. Long-Term Product Direction

The long-term SocialConnect platform can evolve toward:

```text
Social Identity
      ↓
Social Interaction
      ↓
Content Discovery
      ↓
Local Discovery
      ↓
Marketplace
      ↓
Regional Commerce
      ↓
Global Commerce
```

Future capabilities may include:

* advanced marketplace workflows;
* payments;
* seller analytics;
* advertising;
* subscriptions;
* recommendation systems;
* broader localization;
* international marketplace support;
* additional communication capabilities.

These are future product directions unless separately marked as implemented.

---

# 31. Documentation Strategy

The repository maintains two documentation generations.

### Current Canonical Documentation

Located primarily under:

```text
docs/
```

These documents represent the current architecture and implementation contract.

### Historical Documentation

The original HTML documents remain available for project history.

They describe earlier design decisions and should not override current canonical documentation.

The root `README.md` provides the navigation point between current and historical documentation.

---

# 32. Engineering Principle

The central architectural principle of SocialConnect is:

> **A reusable component owns everything within its domain. Consumers configure it, call its public API, subscribe to its events, and react to its results. Consumers never implement or control the component's internal behaviour.**

This principle extends beyond UI components into the broader application architecture:

> **Every domain and application service should own its responsibility completely while exposing stable contracts to the rest of the system.**
