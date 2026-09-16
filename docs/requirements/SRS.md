# SocialConnect

## Software Requirements Specification (SRS)

**Document Status:** Canonical
**Document Version:** 2.0
**Project:** SocialConnect
**Architecture Status:** Living Engineering Contract
**Last Major Architectural Baseline:** 2026

---

# 1. Introduction

## 1.1 Purpose

This Software Requirements Specification defines the functional, architectural, technical, and non-functional requirements for SocialConnect.

It supersedes the assumptions contained in the original SocialConnect+ SRS and early detailed design documents where those documents conflict with the current implementation and locked architecture.

This document establishes the current system contract for:

* system scope;
* architectural principles;
* domain boundaries;
* application responsibilities;
* reusable component architecture;
* media processing;
* social interaction;
* post creation;
* sharing;
* reactions;
* comments;
* events;
* notifications;
* recipient resolution;
* alerting;
* administration;
* infrastructure boundaries;
* testing and extensibility.

This is a **living SRS**.

It must evolve when a new architectural decision is formally locked.

---

# 2. Product Scope

## 2.1 Product Definition

SocialConnect is a hybrid social networking and marketplace platform.

The platform combines:

* social identity;
* profiles;
* social content;
* engagement;
* media;
* notifications;
* discovery;
* marketplace capabilities;
* local and future global commerce.

---

# 3. Scope Boundaries

## 3.1 Current Engineering Scope

The current engineering scope includes the platform architecture and implemented/locked foundations for:

* Identity and Profiles;
* Posts;
* Post Creation;
* Media;
* Comments;
* Reactions;
* Sharing;
* Feed/TImeline;
* Events;
* Notifications;
* Recipient Resolution;
* Alerting;
* Administration architecture;
* reusable UI components;
* application infrastructure.

## 3.2 Future Scope

The product roadmap may introduce:

* marketplace shops;
* products;
* listings;
* orders;
* payments;
* seller analytics;
* advertising;
* subscriptions;
* advanced recommendation systems;
* international marketplace functionality;
* additional localization;
* advanced communication features.

These must not be documented as implemented until their corresponding implementation contracts are finalized.

---

# 4. Architectural Requirements

## AR-001 — Modular Architecture

The system shall use a modular architecture with explicit ownership boundaries.

Modules shall not depend on implementation details belonging to another module.

---

## AR-002 — Layered Responsibility

The application shall maintain clear responsibility boundaries:

```text
Controller
    ↓
Application / Orchestration
    ↓
Domain / Business Services
    ↓
Persistence / Infrastructure
```

The exact implementation may use specialized builders, loaders, resolvers, factories, policies, and adapters where required.

---

## AR-003 — Controller Responsibility

Controllers shall:

* receive requests;
* perform boundary handling;
* invoke appropriate application services;
* return responses.

Controllers shall not contain domain business workflows.

---

## AR-004 — Service Responsibility

Services shall own business rules appropriate to their domain.

Large services shall be decomposed into smaller services.

God services are prohibited.

---

## AR-005 — Repository Responsibility

Repositories shall contain persistence operations only.

Business rules shall not be implemented inside repositories.

---

## AR-006 — DbContext Responsibility

The Entity Framework DbContext shall remain an infrastructure concern.

Business logic shall not be coupled directly to DbContext operations.

---

## AR-007 — DTO Responsibility

DTOs shall represent application/service transport contracts.

Entities shall not be exposed merely because they are convenient transport objects.

---

## AR-008 — ViewModel Responsibility

ViewModels shall represent UI rendering state.

Business rules shall not be embedded in ViewModels.

---

## AR-009 — Mapping

Entity-to-DTO and DTO-to-Entity/ViewModel mapping shall be explicit and controlled.

**AutoMapper is not part of the current architecture.**

---

# 5. Dependency Requirements

## AR-010 — Dependency Direction

Dependencies shall flow toward lower-level technical concerns without allowing infrastructure implementation details to leak upward.

Reusable components must not depend on their consumers.

---

## AR-011 — Public Contract Boundary

Modules and reusable components shall communicate through stable public contracts.

