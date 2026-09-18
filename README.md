# SocialConnect

## Hybrid Social Network & E-Commerce Platform

**SocialConnect** is a long-term software engineering project for designing and developing a modular social-network and e-commerce platform using the **Microsoft .NET ecosystem**.

The project covers the complete lifecycle of a modern application — from product requirements and architecture through domain modeling, implementation, database design, APIs, reusable UI components, media processing, event-driven workflows, notifications, alerting, testing, and technical documentation.

> **Project status:** Active development and continuous architectural refinement.

---

# 🎯 Project Overview

SocialConnect combines social interaction and commerce capabilities within a single platform.

The platform is designed around concepts such as:

* User identity and profiles
* Social relationships and social graph
* Posts and content publishing
* Comments and threaded discussions
* Reactions
* Sharing
* Media management
* Feed generation and discovery
* Notifications
* Event-driven processing
* Operational alerting
* Marketplace and e-commerce capabilities
* Products, shops, and vendors
* Administrative governance and moderation
* Reporting and review workflows
* Auditing and soft deletion
* API-based application integration
* Local, regional, and global discovery
* Localization and globalization
* Future messaging and communication capabilities

The long-term product direction is to bridge **social discovery and marketplace commerce**, allowing community interaction, content discovery, local discovery, and buying/selling capabilities to coexist within one platform.

The project emphasizes:

* Maintainability
* Explicit ownership
* Separation of responsibilities
* Modular architecture
* Reusable components
* Stable contracts
* Testability
* Production-oriented engineering
* Traceable architectural decisions
* Long-term extensibility

---

# 🏗️ Technology Stack

## Backend

* **C#**
* **.NET 8**
* **ASP.NET Core MVC**
* **Entity Framework Core**
* **ASP.NET Core Identity**
* **REST APIs**
* **FluentValidation**

## Database

* **Microsoft SQL Server**
* Entity Framework Core
* Relational data modeling
* Database constraints and relationships
* Transactions
* Auditing
* Soft deletion
* Concurrency handling

## Frontend

* **Razor Views**
* **Razor ViewComponents**
* **Bootstrap 5**
* HTML5
* CSS
* JavaScript

## Architecture & Engineering

* Layered modular monolith
* DTO-based application boundaries
* Generic Repository
* Unit of Work
* Application services
* Domain/business services
* Entity loader abstractions
* Lookup services
* Context builders
* Resolvers
* Factories
* Policies
* Event-driven architecture
* Transactional Outbox
* Notification orchestration
* Generic alerting
* Reusable UI components
* Manual mapping
* Automated testing
* Structured observability

---

# 🧭 Architecture at a Glance

SocialConnect is designed as a **layered modular monolith**.

The application remains one ASP.NET Core MVC application while maintaining explicit boundaries between domains and technical layers.

```text
┌──────────────────────────────────────────────┐
│                  UI / MVC                    │
│ Razor Views / ViewComponents / JavaScript    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│              Controllers / API               │
│              Thin Endpoints                  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│          Application Orchestration           │
│                                              │
│ Context Builders                             │
│ Loaders / Lookup Services / Resolvers        │
│ Factories / Policies                         │
│ DTOs / Results / ViewModels                  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│             Business / Domain               │
│                                              │
│ Domain Rules                                 │
│ Business Services                            │
│ State Transitions                            │
│ Domain Ownership                             │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│          Persistence / Infrastructure         │
│                                              │
│ EF Core / SQL Server                         │
│ Identity                                     │
│ External Provider Adapters                   │
└──────────────────────────────────────────────┘
```

The architecture intentionally separates:

> **UI → Application → Domain/Business → Persistence/Infrastructure**

Cross-cutting capabilities provide reusable infrastructure without taking ownership of the business domains that consume them.

---

# 🧩 Core Architectural Principles

SocialConnect follows explicit architectural rules.

## 1. Modular Monolith

SocialConnect remains a **modular monolith**.

Modules have explicit ownership and contracts while operating inside the same ASP.NET Core application.

Microservices are not part of the current architecture.

---

## 2. Explicit Module Ownership

Every important business concept has an authoritative owner.

Examples:

| Capability         | Authoritative Owner             |
| ------------------ | ------------------------------- |
| Identity           | Identity                        |
| Business profile   | User Profile                    |
| Posts              | Post/Social Content             |
| Comments           | Comment                         |
| Reactions          | Reaction                        |
| Media              | Media                           |
| Feed               | Feed                            |
| Friendship         | Social Graph                    |
| Following          | Social Graph                    |
| Blocking           | Social Graph                    |
| Groups             | Group/Social Graph boundary     |
| Communities        | Community/Social Graph boundary |
| Notifications      | Notification                    |
| Events             | Event infrastructure            |
| Operational alerts | Alerting                        |
| Administration     | Administration                  |
| Marketplace        | Commerce/Marketplace            |

Consumers must use the owning module's application/domain contracts rather than bypassing ownership through direct persistence access.

---

## 3. Thin Controllers

Controllers are transport/application entry points.

Controllers should:

* Authenticate the request
* Bind DTOs
* Invoke application services
* Return the appropriate response

Controllers should not contain:

