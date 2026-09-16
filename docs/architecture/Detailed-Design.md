# SocialConnect — Detailed System Design

**Document Status:** Canonical
**Version:** 1.0
**Scope:** System-level detailed engineering design
**Related Architecture:** `docs/architecture/Architecture.md`

---

# 1. Purpose

This document translates the SocialConnect system architecture into detailed engineering rules and design structures.

The document describes how the major architectural concepts are expected to be organized and interact.

It intentionally does not replace individual module implementation contracts.

Each major module will eventually receive its own detailed documentation covering its requirements, design, implementation, APIs, workflows, testing, and architectural decisions.

---

# 2. Design Objectives

The detailed design follows these objectives:

1. Maintain explicit ownership.
2. Keep responsibilities separated.
3. Keep controllers thin.
4. Keep business services focused.
5. Keep persistence concerns isolated.
6. Prefer reusable components.
7. Use stable contracts between modules.
8. Keep client-side logic minimal.
9. Make workflows explicit.
10. Preserve traceability between requirements and implementation.
11. Support incremental module development.
12. Prevent unrelated modules from becoming tightly coupled.

---

# 3. Application Structure

A typical feature follows:

```text
Request
  │
  ▼
Controller / ViewComponent
  │
  ▼
Application Contract
  │
  ▼
Context Builder / Orchestrator
  │
  ├── Loaders
  ├── Resolvers
  ├── Policies
  └── Factories
  │
  ▼
Business Services
  │
  ▼
Persistence / Infrastructure
```

Not every use case requires every element.

The structure is selected according to the complexity and ownership of the feature.

---

# 4. DTO Design

DTOs represent application transport contracts.

They should contain data required to communicate between application boundaries without becoming domain entities.

For example:

```text
CreatePostDto
```

represents input to post creation.

It should not become responsible for:

* Database persistence
* Business workflow
* Media finalization
* Feed publication
* Notification creation

Those responsibilities belong to their respective owners.

---

# 5. ViewModel Design

ViewModels represent server-rendered UI state.

A ViewModel is not a domain entity and should not become a business-service container.

The preferred direction is:

```text
Domain/Application State
        │
        ▼
Builder
        │
        ▼
ViewModel
        │
        ▼
Razor View
```

JavaScript should not be required to reconstruct authoritative server state that could have been rendered by the server.

---

# 6. Manual Mapping

SocialConnect uses **manual mapping**.

AutoMapper is not part of the current architecture.

Mapping should remain explicit so that:

* Data transformations are visible.
* Contracts are easy to audit.
* Unexpected property propagation is avoided.
* Mapping behavior remains close to the relevant application boundary.

---

# 7. Service Design

Services should have one clear responsibility.

A complex workflow should be decomposed.

For example:

```text
PostCreationService
      │
      ├── Media Finalization
      ├── Media Assignment
      ├── Feed Publication
      └── Result Construction
```

The orchestration service coordinates the workflow.

Specialized services own their respective business or infrastructure operations.

---

# 8. Repository and Unit of Work Design

The repository layer abstracts persistence operations.

The architecture distinguishes read and modification requirements.

```text
Query / GetById
      │
      ▼
No Tracking
```

Modification workflows use:

```text
ForUpdate
   │
   ▼
Tracked Entity
```

The Unit of Work coordinates persistence where a use case requires multiple related changes to be committed consistently.

---

# 9. Entity Loader Design

The application layer uses loader abstractions where appropriate.

Primary contracts include:

```text
IEntityLoader<TEntity,TKey>
ITrackedEntityLoader<TEntity,TKey>
```

The loader owns entity retrieval behavior while keeping business services independent of repository implementation details.

---

# 10. Lookup Services

Lookup services act as public read gateways for reusable application data.

They provide a stable application-level contract rather than exposing repositories directly to consumers.

This helps prevent consumers from becoming coupled to persistence implementation details.

---

# 11. Post Creation Detailed Workflow

The canonical post creation workflow is:

```text
1. Post Composer
       │
2. CreatePostDto
       │
3. PostCreationContext
       │
4. PostCreationService
       │
5. Create Post
       │
6. Finalize Media
       │
7. Assign Media
       │
8. Publish Feed
       │
9. PostCreationResult
```

The workflow may validate:

* Content
* Media
* Comment permission
* Reaction permission
* Ownership
* Authorization
* Related context

Future post capabilities such as feelings, mentions, and geographic context can be introduced through the established workflow without mixing media component internals into post creation.

---

# 12. Media Detailed Design

The MediaUploader is a reusable, consumer-independent component.

Its public API includes operations conceptually equivalent to:

```text
init
destroy
getInstance
getMediaItems
getMediaIds
getExistingMedia
getRemovedExistingMediaIds
upload
remove
removeExisting
cancel
```

The component states include:

```text
idle
uploading
completed
cancelled
failed
```

Media lifecycle events use the `media.uploader.*` event namespace.

The consumer configures the uploader and reacts to its results.

The consumer does not implement the uploader's internal upload lifecycle.

---

# 13. Media Storage Ownership

The media pipeline has explicit ownership:

```text
Upload
  → MediaUploadService

Temporary Storage
  → Media infrastructure

Finalization
  → MediaFinalizationService

Assignment
  → MediaAssignmentService
```

Finalization:

```text
temp → final
```

Assignment:

```text
finalized media → owner
```

The two operations must remain independent.

---

# 14. Comment Detailed Design

The CommentManager is a reusable UI component.

Its presentation and interaction state includes:

```text
idle
create
reply
edit
```

and presentation behavior such as:

```text
collapsed
expanded
```

The component uses target-based ownership.

The current target type for comments is:

```text
Comment = 6
```

The supported reply depth is limited according to the current component contract.

The component communicates with consumers through its public events and API rather than exposing internal implementation details.

---

# 15. Reaction Detailed Design

Reaction operations are target-oriented.

The current supported reaction types are:

```text
Like
Love
Haha
Wow
Sad
Angry
```

The reusable mechanism allows supported entities to use the same reaction contract.

The current API boundary includes:

```text
/api/reaction/toggle
```

Feature-specific controllers should remain thin and delegate the operation to the appropriate application/service contract.

---

# 16. Sharing Detailed Design

Internal sharing creates a new post representing the share.

The original content and media are not copied.

```text
Original Post
      │
      ▼
SharedPostId
      │
      ▼
Share Post
```

Shared relationships use restricted deletion behavior where required.

Feed traversal must protect against:

* Excessive graph depth
* Cyclic relationships
* Invalid shared references

The feed context builder is responsible for constructing a safe shared graph.

---

# 17. Event Processing Design

The event architecture separates synchronous business facts from reliable asynchronous processing.

The lifecycle is:

```text
Business Operation
       │
       ▼
Domain Event
       │
       ▼
Outbox Persistence
       │
       ▼
Dispatcher
       │
       ▼
Handler
```

Outbox processing requires reliable claim behavior.

The worker model must account for:

* Concurrent workers
* Leases
* Retry counts
* Processing status
* Dead-letter conditions
* Idempotency
* Transaction boundaries

---

# 18. Event-to-Notification Workflow

Notification-producing events follow the complete processing chain:

```text
Business Operation
       │
       ▼
Event Raised
       │
       ▼
Transactional Outbox Persistence
       │
       ▼
Dispatcher
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
       ▼
Channel Delivery
```

This chain is the standard reference workflow for event-driven notifications.

---

# 19. Recipient Resolution Design

Recipient resolution is a dedicated concern.

Candidate sources are responsible for discovering potential recipients.

The current required source categories include:

```text
PostOwnerSource
CommentOwnerSource
TargetOwnerSource
ParentAuthorSource
```

Their responsibilities are conceptually:

### PostOwnerSource

Resolve:

```text
Post.UserId
```

### CommentOwnerSource

Resolve:

```text
Comment.UserId
```

### TargetOwnerSource

Resolve:

```text
TargetType + TargetId
        │
        ▼
Target Entity
        │
        ▼
Target Owner
```

### ParentAuthorSource

Resolve:

```text
Comment.ParentCommentId
        │
        ▼
Parent Comment
        │
        ▼
Parent Comment.UserId
```

Candidate source registration is kept separate from the generic recipient-resolution mechanism.

---

# 20. Notification Application Services

The notification module exposes separate query and command responsibilities.

Current application contracts include:

```text
INotificationQueryService
INotificationCommandService
```

The query side handles concerns such as:

* Notification retrieval
* Deterministic pagination
* Unread count
* Visibility
* Expiration-aware filtering

The command side handles:

* Mark read
* Mark unread
* Mark all read

Authenticated-user ownership must be enforced by the application layer.

Controllers remain thin.

---

# 21. Notification API Boundary

The current notification API contract includes:

```text
GET  /api/notifications
GET  /api/notifications/unread-count
POST /api/notifications/{id}/read
POST /api/notifications/{id}/unread
POST /api/notifications/read-all
```

The exact HTTP response contracts are maintained in the module/API documentation.

---

# 22. Notification UI Design

The notification UI uses a single reusable:

```text
NotificationManager
```

rather than independent Bell, Count, List, and Center components.

Its primary state machine is:

```text
CLOSED
   │
   ▼
OPEN
```

The component supports:

* Opening/closing
* Outside-click closing
* Escape-key closing
* Notification selection
* Navigation
* Mark-all-read
* See-all navigation

The server remains authoritative.

JavaScript provides interaction and minimal state.

---

# 23. Alerting Detailed Design

Alerting must remain generic.

The central platform must not contain a large source-specific conditional structure.

Instead:

```text
Source
  │
  ▼
Alert Source Contract
  │
  ▼
Generic Alert Platform
  │
  ▼
Policy
  │
  ▼
Processing
  │
  ▼
Delivery
```

The initial operational evidence domain is the Event/Notification platform.

After the generic alerting platform is verified, additional domains can be integrated independently.

---

# 24. Administration Detailed Design

Administration is a privileged application subsystem.

The existing authentication gateway remains responsible for authentication.

Administrative authorization determines the user's administrative experience.

The defined application roles are:

```text
Admin
Moderator
Vendor
User
```

There is no `SuperAdmin` role in the current architecture.

Administration uses application/domain services.

Direct administrative table manipulation is prohibited where it would bypass domain ownership.

---

# 25. Reporting and Moderation

Reporting and moderation are separate concepts.

The reporting lifecycle is:

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

Moderation policies determine how supported actions are governed.

Administrative configuration may alter policy where explicitly permitted.

Immutable domain rules remain application/domain contracts.

---

# 26. Configuration Design

Configuration categories include:

```text
Platform
Security
Moderation
Marketplace
Notification
Event
Operational
```

The architecture distinguishes:

```text
Configurable Policy
        ≠
Immutable Domain Constant
```

Administrators may configure supported policy values.

They do not redefine the underlying domain contract.

---

# 27. External Provider Integration Design

External providers are isolated through infrastructure adapters.

Example:

```text
Application Service
       │
       ▼
Internal Payment Contract
       │
       ▼
Payment Adapter
       │
       ▼
External Payment Provider
```

The application service must not receive provider-specific response models.

This allows providers to be replaced or added without redesigning business workflows.

---

# 28. UI Component Design

Every reusable ViewComponent follows a consistent structure:

```text
Component
│
├── Defaults
├── ViewModel
├── Builder
├── ViewComponent
├── Default.cshtml
├── JavaScript
├── CSS
└── Documentation
```

The Builder configures the ViewModel.

The ViewComponent renders.

The component's JavaScript owns interaction.

---

# 29. JavaScript Module Design

Feature modules should follow the application module lifecycle:

```text
register
init
bind
destroy
```

Modules should:

* Keep configuration immutable where appropriate.
* Keep constants immutable.
* Track instances safely.
* Avoid global state.
* Clean up event handlers.
* Communicate through documented public events.