Internal implementation details shall remain private.

---

## AR-012 — External Adapter Boundary

External APIs/providers shall be isolated behind infrastructure adapter boundaries.

Application and domain services shall consume internal contracts rather than provider SDKs or provider-specific models.

---

# 6. Domain Ownership

## AR-013 — Domain Ownership

Every reusable component and domain service shall own its responsibility completely.

Ownership includes, where applicable:

* state;
* lifecycle;
* validation;
* UI behaviour;
* events;
* internal implementation.

Consumers shall:

* configure;
* invoke public APIs;
* subscribe to public events;
* consume results.

---

# 7. Reusable UI Requirements

## UI-001 — ViewComponent

Reusable UI components shall be implemented as ViewComponents.

Canonical structure:

```text
ViewComponent
↓
ViewModel
↓
Default.cshtml
↓
JavaScript
↓
CSS
```

---

## UI-002 — Builder

Reusable components shall use Builders for configuration where component configuration is non-trivial.

Builders shall produce complete component configuration rather than relying on scattered property assignment.

---

## UI-003 — Component API

Every reusable component shall expose a stable public API.

Consumers shall not access private component state.

---

## UI-004 — Component Events

Component events shall describe business or meaningful component facts.

Implementation details such as XHR internals or DOM implementation changes shall not become public business events.

---

## UI-005 — Component State Machines

Reusable components shall own their own finite state machines.

A consuming component shall not directly manipulate another component's internal state.

---

## UI-006 — Configuration over Duplication

New consumers shall configure existing reusable components whenever possible.

New feature requirements must not automatically result in a duplicated component.

---

## UI-007 — JavaScript Boundary

JavaScript shall:

* coordinate UI;
* invoke APIs;
* manage client state;
* emit/consume component events.

JavaScript shall not duplicate server business rules.

---

## UI-008 — Server Rendering

Razor shall remain responsible for primary HTML rendering.

JavaScript shall enhance the server-rendered UI rather than recreate entire application interfaces unnecessarily.

---

# 8. Media Requirements

## MED-001 — Upload Architecture

Media uploads shall follow:

```text
MediaUploader
↓
/api/media/upload-async
↓
MediaUploadService
↓
Temporary Storage
↓
Media Domain
↓
Finalization / Assignment
```

---

## MED-002 — Upload Responsibility

`MediaUploadService` shall act as the upload gateway.

Post, Comment, Product, or other feature services shall not duplicate media upload implementation.

---

## MED-003 — Temporary Media

Uploaded media shall initially enter temporary storage.

---

## MED-004 — Finalization

Media finalization shall move media from temporary storage to final storage.

Finalization shall not perform ownership assignment.

---

## MED-005 — Assignment

Media assignment shall operate only on finalized media.

---

## MED-006 — Media Ownership

The current media ownership model includes:

```text
User   = 1
Post   = 2
Product = 3
Shop   = 4
Media  = 5
Comment = 6
```

---

## MED-007 — Media Roles

Current defined roles include:

```text
PostMedia    = 6
CommentMedia = 7
```

---

## MED-008 — Media Types

Current media types include:

```text
Image    = 1
Video    = 2
Document = 3
```

---

## MED-009 — Media Layer Separation

The system shall enforce:

> Detection discovers facts.
> Validation verifies facts.
> Storage persists facts.

No validator shall discover facts.

No detector shall perform business validation.

No storage service shall perform validation.

---

# 9. Post Requirements

## POST-001 — Post Creation

Post creation shall use the application-level workflow:

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

---

## POST-002 — Post Service Ownership

A generic PostService shall not become responsible for the complete creation workflow.

The PostCreation workflow owns orchestration.

---

## POST-003 — Post Media

Posts may use the generic media architecture.

Post-specific code shall configure/use the media component rather than reimplementing it.

---

# 10. Comment Requirements

## COMMENT-001

Comments shall be treated as a distinct domain responsibility.

---

## COMMENT-002

CommentManager shall own its own:

* UI state;
* creation;
* reply;
* edit;
* saving;
* rendering interaction;
* component events.

---