* Business workflows
* Persistence logic
* Complex domain rules
* Event orchestration
* Notification orchestration
* Feed ranking
* Alerting logic

---

## 4. Program.cs Is the Composition Root

`Program.cs` is responsible for application composition.

Feature registrations belong to their owning modules through module registration methods such as:

```csharp
AddIdentityModule();
AddPostModule();
AddCommentModule();
AddReactionModule();
AddFeedModule();
AddNotificationModule();
```

Feature-specific service registrations should not accumulate inside `Program.cs`.

---

## 5. Application Orchestration

Application services coordinate use cases.

Complex workflows may be decomposed through:

* Context Builders
* Entity Loaders
* Tracked Entity Loaders
* Lookup Services
* Resolvers
* Factories
* Policies

The architecture explicitly avoids large **God Services**.

---

## 6. Manual Mapping

DTOs, domain entities, and ViewModels are explicitly separated.

SocialConnect uses **manual mapping**.

AutoMapper is not part of the current architecture.

---

## 7. Server Authority

The server is authoritative for:

* Authentication
* Authorization
* Validation
* Business rules
* Database state
* Ownership
* Visibility
* Transactions
* Event creation
* Notification persistence
* Feed eligibility
* Operational policy

Client-side JavaScript must never be treated as authoritative.

---

# 🗃️ Domain Entity Foundation

The shared entity hierarchy follows the established domain model:

```text
BaseEntity
    │
    ▼
AuditableEntity
    │
    ▼
SoftDeleteEntity
    │
    ▼
Reactable
```

Where appropriate:

* `BaseEntity` provides identity.
* `AuditableEntity` provides creation/update auditing.
* `SoftDeleteEntity` provides soft deletion.
* `Reactable` provides reaction aggregates.

Not every entity is automatically `Reactable`.

`ApplicationUser` remains outside this domain hierarchy because it belongs to ASP.NET Identity infrastructure.

---

# 🎯 Target-Based Architecture

Cross-domain target operations use:

```text
TargetType + TargetId
```

This mechanism provides a consistent reference model for supported target-based functionality such as:

* Reactions
* Comments
* Ownership resolution
* Reporting
* Moderation
* Feed interaction
* Other supported target-oriented capabilities

Target resolution remains explicit and must not become hidden coupling between modules.

---

# 👤 Identity & User Profile

Identity and business profile responsibilities are separated.

```text
ApplicationUser
       │
       ├── Authentication
       ├── Identity
       └── Account Infrastructure
       
UserProfile
       │
       ├── Business Profile Data
       ├── Profile Presentation
       └── Application-Level User Information
```

The application does not use `ApplicationUser` as a general-purpose business entity.

### Documentation

* [`SocialConnect Application User and Identity`](docs/Modules/ApplicationUser/AppUserModel.md)
* [`SocialConnect Application User Profile`](docs/Modules/UserProfile/UserProfileSRS.md)

---

# 📝 Social Content

The Social Content domain covers the core content lifecycle of SocialConnect.

It includes:

* Posts
* Post creation
* Comments
* Reactions
* Sharing
* Media association
* Feed integration

### Documentation

* [`SocialConnect Social Media Functional Requirements`](docs/Modules/SocialMedia/SocialMediaFunctionalReq.md)

The Social Media documentation establishes the functional requirements for the social-content subsystem.

---

# 📝 Post Creation

Post creation is an explicit application workflow.

```text
Post Composer
      │
      ▼
CreatePostDto
      │
      ▼
PostCreationContext
      │
      ▼
PostCreationService
      │
      ├── Create Post
      ├── Finalize Media
      ├── Assign Media
      └── Apply Post Business State
      │
      ▼
PostCreationResult
```

Post creation and Feed generation remain **separate responsibilities**.

A created Post may become a Feed candidate through the Feed eligibility pipeline, but Post creation does not itself own Feed distribution.

---

# 🖼️ Media Management

Media is a cross-cutting application capability with its own ownership and lifecycle.

The canonical pipeline is:

```text
MediaUploader
      │
      ▼
/api/media/upload-async
      │
      ▼
MediaUploadService
      │
      ▼
Temporary Storage
      │
      ▼
Media Domain
      │
      ▼
Media IDs
      │
      ├─────────────────┐
      ▼                 ▼
Media Finalization   Media Assignment
      │                 │
      ▼                 ▼
Final Storage       Entity Association
```

## Media Responsibilities

### MediaUploadService

The only upload gateway.

### MediaFinalizationService

Responsible for:

```text
Temporary Media → Final Media
```

It does not assign media to entities.

### MediaAssignmentService

Responsible for associating already-finalized media with an owning entity.

It does not perform finalization.

### MediaUploader

The reusable `MediaUploader` component is consumer-independent.

Current public lifecycle/API concepts include:

* `init`
* `destroy`
* `getInstance`
* `getMediaItems`
* `getMediaIds`
* `getExistingMedia`
* `getRemovedExistingMediaIds`
* `upload`
* `remove`
* `removeExisting`
* `cancel`

The component uses explicit lifecycle states and `media.uploader.*` events.

---

# 💬 Comments

The Comment subsystem supports:

* Comment creation
* Replies
* Editing
* Threaded discussions
* Target-based ownership
* Media integration
* Reaction integration
* Lazy loading
* Reusable UI

The reusable UI component is:

```text
CommentManager
```

Its current component architecture includes:

```text
Comment Section
      │
      ▼
CommentManager
      │
      ├── Idle
      ├── Create
      ├── Reply
      └── Edit
```

The component uses server-provided state and configuration while JavaScript manages interaction and lifecycle.

---

# ❤️ Reactions

Reactions use a shared target-oriented model.

The canonical mutation gateway is:

```text
/api/reaction/toggle
```

The server determines whether the operation represents:

```text
No existing reaction + new type
        → Added

Existing reaction + different type
        → Changed

Existing reaction + same type
        → Removed
```

The state mutation, aggregate update, event creation, and Outbox persistence belong to the same business transaction.

Reaction events can contribute to downstream notification and Feed engagement/ranking signals, but a reaction does not automatically create a Feed story.

---

# 🔁 Sharing

SocialConnect supports:

```text
ShareType
├── None
├── Internal
└── External
```

Internal sharing creates a new Post referencing the original Post.

```text
Original Post
      │
      ▼
SharedPostId
      │
      ▼
New Post
```

The original content and media are not duplicated.

The Feed layer resolves the shared-content graph with bounded depth and cycle protection.

---

# 📰 Feed & Discovery

Feed is a **distribution and presentation system**, not the authoritative owner of Post content.

The canonical conceptual pipeline is:

```text
Authoritative Domain State
          │
          ▼
Event / Query Inputs
          │
          ▼
Candidate Generation
          │
          ▼
Eligibility
          │
          ▼
Distribution
          │
          ▼
UserFeed / Candidate Inventory
          │
          ▼
Ranking
          │
          ▼
Contextual / Diversity Processing
          │
          ▼
Pagination
          │
          ▼
FeedTimelineService
          │
          ▼
FeedTimelineContextBuilder
          │
          ▼
FeedReactionResolver
          │
          ▼
FeedCardFactory
          │
          ▼
Razor UI
```

The architecture explicitly separates:

```text
Content
Candidate
Eligibility
Distribution
Ranking
Timeline
Rendering
```

### Feed Surfaces

The system supports the architectural distinction between:

* Personalized/Home discovery
* Latest/chronological content

### Feed Rules

Feed must:

* Respect visibility
* Respect authorization
* Respect blocking
* Respect moderation
* Respect Feed controls
* Avoid duplicating authoritative Post state
* Remain rebuildable
* Support deterministic pagination
* Separate ranking from candidate generation
* Allow future ranking evolution without redesigning candidate generation

### Documentation

* [`SocialConnect — Feed Engine Requirements`](docs/Modules/Feed/Feed-Engine-Requirements.md)

---

# 🧑‍🤝‍🧑 Social Graph

SocialConnect treats the Social Graph as the authoritative relationship fabric.

The established capability boundaries include:

```text
Social Graph
│
├── Friendship
├── Following
├── Blocking
├── Groups
└── Communities
```

The architecture intentionally avoids a giant:

```text
SocialGraphService
```

Instead, capabilities use focused services such as:

```text
FriendshipCommandService
FriendshipQueryService

FollowingCommandService
FollowingQueryService

BlockingCommandService
BlockingQueryService

GroupCommandService
GroupQueryService
GroupMembershipService

CommunityCommandService
CommunityQueryService
CommunityMembershipService
```

Groups and Communities remain first-class social spaces while Posts, Comments, Reactions, and Media retain their own domain ownership.

Feed consumes Social Graph context.

The Social Graph does not directly create Feed items.

Blocking is a cross-module policy input and must not be independently reimplemented by Feed, Notifications, or other modules.

### Documentation

* [`SocialConnect — Social Graph Architecture`](docs/Modules/SocialGraph/Social-Graph-Architecture.md)

---

# 🔔 Notifications

Notifications are a platform capability rather than a responsibility of individual business modules.

The architectural boundary is:

```text
Business Event
      │
      ▼
Recipient Resolution
      │
      ▼
Notification Orchestration
      │
      ▼
Notification Persistence
      │
      ▼
Delivery Planning
      │
      ├── In-App
      ├── Email
      ├── Push
      └── SMS
```

Notification persistence is the source of truth.

Real-time transport is not the source of truth.

The finalized Notification application layer includes:

* Notification queries
* Notification commands
* Authenticated-user ownership enforcement
* Deterministic pagination
* Unread count
* Expiration-aware visibility
* Read/unread operations
* Mark-all-read

The reusable UI direction is a single:

```text
NotificationManager
```

rather than separate Bell/Count/List/Center components.

---

# ⚙️ Event Processing

SocialConnect uses a transactional event architecture.

The conceptual pipeline is:

```text
Business Operation
      │
      ▼
Domain State Change
      │
      ▼
Domain Event
      │
      ▼
Transactional Outbox
      │
      ▼
Dispatcher
      │
      ▼
Event Handler
      │
      ▼
Downstream Application Effects
```

The Event/Outbox platform provides:

* Durable event persistence
* Dispatch status
* Retry
* Lease/claim processing
* Dead-letter handling
* Idempotency
* SQL Server-safe worker processing