`site.js` remains infrastructure/foundation and must not become a feature-service container.

---

# 30. Validation and Authorization

Validation and authorization are server responsibilities.

The server is authoritative for:

* Input validation
* Ownership
* Authorization
* Business rules
* Database consistency
* Workflow state

Client-side validation may improve user experience but cannot replace server validation.

FluentValidation is used where appropriate for application validation contracts.

---

# 31. Soft Delete and Auditing

Where soft deletion is part of a domain's contract, deleted records remain represented according to the applicable persistence policy rather than being treated as ordinary active records.

Auditing is generated at the appropriate service/application boundary.

Administrative users should not be given UI mechanisms to edit or delete audit history.

---

# 32. Transaction Boundaries

Operations involving multiple related state changes must define their transaction boundary explicitly.

Particular attention is required for:

* Post creation
* Media finalization/assignment
* Outbox persistence
* Notification persistence
* Marketplace transactions
* Orders
* Payments
* Administrative actions

The transaction boundary belongs to the owning application workflow rather than arbitrary lower-level components.

---

# 33. Error Handling

Errors should be handled at the appropriate application boundary.

Controllers should translate application outcomes into HTTP/UI responses.

Business services should not become responsible for presentation-specific error rendering.

API contracts should use consistent error representations once the API contract for the relevant module is finalized.

---

# 34. Testing Design

Testing should validate architectural ownership as well as functional behavior.

Tests should cover where applicable:

* Business rules
* Application workflows
* Repository behavior
* Database interactions
* API contracts
* UI component behavior
* Event processing
* Outbox behavior
* Recipient resolution
* Notification persistence
* Alerting
* Authorization
* Ownership enforcement

End-to-end workflows should validate the complete chain rather than testing isolated pieces only.

---

# 35. Module Documentation Model

Every significant module should eventually have documentation appropriate to its complexity.

A typical module documentation set may be:

```text
docs/modules/<module>/
│
├── Requirements.md
├── Architecture.md
├── Implementation.md
├── API.md
├── Workflows.md
└── Testing.md
```

Not every module requires every file.

The documentation should be created according to the module's actual requirements.

The module documentation must remain consistent with the system architecture.

---

# 36. Implementation Traceability

For important functionality, the documentation should make it possible to trace:

```text
Requirement
    │
    ▼
Architectural Decision
    │
    ▼
Application Contract
    │
    ▼
Implementation
    │
    ▼
Test
```

This allows future developers to understand not only what the code does, but why the structure exists.

---

# 37. Architecture Evolution Rules

A locked architecture contract should not be changed merely to introduce an alternative coding style.

Changes require a documented reason.

Possible reasons include:

* New business requirement
* Security requirement
* Correctness issue
* Performance evidence
* Maintainability issue
* Integration requirement
* Architectural contradiction

When a change is accepted, the affected documentation should be updated so that one authoritative version remains.

---

# 38. Current Design Boundaries

The following technologies/designs from earlier project documentation are **not part of the current canonical design unless separately reintroduced and documented**:

* Microservices as the current deployment architecture
* JWT as the primary current Identity architecture
* ML.NET as the required current recommendation engine
* Redis as a required current caching layer
* Nginx as a required current load-balancing layer
* Azure Blob Storage as a current mandatory storage implementation
* Stripe/PayPal as currently implemented payment providers
* Signal Protocol/AES-based chat as the current messaging implementation
* Cloud migration as a current deployment requirement

These may be future implementation options, but they must not be treated as current contracts without explicit architectural approval and documentation.

---

# 39. Related Documentation

This document should be read together with:

* [Project Overview](../requirements/Project-Overview.md)
* [Software Requirements Specification](../requirements/SRS.md)
* [System Architecture](Architecture.md)

Detailed module documentation will extend these system-level documents without duplicating or contradicting their architectural contracts.

---

## Document Status

**Canonical system-level detailed design reference.**

Individual module documentation may provide deeper implementation details while remaining subordinate to the system-wide architectural boundaries defined by the canonical SocialConnect architecture.