## COMMENT-003

Comment replies shall support the currently defined reply-depth policy.

---

## COMMENT-004

Comment-related media shall use the generic MediaUploader architecture.

---

# 11. Reaction Requirements

## REACTION-001

Reaction functionality shall be implemented through a dedicated reaction responsibility.

---

## REACTION-002

The reaction system shall support the currently defined reaction types:

* Like;
* Love;
* Haha;
* Wow;
* Sad;
* Angry.

---

## REACTION-003

Reaction targeting shall use the platform target mechanism.

---

## REACTION-004

The current target mechanism shall be:

```text
TargetType + TargetId
```

This allows reusable target-based behaviour without hard-coding a separate implementation for every entity type.

---

# 12. Sharing Requirements

## SHARE-001

The sharing model shall distinguish:

```text
None
Internal
External
```

---

## SHARE-002 — Internal Sharing

Internal sharing shall create a new Post referencing the original Post.

The original content/media shall not be duplicated.

---

## SHARE-003

Internal sharing shall use `SharedPostId`.

---

## SHARE-004

The shared-post relationship shall use restricted deletion semantics.

---

## SHARE-005

Shared-post graph traversal shall protect against excessive recursion and cycles.

---

# 13. Feed Requirements

## FEED-001

The feed architecture shall support social content and future marketplace content without requiring each domain to implement feed behaviour itself.

---

## FEED-002

Feed composition shall be performed by dedicated feed/timeline orchestration.

---

## FEED-003

Shared content shall be loaded with maximum-depth and cycle-protection rules.

---

# 14. Event Architecture

## EVENT-001

The platform shall distinguish:

* Domain Events;
* Integration Events;
* Outbox Events.

---

## EVENT-002 — Transactional Outbox

Events requiring durable asynchronous processing shall use transactional outbox persistence.

The business operation and corresponding outbox persistence shall maintain transactional consistency.

---

## EVENT-003 — Dispatcher

The event dispatcher shall process persisted outbox events.

---

## EVENT-004 — Reliability

The outbox architecture shall support:

* processing status;
* retry;
* lease/claim behaviour;
* dead-letter handling;
* idempotency.

---

## EVENT-005 — SQL Server Safety

Event claiming and processing shall be safe for concurrent workers under SQL Server semantics.

---

# 15. Notification Requirements

## NOTIF-001

Notification persistence is the source of truth for notifications.

---

## NOTIF-002

Notification application services shall expose separate responsibilities for:

* querying;
* commands.

---

## NOTIF-003

Notification query operations shall support:

* authenticated-user ownership;
* deterministic pagination;
* unread count;
* expiration-aware visibility.

---

## NOTIF-004

Notification command operations shall support:

* mark read;
* mark unread;
* mark all read.

---

## NOTIF-005

Controllers shall remain thin and shall not contain notification business logic.

---

## NOTIF-006

Real-time mechanisms such as SignalR shall be treated as delivery mechanisms rather than the persistence source of truth.

---

# 16. Recipient Resolution

## REC-001

Recipient resolution shall be a dedicated architectural concern.

---

## REC-002

Candidate source responsibilities shall include:

### PostOwnerSource

Resolves the owner through:

```text
Post.UserId
```

### CommentOwnerSource

Resolves the owner through:

```text
Comment.UserId
```

### TargetOwnerSource

Resolves:

```text
TargetType + TargetId
        ↓
Target Entity
        ↓
Owner
```

### ParentAuthorSource

Resolves:

```text
Comment.ParentCommentId
        ↓
Parent Comment
        ↓
Parent Author
```

---

## REC-003

Candidate sources shall be independently composable through the candidate-source architecture.

Recipient resolution shall not become duplicated inside individual event handlers.

---

# 17. Alerting Requirements

## ALERT-001

Alerting shall be implemented as a generic platform.

---

## ALERT-002

Alerting shall not contain Notification-specific assumptions in its core infrastructure.

---

## ALERT-003

The Alert Source Catalog shall not become a giant hard-coded switch.

---

## ALERT-004

Operational conditions shall integrate through the Alert Source / Policy boundary.

