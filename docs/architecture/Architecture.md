# SocialConnect — System Architecture

**Document Status:** Canonical
**Version:** 1.0
**Scope:** System-wide architecture
**Technology Baseline:** .NET 8 / ASP.NET Core MVC
**Architecture Style:** Layered Modular Monolith

---

## 1. Purpose

This document defines the current system architecture of **SocialConnect**.

It establishes the major architectural layers, application boundaries, domain ownership rules, infrastructure boundaries, reusable component model, cross-cutting capabilities, and communication patterns used throughout the platform.

The purpose of this document is to provide a stable architectural reference for future implementation and module-specific design.

Detailed module requirements and implementation contracts are documented separately.

---

# 2. System Scope

SocialConnect is a hybrid social-network and e-commerce platform designed to combine social interaction, content discovery, marketplace capabilities, and commerce within a single application.

The system is being developed as a **modular monolith**, allowing strong internal separation between application domains while retaining a single deployable ASP.NET Core application.

The major architectural domains include:

* Identity and User Profiles
* Social Content
* Posts
* Post Creation
* Feed and Discovery
* Comments
* Reactions
* Sharing
* Media
* Marketplace
* Products
* Shops
* Orders
* Payments
* Events
* Notifications
* Alerting
* Administration
* Moderation
* Reporting
* Auditing
* Localization and Geographic Context

Not all domains are at the same implementation stage. Module documentation defines the implementation status of individual domains.

---

# 3. Architectural Style

## 3.1 Layered Modular Monolith

The current system uses a **layered modular monolith**.

```text
┌─────────────────────────────────────────────┐
│                 Presentation                │
│       MVC Controllers / ViewComponents      │
│              Razor / JavaScript             │
├─────────────────────────────────────────────┤
│                 Application                 │
│     Use Cases / Orchestration / Contexts    │
│       Loaders / Resolvers / Policies        │
├─────────────────────────────────────────────┤
│              Business / Domain              │
│       Domain Rules / Business Services      │
├─────────────────────────────────────────────┤
│              Infrastructure                 │
│ EF Core / SQL Server / Identity / Adapters  │
└─────────────────────────────────────────────┘
```

The modular structure provides logical boundaries without requiring separate deployable services.

Microservices are **not** the current architecture.

Future extraction of an individual module into a separate service would require a deliberate architectural decision and is not assumed by the current design.

---

# 4. Architectural Layers

## 4.1 Presentation Layer

The presentation layer contains:

* MVC Controllers
* Razor Views
* Razor ViewComponents
* ViewModels
* Client-side JavaScript
* CSS
* Bootstrap-based UI

Presentation components are responsible for presentation and interaction.

They do not own application business workflows.

---

## 4.2 Application Layer

The application layer coordinates use cases and workflows.

Typical responsibilities include:

* Application services
* Context builders
* Orchestrators
* Entity loaders
* Tracked entity loaders
* Resolvers
* Factories
* Policies
* DTO handling
* ViewModel construction
* Workflow coordination

The application layer determines **how a use case is coordinated**, while business services own business rules.

---

## 4.3 Business / Domain Layer

The business/domain layer contains:

* Domain entities
* Business rules
* Domain-oriented services
* Domain events
* Domain concepts
* Ownership rules

Business services should not directly depend on infrastructure repositories.

Infrastructure access is reached through appropriate abstractions.

---

## 4.4 Infrastructure Layer

Infrastructure contains technical implementations such as:

* Entity Framework Core
* SQL Server access
* Generic Repository
* Unit of Work
* ASP.NET Core Identity infrastructure
* File/media storage
* External integration adapters
* Background processing infrastructure
* Other framework/provider-specific implementations

Infrastructure details must not leak into domain contracts.

---

# 5. Dependency Direction

The preferred dependency direction is:

```text
Application Feature
        │
        ▼
Reusable Component / Application Contract
        │
        ▼
Infrastructure
        │
        ▼
Framework / External Provider
```

Higher-level business concepts should not become tightly coupled to provider-specific implementations.

---

# 6. Core Ownership Rule

SocialConnect follows a strict ownership principle:

> **A reusable component owns everything within its domain. Consumers configure it, call its public API, subscribe to its events, and react to its results. Consumers never implement or control its internal behavior.**

This applies to both backend services and reusable UI components.

A consumer may:

* Configure a component
* Invoke its public contract
* Subscribe to documented events
* React to results

A consumer must not:

* Manipulate internal state
* Reimplement internal workflows
* Depend on internal implementation details
* Bypass the component's public contract

---

# 7. Controllers

Controllers are intentionally thin.

A controller should primarily:

1. Receive the request.
2. Validate/request-bind the input as appropriate.
3. Invoke the appropriate application contract.
4. Return the appropriate response.

Business workflows should not be implemented inside controllers.

Repository access should not be placed directly inside controllers.

---

# 8. Application Orchestration