Business modules do not create parallel event or Outbox infrastructures.

---

# 🚨 Operational Alerting

SocialConnect has a generic operational Alerting architecture.

However, the **Alerting architecture and implementation documents are currently retained as internal/reference architecture and are intentionally not included in the public README documentation index at this stage**.

The current public documentation therefore does not treat Alerting as a released documentation artifact.

The architectural direction remains:

```text
Operational Signal
      │
      ▼
Alert Source / Policy
      │
      ▼
Generic Alert Platform
      │
      ▼
Alert Evaluation
      │
      ▼
Alert Lifecycle
```

The generic Alert platform must not become a giant module-specific switch.

---

# 🛠️ Worker Health & Operational Reliability

Background workers share a common operational-health model.

Worker health evidence includes concepts such as:

* Heartbeat
* Progress
* Last successful operation
* Last failure
* Current operation
* Stale heartbeat
* Overall health

Worker Health provides operational evidence.

Alerting evaluates whether that evidence represents a significant operational condition.

These responsibilities remain separate.

There is no universal self-healing or retry service.

---

# 🛡️ Administration, Governance & Moderation

Administration is a privileged subsystem inside the existing MVC application.

## Administration does not use ASP.NET Core Areas.

The application uses the unified authentication gateway:

```text
/Account/Login
```

Authenticated users are directed according to their platform role and administrative permissions.

Current platform roles are:

```text
Admin
Moderator
Vendor
User
```

There is no `SuperAdmin` role in V1.

---

## Administrative Ownership

Administration must never bypass domain ownership through direct table manipulation.

The canonical administrative workflow is:

```text
Admin / Moderator
       │
       ▼
Admin UI
       │
       ▼
HTTP Request
       │
       ▼
Authorization
       │
       ▼
Validation
       │
       ▼
Admin Application Service
       │
       ▼
Appropriate Domain/Application Service
       │
       ▼
Transaction
       │
       ├── State Change
       ├── Audit
       └── Event + Outbox
       │
       ▼
Commit
       │
       ▼
Dispatcher
       │
       ▼
Downstream Effects
```

---

## Reporting vs Moderation

Reporting and moderation are distinct concepts.

```text
Report
  │
  ▼
Review / Case
  │
  ▼
Decision
  │
  ▼
Action
```

Moderation policy determines applicable governance behavior.

Administration also provides governance for:

* Marketplace
* Notifications
* Event policies
* Operational configuration
* Maintenance Mode

### Configuration Categories

Administrative configuration is organized around categories such as:

* Platform
* Security
* Moderation
* Marketplace
* Notification
* Event
* Operational

Admin-configurable policy must remain distinct from immutable domain constants.

### Documentation

* [`Administration Architecture`](docs/architecture/Administration-Architecture.md)
* [`Administration Panel SRS`](docs/Modules/Administration/Admin-SRS.md)

---

# 🌍 Localization, Globalization & Geographic Discovery

SocialConnect distinguishes between:

### Geographic Location

Where something or someone is associated with:

* Country
* Region
* City
* Other supported geographic context

### Localization

How information is presented for a particular locale, including:

* Language
* Culture
* Date formatting
* Number formatting
* Currency presentation
* Timezone
* Regional conventions

### Globalization

The broader platform capability required to support users, products, content, and commerce across different countries, cultures, currencies, and regional rules.

The platform direction includes:

* Local discovery
* Regional discovery
* Global discovery
* Locale-aware experiences
* Product localization
* Search implications
* Commerce implications
* Regional product reach
* Future taxation/shipping/currency requirements

Country/City data alone is therefore not considered a complete localization/globalization system.

Privacy remains an important architectural concern.

Continuous user location tracking is not assumed by default.

---

# 💬 Messages & Conversations

Messaging is treated as a separate application domain.

The cross-cutting architectural rule is that:

> **SignalR is transport infrastructure, not the messaging database or business-logic owner.**

Where messaging is implemented:

```text
Message Business Operation
          │
          ▼
Application / Domain
          │
          ▼
Database
          │
          ├── Message State
          ├── Conversation State
          └── Participants
          
SignalR
          │
          ▼
Real-Time Transport
```

Messaging must integrate with existing platform capabilities rather than creating parallel:

* Media infrastructure
* Notification infrastructure
* Authentication
* Authorization
* Event infrastructure

A dedicated Messages & Conversations implementation contract remains a module-level artifact and is finalized separately before implementation.

---

# 🛒 Marketplace & Commerce

SocialConnect includes a marketplace domain intended to support:

* Products
* Shops
* Vendors
* Listings
* Orders
* Payments
* Commerce discovery
* Local marketplace reach
* Regional/global commerce

Marketplace functionality remains domain-owned.

Administrative governance must operate through the appropriate Marketplace/Commerce application and domain services.

External commerce providers are isolated through adapter boundaries.

Commerce capabilities are also expected to integrate with:

* Localization
* Geographic discovery
* Feed/Discovery
* Media
* Notifications
* Events
* Administration
* Alerting

without transferring business ownership between these modules.

---

# 🔌 External Integration Architecture

External providers are isolated behind adapter boundaries.