---

## ALERT-005

The implementation sequence shall be:

```text
Generic Alerting Backend
        ↓
Verification
        ↓
Event/Notification operational evidence
        ↓
Additional Alert Sources
        ↓
Independent Vertical Slices
```

---

# 18. Administration Requirements

## ADMIN-001

Administration shall remain within the existing SocialConnect application.

---

## ADMIN-002

ASP.NET Core Areas shall not be introduced for Administration.

---

## ADMIN-003

The authentication gateway shall determine the authenticated user's permitted experience.

---

## ADMIN-004

The current privileged roles are:

* Admin;
* Moderator.

The platform also supports:

* Vendor;
* User.

There is no SuperAdmin role.

---

## ADMIN-005

Administrators and moderators shall use domain/application services.

They shall not bypass domain ownership through direct table manipulation.

---

## ADMIN-006

Reporting and moderation shall remain distinct concepts.

The moderation flow is conceptually:

```text
Report
 ↓
Review / Case
 ↓
Decision
 ↓
Action
```

---

## ADMIN-007

Administration configuration shall distinguish:

* Platform;
* Security;
* Moderation;
* Marketplace;
* Notification;
* Event;
* Operational.

---

## ADMIN-008

Configurable policy shall not be confused with immutable domain constants.

---

## ADMIN-009

Notification channel definitions are platform contracts.

Administration may enable/disable supported channels but shall not redefine the channel contract through ordinary configuration.

---

# 19. Security Requirements

## SEC-001

Authentication and authorization shall be handled through the platform's identity infrastructure.

---

## SEC-002

Authorization shall be enforced at appropriate application/domain boundaries.

---

## SEC-003

Business operations shall not rely solely on UI-level access restrictions.

---

## SEC-004

External provider credentials and provider-specific mechanisms shall remain within infrastructure adapter boundaries.

---

## SEC-005

Security-sensitive operations shall produce appropriate audit information at the application/service boundary where required.

---

# 20. Data Requirements

## DATA-001

SQL Server is the current relational persistence platform.

---

## DATA-002

Entity Framework Core is the current ORM/persistence technology.

---

## DATA-003

The database design shall maintain referential integrity through appropriate relationships and constraints.

---

## DATA-004

Soft-delete and audit conventions shall be applied according to domain requirements.

---

## DATA-005

Repositories shall not expose business logic.

---

# 21. Identity Requirements

## ID-001

`ApplicationUser` is primarily an Identity concern.

---

## ID-002

Business profile information belongs to `UserProfile` rather than turning `ApplicationUser` into a general business entity.

---

## ID-003

Identity and profile responsibilities shall remain separated.

---

# 22. Location Requirements

## GEO-001

Location shall be modeled as a reusable platform concept.

---

## GEO-002

Location-aware functionality shall respect authorization, privacy, and domain ownership.

---

## GEO-003

Location shall support future:

* local discovery;
* marketplace proximity;
* regional behaviour;
* globalization.

---

## GEO-004

Location must not automatically imply continuous tracking.

A future tracking capability must have its own explicit requirements and privacy contract.

---

# 23. Marketplace Requirements

The marketplace bounded context shall be introduced independently from the existing Social context.

Potential capabilities include:

* shops;
* products;
* listings;
* seller identity;
* product media;
* product discovery;
* categories;
* orders;
* payments;
* seller analytics.

Marketplace implementation must not weaken existing SocialConnect architectural boundaries.

---

# 24. Integration Requirements

Every external integration shall terminate at an Infrastructure Adapter Boundary.

Example:

```text
Application Contract
        ↓
Infrastructure Adapter
        ↓
External Provider
```

The application must not directly depend on provider SDK types.

---

# 25. Non-Functional Requirements

## NFR-001 — Maintainability

The system shall remain modular and maintainable through clear ownership boundaries.

---

## NFR-002 — Extensibility

New consumers should be added through configuration and public contracts where an existing reusable capability is suitable.

---

## NFR-003 — Testability

Services and components shall have explicit boundaries suitable for unit, integration, and workflow testing.

---

## NFR-004 — Reliability