Complex application workflows are coordinated by application-level orchestration.

A typical workflow may contain:

```text
Request
   │
   ▼
DTO
   │
   ▼
Context Builder
   │
   ├── Loaders
   ├── Resolvers
   ├── Policies
   └── Factories
   │
   ▼
Application Service / Orchestrator
   │
   ▼
Business Services
   │
   ▼
Persistence / Infrastructure
```

Large services should be decomposed rather than accumulating unrelated responsibilities.

---

# 9. Entity Loading

Business services should not directly depend on repositories where a loader abstraction is appropriate.

The architecture uses loader contracts such as:

```text
IEntityLoader<TEntity,TKey>
ITrackedEntityLoader<TEntity,TKey>
```

The distinction allows:

* Read operations to remain no-tracking.
* Update workflows to explicitly request tracked entities.

Lookup services act as public read gateways where appropriate.

---

# 10. Persistence Architecture

SocialConnect uses:

* Entity Framework Core
* Microsoft SQL Server
* Generic Repository
* Unit of Work

The persistence layer is responsible for database interaction.

The generic repository provides read operations using no-tracking semantics where appropriate, while explicit `ForUpdate` operations provide tracked entities for modification workflows.

Database concerns should remain within persistence/infrastructure boundaries.

---

# 11. Identity Architecture

ASP.NET Core Identity provides the platform's identity foundation.

The architecture separates Identity concerns from business profile information.

```text
ASP.NET Core Identity
        │
        ▼
ApplicationUser
        │
        │ Identity concerns
        │
        ▼
UserProfile
        │
        │ Business profile data
        ▼
Application Domain
```

`ApplicationUser` remains focused on Identity.

Business profile information belongs to `UserProfile` and related application-domain structures.

External OAuth providers and MFA are future/pending capabilities rather than assumptions of the current authentication implementation.

---

# 12. Target Resolution

SocialConnect uses a generic target mechanism where supported features operate against different entity types.

The fundamental representation is:

```text
TargetType + TargetId
```

This mechanism is used where features such as reactions, comments, ownership resolution, and other target-oriented operations need to identify a business object.

Target resolution must remain explicit and type-safe at the appropriate application boundary.

---

# 13. Media Architecture

Media is implemented as a dedicated subsystem.

The canonical processing pipeline is:

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
uploads/temp/
      │
      ▼
Media Domain
      │
      ▼
Media IDs
      │
      ├───────────────┐
      ▼               ▼
Finalization      Assignment
      │               │
      ▼               ▼
Final Storage     Entity Association
```

### Ownership rules

`MediaUploadService` is the upload gateway.

Finalization performs temporary-to-final storage movement.

Assignment associates already-finalized media with its owner.

Finalization must not perform assignment.

Assignment must not perform finalization.

---

# 14. Post Creation Architecture

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
      └── Publish Feed
      │
      ▼
PostCreationResult
```

Post creation orchestration does not make `PostService` the owner of the complete workflow.

The workflow owns coordination between post creation, media processing, assignment, and feed publication.

---

# 15. Comments

Comments are implemented as a target-oriented subsystem.

The reusable `CommentManager` provides the UI component boundary.

The current component supports concepts including:

* Creation
* Replies
* Editing
* Collapsed/expanded presentation
* Lazy attachment
* Reaction integration
* Target ownership
* Controlled reply depth

The comment subsystem owns its own internal UI state and behavior.

Consumers configure and interact with its public contract.

---

# 16. Reactions

Reactions operate through the target mechanism.

Supported reaction types currently include:

```text
Like
Love
Haha
Wow
Sad
Angry
```

The reaction mechanism is designed for reuse across supported target types rather than implementing separate reaction workflows for each entity.

---

# 17. Sharing

Internal sharing is represented by a relationship between posts.

```text
Original Post
      │
      ▼
SharedPostId
      │
      ▼
New Post
```

The shared post does not duplicate the original content or media.

The database relationship uses restricted deletion semantics where required to protect the shared relationship.

The feed/application layer resolves shared content.

Shared graph traversal must support maximum-depth and cycle protection.

---

# 18. Feed & Discovery

Feed functionality is treated as an application/discovery concern rather than being embedded into unrelated domain services.

The platform is intended to support:

* Social content
* Marketplace content
* Geographic relevance
* Discovery
* Shared posts
* Future recommendation capabilities

Advanced recommendation/ML functionality remains a future extensibility direction and is not treated as a required current implementation technology.

---

# 19. Event Architecture

SocialConnect distinguishes between:

```text
Domain Events
```

and:

```text
Integration / Outbox Events
```

A domain event represents a business fact.

An integration/outbox event represents reliable asynchronous processing across application boundaries.

The event architecture uses a transactional outbox.

```text
Business Operation
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
```

The outbox architecture supports concepts including:

* Status
* Retry handling
* Leasing
* Claiming
* Dead-letter handling
* Idempotency
* Reliable persistence
* SQL Server-safe worker processing