```text
Application / Domain
        │
        ▼
Internal Contract
        │
        ▼
Infrastructure Adapter
        │
        ▼
External Provider
```

Application and domain services must not directly depend on:

* Provider SDKs
* Provider payload models
* Provider endpoints
* Provider authentication mechanisms

External integrations should therefore remain replaceable without redesigning the domain layer.

---

# 🧱 Persistence Architecture

SocialConnect uses:

```text
Entity Framework Core
        │
        ▼
Generic Repository
        │
        ▼
Unit of Work
        │
        ▼
SQL Server
```

### Repository Rules

Normal read queries use no-tracking behavior.

Tracked queries are used explicitly for update workflows where required.

Business logic does not leak into repositories.

Repositories remain persistence-oriented.

### Transactions

Where applicable, a transaction must atomically protect:

```text
Business State Change
+
Audit
+
Domain Event / Outbox
```

External network operations are not assumed to participate in SQL transactions.

---

# 🧾 Audit & Soft Delete

Where applicable, entities use auditable and soft-delete behavior.

Audit information includes concepts such as:

* CreatedAt
* CreatedBy
* UpdatedAt
* UpdatedBy
* DeletedAt
* DeletedBy

Soft deletion is a domain/persistence concern and must not be treated as physical deletion by default.

Administrative audit records are protected from unauthorized editing or deletion through the UI.

---

# 🔐 Security & Authorization

Security is enforced server-side.

The architecture distinguishes:

```text
Authentication
      │
      ▼
Identity
      │
      ▼
Authorization
      │
      ▼
Capability / Permission
      │
      ▼
Operation
```

Security rules include:

* Server-side authorization
* Ownership validation
* Target authorization
* Anti-forgery protection
* Input validation
* Safe error messages
* Protected configuration/secrets
* External provider isolation
* No trust in browser-supplied identity/ownership claims

Administrative authorization follows:

```text
System Role
      │
      ▼
Administrative Capability / Permission
      │
      ▼
Authorized Operation
```

---

# ⚙️ Configuration & Policy

Configuration is divided conceptually between:

### Immutable Domain Constants

Rules that define domain identity or invariants and must not be casually changed through administration.

### Configurable Policies

Operational/business policies that may be configurable where explicitly supported.

The Administration system must not turn every constant into an editable setting.

Typed configuration categories include:

```text
Platform
Security
Moderation
Marketplace
Notification
Event
Operational
```

---

# 🧩 Reusable UI Component Architecture

Reusable UI components follow a consistent structure:

```text
Component
│
├── ViewComponent
├── Defaults
├── ViewModel
├── Builder
├── Default.cshtml
├── CSS
├── JavaScript
└── Documentation
```

### Responsibility Boundary

```text
Builder
    → Configures ViewModel

ViewComponent
    → Renders

ViewModel
    → Carries server-provided UI state

JavaScript
    → Interaction + lifecycle + minimal client state
```

The architecture follows:

> **Server state → ViewModel → Rendered UI → JavaScript enhancement**

---

# 🟨 JavaScript Architecture

Feature JavaScript uses the shared module foundation.

The architecture includes:

```text
App.Modules.register
App.Events
Utilities
```

Modules use explicit lifecycle management.

Where appropriate, components use:

* `WeakMap`
* Frozen configuration/constants
* `init`
* `bind`
* `destroy`

`site.js` remains a **foundation layer**.

Feature-specific:

* API workflows
* Business rules
* Component-specific domain logic

do not belong in `site.js`.

---

# 📡 API Architecture

APIs use thin controllers and DTO-based boundaries.

A typical flow is:

```text
HTTP Request
      │
      ▼
Controller
      │
      ▼
DTO
      │
      ▼
Application Service
      │
      ▼
Domain / Business Service
      │
      ▼
Persistence
```

Authenticated actor identity is determined server-side.

API contracts define:

* Route
* HTTP method
* Request DTO
* Response model
* Authorization
* Validation
* Ownership
* Error behavior

The API layer must not duplicate business rules already owned by application/domain services.

---

# ❗ Error Handling & Operation Results

SocialConnect uses consistent application outcome semantics across MVC and API workflows.

Conceptually, operations distinguish:

```text
Success
Validation Failure
Authentication Failure
Authorization Failure
Not Found
Conflict
Business Rule Failure
Unexpected Failure
```

Unexpected technical failures must pass through the global error-handling pipeline.

User-facing messages must be safe and localizable.

Internal details such as:

* Stack traces
* Database details
* Provider credentials
* Internal implementation details

must not be exposed to end users.

Correlation/trace information should be available for diagnostics.

---

# 🔁 Reliability & Retry Ownership

Retries are owned by the subsystem responsible for the failure.

```text
Business Workflow Retry
        │
        ▼
Event / Outbox Retry
        │
        ▼
Notification Delivery Retry
        │
        ▼
External Sink Retry
        │
        ▼
Worker Recovery
        │
        ▼
Alert Recovery
```

There is intentionally **no universal retry/self-healing service**.

Idempotency must be considered whenever retries can repeat an operation.

---

# 📊 Observability

Observability is treated separately from audit and alerting.

The platform supports:

* Structured logging
* Correlation IDs
* Metrics
* Tracing
* Operational diagnostics
* Worker health
* Application health

The distinction is:

```text
Logs
  → What happened?

Metrics
  → How much / how often?

Traces
  → Where did the operation travel?

Audit
  → Who changed what?

Worker Health
  → Is the worker operating?

Alerting
  → Does an operational condition require attention?
```

These mechanisms complement one another and do not replace each other.

---

# 🧪 Testing Strategy

Every major module should be validated against its documented contract.

Testing may include:

* Unit tests
* Domain tests
* Application workflow tests
* Integration tests
* Persistence tests
* API tests
* UI/component tests
* Event/Outbox tests
* Notification tests
* Feed tests
* Social Graph tests
* Administration tests
* End-to-end tests

Important cross-cutting workflows should be tested vertically.

For example:

```text
Business Operation
      ↓
State Change
      ↓
Event
      ↓
Outbox
      ↓
Dispatcher
      ↓
Handler
      ↓
Recipient / Target Resolution
      ↓
Notification / Feed / Other Effect
      ↓
Persistence
      ↓
Delivery
```

---

# 📚 Documentation Architecture

SocialConnect documentation is organized into multiple levels.

```text
SocialConnect
│
├── Project Definition
│
├── High-Level Cross-Cutting Contract
│
├── Software Requirements Specification
│
├── System Architecture
│
├── Detailed System Design
│
└── Module Contracts
    │
    ├── Identity
    ├── User Profile
    ├── Social Content
    ├── Feed
    ├── Social Graph
    ├── Administration
    ├── Media
    ├── Events
    ├── Notifications
    ├── Alerting
    └── Marketplace / Commerce
```

The documentation hierarchy is intentional.

---

# 📘 Requirements Documentation

## Project Definition

* [`Project Overview`](docs/requirements/Project-Overview.md)
* [`Software Requirements Specification`](docs/requirements/SRS.md)
* [`Product-Level Commerce SRS`](docs/requirements/CommerceSRS.md)
* [`SocialConnect Feasibility & Business Model`](docs/requirements/Market-Value-and-Business-Model.md)

---

# 🏛️ System Architecture Documentation

These documents describe the system at project level.

* [`High-Level Cross-Cutting Architecture Contract & System Requirements`](docs/architecture/High-Level-Cross-Cutting-Architecture-Contract.md)
* [`System Architecture`](docs/architecture/Architecture.md)
* [`Detailed System Design`](docs/architecture/Detailed-Design.md)
* [`Domain Model Architecture`](docs/architecture/DomainModelsArch.md)

The **High-Level Cross-Cutting Architecture Contract** establishes system-wide rules that apply across modules.

The **System Architecture** describes the overall structural architecture.

The **Detailed System Design** translates those principles into engineering-level structures.

The **Domain Model Architecture** documents the shared domain model and foundational entity architecture.

---

# 🧩 Module Documentation

## 👤 Application User & Identity

* [`Application User and Identity Model`](docs/Modules/ApplicationUser/AppUserModel.md)

---

## 👤 User Profile

* [`User Profile SRS`](docs/Modules/UserProfile/UserProfileSRS.md)

---

## 📰 Social Media

* [`Social Media Functional Requirements`](docs/Modules/SocialMedia/SocialMediaFunctionalReq.md)

This documentation covers the social-content requirements and the relationship between Posts, Comments, Reactions, Sharing, Media, and Feed.

---

## 📰 Feed

* [`Feed Engine Requirements`](docs/Modules/Feed/Feed-Engine-Requirements.md)

This document defines the Feed candidate-generation, eligibility, distribution, ranking, timeline, pagination, rendering, rebuild, and integration architecture.

---

## 🧑‍🤝‍🧑 Social Graph

* [`Social Graph Architecture`](docs/Modules/SocialGraph/Social-Graph-Architecture.md)

This document defines the architecture for:

* Friendship
* Following
* Blocking
* Groups
* Communities
* Social Graph query/context consumption
* Feed integration
* Administrative consumption
* Cross-module blocking policy

---

## 🛡️ Administration

* [`Administration Panel SRS`](docs/Modules/Administration/Admin-SRS.md)

The Administration module covers:

* Governance
* Moderation
* Reporting
* Review/Case management
* Decisions
* Actions
* Marketplace governance
* Configuration
* Maintenance Mode
* Administrative authorization

### Architecture

* [`Administration Architecture`](docs/architecture/Administration-Architecture.md)

---

# 📎 Cross-Cutting Reference Documentation

The following capabilities have architectural documentation or established contracts that support the modules above:

* Media
* Event Processing
* Notification Processing
* Recipient Resolution
* Operational Alerting
* Worker Health
* Reusable UI Components
* JavaScript Module Architecture
* External Provider Adapters
* Persistence conventions
* Security conventions

Not every cross-cutting capability requires a separate public document.

Where a capability is already sufficiently defined inside a canonical architecture or module contract, that document remains authoritative.

---

# 🔒 Internal / Reference-Only Architecture

Some architectural work has already been completed for future implementation but is **not currently being released as part of the public project documentation index**.

In particular:

* Event & Notification implementation/reference architecture
* Alerting implementation/reference architecture
* Internal event-handler implementation planning
* Internal operational reliability design