Durable asynchronous workflows shall use appropriate persistence and retry mechanisms.

---

## NFR-005 — Observability

Operational workflows shall support appropriate logging, audit, and diagnostic information without leaking implementation details across module boundaries.

---

## NFR-006 — Performance

Performance optimizations shall be introduced based on measured requirements and actual system behaviour rather than prematurely introducing distributed infrastructure.

---

## NFR-007 — Scalability

The modular architecture shall permit future scaling and infrastructure evolution without requiring the current application to be prematurely converted into microservices.

---

## NFR-008 — Security

Security responsibilities shall remain enforced server-side and at appropriate architectural boundaries.

---

## NFR-009 — Usability

Server-rendered Razor UI shall remain usable without requiring large client-side application frameworks for ordinary interactions.

---

# 26. Testing Requirements

Testing shall validate both individual responsibilities and complete workflows.

Important workflow tests shall follow the business lifecycle.

For event-driven functionality:

```text
Business Operation
        ↓
Event Raised
        ↓
Outbox Persistence
        ↓
Dispatcher
        ↓
Handler
        ↓
Recipient Resolution
        ↓
Notification Orchestration
        ↓
Persistence / Delivery Planning
        ↓
Delivery
```

Testing shall verify each meaningful boundary rather than only testing isolated methods.

---

# 27. Development Rules

The following rules are mandatory architectural constraints.

### Rule 1

Do not introduce a new component when an existing reusable component can support the requirement through configuration.

### Rule 2

Do not expose private component implementation.

### Rule 3

Do not move business logic into JavaScript.

### Rule 4

Do not put business rules in controllers.

### Rule 5

Do not put business rules in repositories.

### Rule 6

Do not allow reusable components to depend on their consumers.

### Rule 7

Do not create God services.

### Rule 8

Do not bypass domain/application ownership for administrative convenience.

### Rule 9

Do not introduce infrastructure complexity without a documented requirement.

### Rule 10

Do not redesign locked architecture unless an actual contradiction is demonstrated.

---

# 28. Current Development Sequence

The system shall continue to be developed through controlled vertical slices.

The current architectural direction includes:

```text
Identity / Profiles
        ↓
Feed / Social Interaction
        ↓
Reactions / Comments / Sharing
        ↓
Media
        ↓
Events
        ↓
Notifications
        ↓
Alerting
        ↓
Administration
        ↓
Marketplace
        ↓
Expanded Commerce / Globalization
```

The sequence may evolve where implementation dependencies require it, but architectural ownership must remain intact.

---

# 29. Future Architecture Evolution

The system may eventually evolve toward:

* additional infrastructure scaling;
* cloud deployment;
* distributed processing;
* external integrations;
* recommendation systems;
* marketplace services;
* advanced communication;
* international commerce.

However, future architecture shall be introduced only when supported by concrete requirements.

Microservices are not currently a mandatory architectural requirement.

---

# 30. Documentation Requirements

Every major architectural capability shall have a dedicated canonical document once its contract is sufficiently mature.

Documentation shall distinguish:

* current implementation;
* locked architecture;
* in-development capability;
* planned functionality;
* historical design.

The root `README.md` shall serve as the primary navigation point.

---

# 31. Traceability Principle

Every major feature should ultimately be traceable through:

```text
Business Requirement
        ↓
Domain Responsibility
        ↓
Application Workflow
        ↓
Infrastructure Boundary
        ↓
Persistence
        ↓
UI / API
        ↓
Events
        ↓
Notifications / Alerts
        ↓
Tests
```

This traceability is essential to maintaining architectural consistency as SocialConnect grows.

---

# 32. Final Architectural Contract

The central architectural rule of SocialConnect is:

> **A reusable component owns everything within its domain. Consumers configure it, call its public API, subscribe to its events, and react to its results. Consumers never implement or control the component's internal behaviour.**

At the broader application level:

> **Every domain, module, service, and infrastructure boundary must own its responsibility clearly and expose only the contract required by its consumers.**

This principle is the foundation for SocialConnect's maintainability, extensibility, testability, and future evolution.