---

# 20. Notification Architecture

Notification persistence is the source of truth for notifications.

The processing model is:

```text
Event
  │
  ▼
Event Handler
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

Real-time delivery technologies such as SignalR are delivery mechanisms only.

They do not replace persisted notification state.

---

# 21. Alerting Architecture

Alerting is implemented as a generic operational capability.

The architecture deliberately avoids a central service containing a large hard-coded switch for every possible alert source.

Instead:

```text
Operational Module
       │
       ▼
Alert Source / Policy
       │
       ▼
Generic Alert Platform
       │
       ▼
Alert Processing
       │
       ▼
Configured Delivery
```

Future sources may originate from:

* Posts
* Comments
* Reactions
* Marketplace
* Orders
* Payments
* Media
* Administration
* Other modules

The generic alerting platform remains stable while individual modules integrate through the alert source/policy boundary.

---

# 22. Administration Architecture

Administration is a privileged subsystem inside the existing MVC application.

ASP.NET Core Areas are not used for Administration.

Administrative functionality uses application/domain services and respects domain ownership.

The Administration domain includes:

* Governance
* Moderation
* Reporting
* Review/case management
* Decisions
* Actions
* Marketplace governance
* Notification policy
* Event policy
* Operational configuration
* Maintenance mode
* Auditing

Reporting and moderation remain separate concepts:

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

---

# 23. External Integration Boundary

Every external provider integration terminates at an Infrastructure Adapter Boundary.

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

Application/domain code must not depend directly on:

* Provider SDKs
* Provider endpoints
* Provider authentication mechanisms
* Provider request models
* Provider response models

This allows external providers to change without propagating provider-specific concerns throughout the application.

---

# 24. Reusable UI Architecture

Reusable UI components follow:

```text
ViewComponent
      │
      ├── Defaults
      ├── ViewModel
      ├── Builder
      ├── Default.cshtml
      ├── CSS
      ├── JavaScript
      └── Documentation
```

The ViewComponent renders.

The Builder configures.

The ViewModel represents server-rendered UI state.

JavaScript manages interaction and minimal client state.

---

# 25. JavaScript Architecture

Feature JavaScript modules use the application module infrastructure.

The general lifecycle is:

```text
register
   │
   ▼
init
   │
   ▼
bind
   │
   ▼
destroy
```

Where appropriate, modules use:

* `App.Modules.register`
* WeakMap instance tracking
* Immutable configuration
* Immutable constants
* Explicit lifecycle management

The global `site.js` file remains a foundation layer.

Feature API/business logic does not belong in `site.js`.

---

# 26. Configuration and Constants

Administrative configuration represents policy that may be changed by authorized administrators.

Immutable domain constants remain code/domain contracts.

These concepts must not be conflated.

For example:

```text
Admin-configurable policy
        ≠
Immutable domain constant
```

This distinction applies across Notification, Event, Moderation, Marketplace, Security, and Operational configuration.

---

# 27. Localization and Geographic Architecture

Geographic context is treated as a shared capability across Social and Marketplace domains.

The platform is intended to support:

* Country
* City
* Local discovery
* Regional discovery
* Global discovery
* Locale-aware presentation
* Future language localization
* Future currency handling
* Future regional commerce rules

The architecture should allow geographic context to be shared without making Social or Marketplace domains dependent on each other's internal implementation.

---

# 28. Architecture Quality Goals

The architecture prioritizes:

* Maintainability
* Clear ownership
* Testability
* Explicit contracts
* Separation of concerns
* Reusability
* Extensibility
* Traceability
* Controlled complexity
* Secure integration boundaries

Scalability is addressed through modularity, asynchronous processing where appropriate, efficient persistence patterns, and clear infrastructure boundaries rather than assuming a microservices architecture.

---

# 29. Architectural Evolution

The architecture is intentionally evolutionary.

A module may become more sophisticated as its requirements mature.

However:

> Existing locked contracts should not be redesigned merely for stylistic reasons.

Architectural changes should be introduced when:

* A genuine requirement appears.
* An existing contract is contradictory.
* A security or correctness problem is discovered.
* A scalability or maintainability issue is demonstrated.
* A new module requires an explicit architectural extension.

All significant changes should be documented.

---

# 30. Related Documentation

This document defines the system-wide architectural baseline.

Related documentation includes:

* [Project Overview](../requirements/Project-Overview.md)
* [Software Requirements Specification](../requirements/SRS.md)
* [Detailed Design](Detailed-Design.md)
* Event & Notification Architecture
* Alerting Architecture
* Administration Architecture
* Module-specific architecture and implementation documents

Module documentation provides the detailed contract for individual domains.

---

## Document Status

**Canonical system architecture for the current SocialConnect implementation direction.**

Module-specific documents may provide additional detail, but they must remain consistent with the architectural boundaries established here.