These documents remain important architectural references during development but are intentionally not linked from the public README until the corresponding documentation/release stage is approved.

This prevents the GitHub landing page from presenting internal implementation work as a released project contract.

---

# 🔌 External Integrations

External integrations follow:

```text
Domain / Application
        │
        ▼
Internal Contract
        │
        ▼
Adapter
        │
        ▼
External Provider
```

Provider-specific implementation details are intentionally isolated.

The architecture does not currently depend on a particular external payment, storage, messaging, or infrastructure provider.

Historical exploratory technologies such as:

* Microservices
* JWT-based replacement authentication
* ML.NET ranking
* Redis as a required infrastructure dependency
* Nginx as a required application component
* Azure Blob Storage as a mandatory media store
* Specific payment providers
* Signal Protocol/AES as a messaging architecture
* Cloud migration

are **not current baseline architectural requirements** unless explicitly reintroduced through a future architectural decision.

---

# 🗄️ Database Documentation

Database documentation will describe finalized persistence contracts.

Topics include:

* Entity relationships
* Keys
* Constraints
* Indexes
* Ownership
* Transactions
* Concurrency
* Soft deletion
* Auditing
* Event/Outbox persistence
* Module-specific persistence

Database design must remain aligned with authoritative domain ownership.

---

# 🌐 API Documentation

Finalized API contracts will be maintained separately.

API documentation includes:

* Routes
* HTTP methods
* Request DTOs
* Response models
* Authorization
* Validation
* Ownership
* Error behavior
* Integration requirements

Only finalized contracts should be treated as authoritative.

---

# 🧪 Testing Documentation

Project-wide testing documentation will be maintained under:

```text
docs/testing/
```

The primary project-level document is ( Note : this document is still missing):

* [`Testing Strategy`](docs/testing/Testing-Strategy.md)

Module-specific testing documentation should remain close to the module it validates.

---

# 📐 Architecture Decision & Change Governance

SocialConnect treats architecture as an evolving but controlled engineering artifact.

A change to a locked contract should not be introduced implicitly through implementation.

The expected lifecycle is:

```text
Problem / New Requirement
          │
          ▼
Impact Analysis
          │
          ▼
Architecture Review
          │
          ▼
Contract Revision
          │
          ▼
Implementation
          │
          ▼
Testing
          │
          ▼
Documentation Update
```

A developer or implementation assistant must not silently redesign an already-locked architecture merely because another implementation approach appears simpler.

---

# 🔗 Requirement Traceability

Important requirements should remain traceable through:

```text
Requirement
     │
     ▼
Architecture
     │
     ▼
Module Contract
     │
     ▼
Implementation
     │
     ▼
Test
     │
     ▼
Documentation
```

The purpose is to make it possible to determine:

* Why a capability exists
* Which module owns it
* Which contract defines it
* Where it is implemented
* How it is tested
* Which downstream systems depend on it

---

# 📋 Documentation Status

SocialConnect documentation uses the following status model:

| Status          | Meaning                                                                    |
| --------------- | -------------------------------------------------------------------------- |
| **Draft**       | Under analysis and subject to change                                       |
| **Reviewed**    | Technically reviewed but not yet locked                                    |
| **Locked**      | Approved architectural/implementation contract                             |
| **Implemented** | Corresponding functionality implemented                                    |
| **Validated**   | Implementation tested against its documented contract                      |
| **Reference**   | Internal/supporting architecture not currently part of the public contract |
| **Superseded**  | Replaced by a newer authoritative document                                 |

There should be **one authoritative document for each contract**.

Competing versions of the same architectural contract should not be maintained.

---

# 🗺️ Development Lifecycle

SocialConnect follows a deliberate engineering lifecycle:

```text
Product Requirement
        │
        ▼
Requirements Analysis
        │
        ▼
Architecture
        │
        ▼
Module Boundary
        │
        ▼
Detailed Design
        │
        ▼
Implementation Contract
        │
        ▼
Implementation
        │
        ▼
Compilation / Smoke Validation
        │
        ▼
Automated Testing
        │
        ▼
Integration Validation
        │
        ▼
Documentation
        │
        ▼
Architecture Review
```

Major modules should be finalized progressively rather than implementing large amounts of functionality without an established contract.

---

# 🧭 Repository Documentation Map

| Location             | Purpose                                       |
| -------------------- | --------------------------------------------- |
| `README.md`          | GitHub project and documentation landing page |
| `docs/requirements/` | Product and system requirements               |
| `docs/architecture/` | System-wide and cross-cutting architecture    |
| `docs/Modules/`      | Module-specific requirements and architecture |
| `docs/engineering/`  | Engineering implementation conventions        |
| `docs/api/`          | Finalized API contracts                       |
| `docs/testing/`      | Testing strategy and validation               |
| `docs/database/`     | Database and persistence documentation        |

---

# 🖼️ Architecture & Design Diagrams

The repository contains visual architecture and design artifacts where appropriate.

Current diagrams include:

* [`Complete Blueprint`](completeblueprint.png)
* [`Complete Overview Blueprint / UML`](completeoverviewblueprintuml.png)
* [`SocialConnect Class Diagram`](socialconnectclassdiagram.png)

Additional diagrams will be added as their corresponding architecture is finalized.

---

# 📚 Recommended Documentation Entry Points

For someone discovering SocialConnect for the first time, the recommended reading order is:

```text
1. README.md
      │
      ▼
2. Project Overview
      │
      ▼
3. High-Level Cross-Cutting Architecture Contract
      │
      ▼
4. Software Requirements Specification
      │
      ▼
5. System Architecture
      │
      ▼
6. Detailed System Design
      │
      ▼
7. Module Contracts
      │
      ├── Identity
      ├── User Profile
      ├── Social Media
      ├── Feed
      ├── Social Graph
      └── Administration
```

This provides a progression from:

> **What SocialConnect is → How it is architected → How each module works.**

---

# 🚀 Current Architectural Direction

The current SocialConnect architecture is centered around the following principle:

> **Business domains own business meaning. Cross-cutting infrastructure provides reusable capabilities without taking ownership of those domains.**

This principle applies throughout the system.

Examples:

```text
Post
 └── owns post content

Feed
 └── distributes/displays post-derived content

Reaction
 └── owns reaction state

Notification
 └── owns notification persistence and delivery planning

SignalR
 └── provides real-time transport

Event / Outbox
 └── provides durable event processing

Alerting
 └── evaluates operational conditions

Worker Health
 └── reports worker operational evidence

Administration
 └── governs through domain/application contracts

Media
 └── owns media lifecycle

Social Graph
 └── owns relationships

Marketplace
 └── owns commerce
```

This prevents cross-cutting systems from becoming hidden business owners.

---

# 🏁 Architectural Definition of Done

A major SocialConnect module should not be considered architecturally complete until its important responsibilities have been addressed.

At minimum, the module should establish:

* Purpose and scope
* Domain ownership
* Entities/value objects where applicable
* Business rules
* Application services
* DTO contracts
* Validation
* Authorization
* Persistence requirements
* Transaction boundaries
* Loader/lookup/resolver requirements
* Event behavior where applicable
* Notification behavior where applicable
* Feed/discovery impact where applicable
* Media integration where applicable
* Alert/operational impact where applicable
* API contracts
* UI/component contracts
* JavaScript boundaries
* Error handling
* Idempotency/concurrency where applicable
* Testing requirements
* Documentation
* Traceability

Not every module requires every item, but each omission should be deliberate.

---

# 📖 Documentation Philosophy

SocialConnect documentation is treated as part of the engineering system.

The objective is not simply to document what was coded.

The objective is to preserve:

```text
Why
│
├── Why the module exists
├── Why ownership is defined this way
├── Why boundaries exist
├── Why workflows are structured this way
└── Why architectural decisions were made
      │
      ▼
How
│
├── How the module is structured
├── How requests flow
├── How state changes
├── How events propagate
├── How UI interacts
└── How the system is tested
```

This allows SocialConnect to remain understandable as the codebase grows.

---

# 👨‍💻 Author

**Khalid Zada**

Computer Engineer | IT & Systems | Software Development | Infrastructure & Troubleshooting

GitHub: [@khalidzada](https://github.com/khalidzada)

---

# 📌 Repository Navigation

| Location             | Purpose                                              |
| -------------------- | ---------------------------------------------------- |
| `README.md`          | Primary project and documentation landing page       |
| `docs/requirements/` | Requirements and product definition                  |
| `docs/architecture/` | System-wide architecture and cross-cutting contracts |
| `docs/Modules/`      | Module-specific requirements and architecture        |
| `docs/engineering/`  | Engineering and implementation documentation         |
| `docs/api/`          | Finalized API contracts                              |
| `docs/testing/`      | Testing and validation documentation                 |
| `docs/database/`     | Database and persistence documentation               |
| `*.png`              | Architecture and design diagrams                     |

---

# 📌 Final Project Statement

> **SocialConnect is a continuously evolving engineering project designed around explicit ownership, modular architecture, reusable infrastructure, server authority, traceable contracts, and production-oriented engineering practices.**
>
> **The repository documentation evolves together with the architecture and implementation so that requirements, architectural decisions, module contracts, workflows, testing strategies, and engineering practices remain visible, traceable, and understandable.**

---

## SocialConnect Documentation Principle

```text
                     SOCIALCONNECT
                           │
             ┌─────────────┴─────────────┐
             │                           │
       Product Definition          Architecture
             │                           │
             ▼                           ▼
          SRS / Vision        Cross-Cutting Contract
                                         │
                                         ▼
                               System Architecture
                                         │
                                         ▼
                                Detailed System Design
                                         │
                         ┌───────────────┼───────────────┐
                         │               │               │
                         ▼               ▼               ▼
                     Identity       Social Media     Social Graph
                         │               │               │
                         ▼               ▼               ▼
                      Profile          Feed        Administration
                                         │
                                         ▼
                              Implementation Contracts
                                         │
                                         ▼
                                  Implementation
                                         │
                                         ▼
                                  Testing / Validation
                                         │
                                         ▼
                                  Living Documentation
```

**One system. One authoritative contract per responsibility. Explicit ownership. Traceable implementation.**
