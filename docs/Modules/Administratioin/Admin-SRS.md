# SocialConnect — Central Administration & Moderation

## Final Software Requirements Specification & Implementation Contract

**Project:** SocialConnect Enterprise Platform
**Module:** Central Administration, Governance & Moderation
**Platform:** Social Media + E-Commerce
**Technology Foundation:** ASP.NET Core MVC (.NET 8), EF Core, SQL Server, ASP.NET Core Identity, Razor Views, Bootstrap 5, SignalR
**Architecture:** Unified Single-MVC Modular Monolith
**Document Status:** **CANONICAL — REQUIREMENTS CONTRACT**
**Implementation Status:** **NOT YET IMPLEMENTED**
**Dependency:** **Finalized Event & Notification architecture is a prerequisite for Administration implementation**

---

# 1. Purpose

The SocialConnect Administration subsystem provides centralized, privileged control over platform governance, moderation, security, operational configuration, marketplace governance, and administrative activities.

Administration is a **privileged subsystem within the existing SocialConnect application**.

It is not a separate application, does not introduce a second business architecture, and must not create parallel versions of infrastructure already owned by the existing SocialConnect platform.

The Administration subsystem must operate through the same:

* authentication infrastructure
* authorization foundation
* application/domain services
* persistence architecture
* Unit of Work / transaction architecture
* Event architecture
* Transactional Outbox
* Notification architecture
* target-identification mechanism
* shared infrastructure

used by the rest of SocialConnect.

The purpose of this document is to establish the authoritative requirements, ownership boundaries, architectural constraints, and implementation expectations for Administration.

This document is a **requirements and implementation contract**. It does not by itself define every database entity, application interface, Razor page, or implementation class. Those details are defined by the required companion contracts.

---

# 2. Scope

The Administration subsystem covers the following logical capabilities:

```text
Administration
│
├── Authorization & Administrative Access
├── Dashboard / Operational Visibility
├── Account Governance
├── Role Membership
├── Reporting
├── Moderation
├── Moderation Policy
├── Marketplace Governance
├── Platform Configuration
├── Maintenance Operations
├── Event Catalog Policy
├── Notification Channel Policy
├── Security Administration
└── Administrative Audit
```

These are **logical administrative capabilities**, not necessarily separate projects or assemblies.

The Administration architecture must remain capable of incorporating future governance requirements for additional SocialConnect domains without becoming a centralized business-logic or database-bypass layer.

---

# 3. Architectural Vision

Administration is a first-class privileged consumer of the existing SocialConnect architecture.

```text
                         SOCIALCONNECT
                              │
             ┌────────────────┴────────────────┐
             │                                 │
       USER-FACING PLATFORM              ADMINISTRATION
             │                                 │
             │                          Admin MVC UI
             │                                 │
             │                    Admin Application Services
             │                                 │
             └────────────────┬────────────────┘
                              │
                    Existing Application
                       / Domain Services
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
        Posts              Comments            Reactions
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                         Event Model
                              │
                    Transactional Outbox
                              │
                         Dispatcher
                              │
                       Event Handlers
                              │
                    Notification Platform
```

The Administration subsystem therefore **controls and governs the platform without replacing the platform's domain architecture**.

---

# 4. Locked Architectural Principles

The following principles are mandatory.

## 4.1 Administration is a privileged subsystem

Administration exists inside the existing SocialConnect MVC application.

It is not:

* a separate application;
* a separate service;
* a separate authentication system;
* a second business architecture.

---

## 4.2 No ASP.NET Core MVC Areas

Administration shall **not use ASP.NET Core MVC Areas**.

Administration shall use the existing MVC routing and application infrastructure while maintaining structural separation through dedicated:

* controllers;
* application services;
* ViewModels;
* views;
* layouts;
* authorization boundaries;
* feature-specific modules.

The exact physical directory organization may evolve during implementation, but the logical Administration boundary must remain explicit.

---

## 4.3 Unified authentication gateway

All users authenticate through the existing SocialConnect authentication gateway.

Administration shall not introduce a separate administrative login mechanism.

The default post-authentication routing policy is:

```text
/Account/Login
      │
      ↓
Authentication
      │
      ├── Admin → Administration Dashboard
      │
      ├── Moderator → Authorized Administration experience
      │
      └── User/Vendor → Normal application flow
```

A validated and explicitly permitted return URL may be honored when security requirements allow it.

Authentication remains owned by the existing Identity/authentication architecture.

---

## 4.4 Thin Administration controllers

Administration controllers shall remain thin.

Controllers are responsible for:

* HTTP request coordination;
* model binding;
* invoking the appropriate application operation;
* returning the appropriate response/view.

Controllers shall not become owners of:

* administrative business rules;
* moderation state transitions;
* persistence logic;
* event orchestration;
* notification orchestration;
* audit generation;
* authorization business rules.

---

## 4.5 Domain ownership is non-negotiable

The following is a **golden architectural rule**:

> **Administration must never bypass domain ownership.**

Elevated administrative privileges do not permit direct manipulation of another module's persistence merely because the actor is an administrator.

For example, Administration must not directly manipulate Post, Comment, Reaction, Media, Product, Vendor, Order, or other domain tables to perform business operations.

The correct conceptual flow is:

```text
Administrative Command
        ↓
Administrative Authorization
        ↓
Administrative Application Service
        ↓
Appropriate Domain/Application Operation
        ↓
Domain State Change
        ↓
Event where applicable
        ↓
Transactional Outbox
        ↓
Existing Event / Notification infrastructure
```

The domain that owns a business operation remains the authority for that operation.

---

## 4.6 Existing target mechanism is reused

Administration reporting and moderation shall reuse the existing SocialConnect target mechanism:

```text
TargetType + TargetId
```

Administration must not introduce a competing target identity architecture.

The target mechanism must remain extensible as additional entities become administratively governable.

---

## 4.7 Reporting and moderation are distinct

A report is an assertion that a target may require administrative attention.

Moderation is the process of reviewing and acting upon that assertion.

The canonical conceptual lifecycle is:

```text
Report
   ↓
Moderation Case / Review
   ↓
Decision
   ↓
Administrative Action
```

These concepts must not be collapsed into a single report record or treated as interchangeable.

---

## 4.8 Moderation is policy-driven

Moderation shall operate according to an explicit Moderation Policy model.

The policy architecture must be capable of defining:

* policy categories;
* report reasons;
* review outcomes;
* moderation decisions;
* permitted administrative actions;
* escalation requirements;
* applicable consequences.

The policy model must remain extensible.

---

## 4.9 Marketplace governance is first-class

SocialConnect is both a social platform and an e-commerce platform.

Marketplace governance therefore requires its own administrative boundary.

Administration must be capable of supporting governance for:

* Vendors;
* Products;
* Marketplace listings;
* Vendor status;
* Product governance;
* Marketplace policies;
* Platform commissions;
* Marketplace configuration;
* Marketplace moderation;
* future order/dispute governance.

These capabilities may be implemented incrementally, but their ownership boundaries must not be lost.

---

## 4.10 Typed and governed configuration

Platform configuration shall not be treated as uncontrolled arbitrary key/value data.

Configuration shall be classified into appropriate categories, including where applicable:

```text
Platform
Security
Moderation
Marketplace
Notification
Event
Operational
```

Every configurable setting must have defined:

* ownership;
* type;
* semantics;
* validation;
* default behavior;
* permitted administrative mutability.

---

## 4.11 Configuration policy is different from immutable contracts

The following is a locked rule:

> **Admin-configurable platform policy ≠ immutable application/domain constants.**

Administration may modify approved operational policy.

Administration must not modify immutable architectural or domain contracts.

Potentially configurable examples include:

```text
MaintenanceMode
PlatformLimits
CommissionPolicy
ModerationPolicySetting
NotificationChannelEnabled
EventOperationalPolicy
```

Potentially immutable examples include:

```text
Domain invariants
Event semantic contracts
Event payload contracts
Internal identifiers
Database invariants
Security-critical architectural constants
```

The final implementation must explicitly classify each setting.

---

## 4.12 Maintenance Mode is a platform capability

Maintenance Mode is a platform-wide operational policy.

It must explicitly define behavior for:

* normal users;
* administrators;
* moderators;
* authentication;
* MVC requests;
* APIs;
* background processing;
* Event processing;
* Transactional Outbox processing;
* notification processing;
* delivery workers;
* scheduled operations.

Maintenance Mode must not be implemented merely as an isolated Boolean whose consequences are undefined elsewhere.

---

## 4.13 Administrative audit is mandatory

All privileged administrative mutations must be auditable.

Audit generation must occur at the application/service boundary rather than depending solely on:

```text
AdminController
```

or:

```text
Admin Razor View
```

The audit architecture must be capable of recording, where applicable:

```text
Actor
Action
Target
Timestamp
Reason
Outcome
Before State
After State
Correlation Information
Request / Security Context
```

Not every operation must populate every field.

Administrative audit records must not be silently editable or deletable through ordinary Administration functionality.

Where SocialConnect provides a platform-level audit infrastructure, Administration must use that authoritative mechanism rather than introducing a competing audit architecture.

---

## 4.14 Existing Event and Outbox architecture is reused

Administration must reuse the canonical SocialConnect Event architecture:

```text
Event
→ Transactional Outbox
→ Dispatcher
→ Event Handler
→ Consequences
```

Administration must not introduce:

* AdminEventBus;
* AdminOutbox;
* AdminDispatcher;
* AdminNotificationPipeline;
* or equivalent parallel infrastructure.

---

## 4.15 Administrative operations may produce canonical events

Where an administrative operation changes business/domain state, the operation must participate in the existing Event architecture where the canonical event contract requires it.

Conceptually:

```text
Admin Operation
      ↓
Domain State Change
      ↓
Canonical Event
      ↓
Transactional Outbox
      ↓
Dispatcher
      ↓
Event Handler
      ↓
Consequences
```

The exact event definitions remain governed by the canonical Event architecture.

---

# 5. Administrative Actors

SocialConnect Version 1 shall contain exactly these four system roles:

```text
Admin
Moderator
Vendor
User
```

There shall be:

> **NO SUPER ADMIN ROLE IN VERSION 1.**

A future authorization model may become more granular through permissions/capabilities without introducing a SuperAdmin role merely to solve authorization complexity.

---

# 6. Administrative Role Model

## 6.1 Admin

The `Admin` role represents the primary platform-governance authority.

Depending on the finalized authorization contract, Admin capabilities may cover:

* Administration dashboard;
* account governance;
* role membership;
* moderation;
* marketplace governance;
* platform configuration;
* Event Catalog operational policy;
* Notification Channel policy;
* Maintenance Mode;
* audit;
* security administration;
* platform operations.

Exact permissions must be defined by the Administration Authorization Contract.

---

## 6.2 Moderator

The `Moderator` role may access Administration only within its authorized moderation scope.

Moderator does not automatically receive:

* system configuration authority;
* role-management authority;
* marketplace configuration authority;
* Event Catalog governance;
* notification channel configuration;
* security administration.

Additional permissions require explicit authorization design.

---

## 6.3 Vendor

A Vendor remains a normal platform role and does not automatically receive Administration access.

Vendor administration is performed by authorized administrative actors.

---

## 6.4 User

A normal User has no Administration privileges.

---

# 7. Authorization Architecture

Administration shall use an extensible authorization model.

The architectural model is:

```text
System Role
     ↓
Administrative Capability / Permission
     ↓
Administrative Operation
```

rather than relying exclusively on:

```text
Role == Admin
```

for every administrative operation.

The four Version 1 system roles remain application-controlled.

Administrators may manage membership in predefined system roles where authorized, but Administration shall not provide arbitrary application-role creation.

---

# 8. Role Governance

System roles are application-defined.

Administration shall not provide functionality to:

* create system roles;
* rename system roles;
* delete system roles;
* redefine the semantic meaning of system roles.

Administration may, subject to authorization:

* assign predefined roles to users;
* remove predefined roles from users;
* inspect role membership.

Role definitions remain owned by the application architecture.

---

# 9. Account and User Governance

Administration shall provide centralized account governance.

The architecture must support, as implementation phases permit:

* user search;
* user inspection;
* deterministic pagination;
* account status inspection;
* account suspension;
* account reactivation;
* role membership management;
* administrative notes where justified;
* administrative history;
* additional approved account-governance actions.

Account governance must respect the existing ASP.NET Identity and SocialConnect account architecture.

Administration must not introduce a duplicate account-status mechanism when an authoritative account-state mechanism already exists.

---

# 10. Account Suspension

Account suspension shall be an explicit administrative operation.

The implementation contract must define:

* authorized actors;
* required permission;
* suspension state;
* reason;
* audit requirements;
* user-facing consequences;
* authentication consequences;
* active-session behavior;
* event consequences;
* notification consequences where applicable.

The exact persistence and Identity implementation shall be derived after auditing the existing account architecture.

---

# 11. Reporting Architecture

Reporting is the mechanism through which a potentially problematic platform entity is brought to administrative attention.

Reports shall identify supported targets through:

```text
TargetType
TargetId
```

Potential targets include:

```text
Post
Comment
Media
Profile
Product
Marketplace Listing
```

Additional target types may be introduced as the platform evolves.

The target mechanism itself remains owned by the existing SocialConnect architecture.

---

# 12. Report Lifecycle

Reports shall have an explicit lifecycle.

The initial conceptual states are:

```text
Pending
UnderReview
Resolved
Dismissed
```

The final state machine must define:

* permitted transitions;
* authorized actors;
* timestamps;
* ownership;
* escalation;
* resolution requirements;
* audit behavior.

A report is not itself a moderation decision.

---

# 13. Moderation Case Architecture

The Administration subsystem must preserve the distinction:

```text
Report
Moderation Case / Review
Decision
Action
```

Multiple reports may relate to the same underlying target.

The architecture must therefore allow multiple reports to be associated with an appropriate moderation review/case without making a one-report/one-action assumption mandatory.

The final implementation must preserve the ability to introduce case aggregation and escalation without redesigning the foundational reporting architecture.

---

# 14. Moderation Policy

The Moderation Policy model shall support concepts including:

```text
Policy Category
Policy Rule
Report Reason
Review Outcome
Moderation Decision
Permitted Action
Escalation
```

Examples of report reasons may include:

* Spam;
* Harassment;
* Fraud;
* Illegal content;
* Copyright concerns;
* Misleading content;
* Other policy violation.

These examples do not constitute the complete policy catalog.

Policy definitions must remain extensible.

---

# 15. Moderation Actions

The moderation architecture may support actions such as:

```text
No Action
Dismiss
Warn
Hide
Remove
Restrict
Suspend
Escalate
```

The final action catalog is governed by the Moderation Policy.

An administrative moderation action must invoke the appropriate domain/application operation.

Administration must not directly manipulate another domain's persistence to simulate a moderation operation.

---

# 16. Moderation Auditability

Every moderation decision and administrative moderation action must be traceable.

The audit trail must support investigation of:

```text
Who
What
When
Why
Target
Decision
Action
Outcome
```

Where applicable, the system should preserve the relationship:

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

# 17. Marketplace Governance

Administration shall provide a dedicated Marketplace/E-Commerce Governance boundary.

## 17.1 Vendor Governance

The architecture must support future capabilities including:

* vendor status;
* vendor approval;
* vendor suspension;
* vendor compliance;
* vendor moderation;
* vendor policy enforcement.

## 17.2 Product Governance

The architecture must support future capabilities including:

* product moderation;
* product visibility;
* prohibited-product governance;
* product reporting;
* product policy enforcement.

## 17.3 Marketplace Governance

The architecture must support future capabilities including:

* commission policies;
* marketplace configuration;
* seller rules;
* marketplace limits;
* platform fees;
* marketplace operational policies.

## 17.4 Future Commerce Governance

The architecture must leave room for:

* order governance;
* refund/dispute governance;
* marketplace escalations;
* transaction-related administrative review.

Implementation of these capabilities is phased and must not be assumed to exist merely because the architectural boundary exists.

---

# 18. Platform Configuration

Administration shall provide centralized management of approved platform configuration.

Configuration categories include, where applicable:

```text
Platform
Security
Moderation
Marketplace
Notification
Event
Operational
```

A configuration item conceptually contains:

```text
Key
Type
Group
Description
Default
Validation Rules
Current Value
Last Modified
Modified By
```

The exact persistence model is defined by the Administration Data Model Contract.

---

# 19. Event Catalog Administration

The SocialConnect Event Catalog is an administrative policy surface, not an event-contract editor.

Administration may eventually govern approved operational policies such as:

* event enablement;
* notification eligibility;
* event-related operational policy;
* approved channel policy.

Administration must **not** redefine:

* event semantic meaning;
* event ownership;
* event identity;
* payload schema;
* version contract;
* fundamental event contract.

Those remain owned by the canonical Event architecture.

The Event Catalog therefore represents **administrative policy over existing events**, not the definition of the events themselves.

---

# 20. Notification Channel Administration

Administration may control the operational state of existing notification channels.

Initial channels are:

```text
In-App
Email
Push
SMS
```

Authorized administrators may:

* enable an existing channel;
* disable an existing channel;
* manage approved channel-level operational policy.

Administrators may not:

* create a new notification channel;
* redefine the fundamental implementation of a channel;
* delete a channel.

The Notification subsystem remains the execution authority.

Administration provides policy/configuration only.

---

# 21. Event → Notification Boundary

The canonical Event → Notification flow remains:

```text
Business Operation
      ↓
Event Raised
      ↓
Transactional Outbox Persistence
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
Delivery
```

Administration may govern approved policy at appropriate points in this architecture.

Administration must not replace or bypass the chain.

For example:

```text
Admin
 ↓
Event Catalog Policy
 ↓
Existing Event Infrastructure
 ↓
Notification Policy
 ↓
Channel Selection
 ↓
Delivery
```

Administration is therefore a **policy/control surface**, not an alternative event or notification engine.

---

# 22. Maintenance Mode

Maintenance Mode shall be a first-class operational capability.

The final implementation must define its behavior across the platform.

## 22.1 Administrative behavior

Authorized administrators must remain able to access the Administration subsystem when permitted by the maintenance policy.

## 22.2 User behavior

Normal users must receive the appropriate maintenance experience.

## 22.3 API behavior

API endpoints must follow explicitly defined maintenance policy.

## 22.4 Authentication behavior

Authentication behavior during maintenance must be explicitly defined.

## 22.5 Background processing

The following must have explicitly defined behavior:

* Event dispatcher;
* Transactional Outbox workers;
* Notification workers;
* delivery workers;
* scheduled jobs;
* other background processing.

Maintenance Mode is therefore a **platform-wide operational policy**.

---

# 23. Administrative Audit Architecture

The Administration subsystem must provide comprehensive administrative auditability.

The conceptual audit contract includes:

```text
Actor
Action
Target
Timestamp
Reason
Outcome
Before State
After State
Correlation Information
Request / Security Context
```

Not every operation necessarily requires every field.

Audit generation occurs at the application/service boundary.

Audit records must be immutable from the ordinary Administration UI.

Administrators must not be able to silently alter or delete historical audit records.

Operational logs and administrative audit records remain distinct:

```text
Operational Logging ≠ Administrative Audit
```

Technical logs must not be treated as a replacement for formal administrative audit.

---

# 24. Dashboard

The Administration Dashboard shall provide centralized operational visibility.

The initial conceptual dashboard may expose:

```text
Total Users
Active Vendors
Pending Reports
Moderation Status
System Maintenance Status
Recent Administrative Actions
Important Moderation Items
Platform Status
```

Dashboard information must be provided through appropriate query/application services and presentation ViewModels.

The dashboard must not become an owner of domain business logic.

---

# 25. Administration UI Architecture

Administration shall use a dedicated administrative presentation structure within the existing MVC application.

The Admin UI shall have its own:

* navigation;
* layout;
* ViewModels;
* controllers;
* views;
* feature-specific JavaScript where required.

The Admin layout remains separate from the ordinary user-facing layout.

The UI must follow the established SocialConnect frontend architecture:

* Razor Views;
* Bootstrap 5;
* server-side rendering;
* minimal JavaScript;
* feature-specific JavaScript modules;
* no Administration business logic in `site.js`.

---

# 26. Admin Navigation Contract

The initial navigation architecture should accommodate:

```text
Administration
│
├── Dashboard
│
├── Users & Accounts
│   ├── Users
│   ├── Account Status
│   └── Roles
│
├── Moderation
│   ├── Reports
│   ├── Moderation Queue
│   ├── Decisions
│   └── Moderation History
│
├── Marketplace
│   ├── Vendors
│   ├── Products
│   └── Marketplace Governance
│
├── Platform
│   ├── System Settings
│   ├── Platform Limits
│   └── Maintenance
│
├── Events & Notifications
│   ├── Event Catalog
│   └── Notification Channels
│
├── Security
│
└── Audit
```

The exact page structure, Razor contract, forms, ViewModels, accessibility behavior, and interaction details are defined by the dedicated Administration HTML/Razor Contract.

---

# 27. Query and Command Separation

Administration shall distinguish:

```text
Queries
```

from:

```text
Commands
```

Queries retrieve administrative information.

Commands perform administrative mutations.

Examples of queries:

```text
GetUsers
GetReports
GetModerationQueue
GetAuditHistory
GetSystemSettings
```

Examples of commands:

```text
SuspendUser
ReactivateUser
AssignRole
RemoveRole
ProcessReport
ApplyModerationAction
UpdatePlatformSetting
SetMaintenanceMode
EnableNotificationChannel
DisableNotificationChannel
UpdateEventPolicy
```

The final application contracts must preserve this separation.

---

# 28. Security Requirements

Administration is a high-privilege subsystem and must receive appropriate security controls.

The implementation must address:

* authentication;
* authorization;
* anti-forgery protection;
* privilege boundaries;
* input validation;
* target authorization;
* auditability;
* session/security behavior;
* sensitive configuration protection;
* concurrency;
* unauthorized access;
* privilege escalation prevention.

Administrative operations must fail closed when authorization cannot be established.

---

# 29. Administrative Target Security

Every administrative operation involving a target must validate:

```text
TargetType
TargetId
```

before performing the operation.

The system must verify:

* target type is supported;
* target exists where required;
* target belongs to the expected domain;
* requested operation is permitted;
* actor has the required permission;
* operation does not violate domain rules.

Broad administrative privileges do not eliminate target validation.

---

# 30. Concurrency Requirements

Administrative operations must be safe under concurrent execution.

Examples include:

```text
Two moderators processing the same report
Two administrators changing the same setting
Two administrators changing account status
Multiple workers processing administrative consequences
```

The implementation must prevent inconsistent administrative state.

Where optimistic concurrency is required, the exact mechanism shall be defined by the Administration Data Model and Application Contracts.

---

# 31. Event and Notification Integration Prerequisite

Administration implementation shall depend upon the finalized canonical Event and Notification architecture.

Before Administration implementation begins, the following contracts must be finalized:

### Event

* Event model;
* event metadata;
* event identity;
* versioning;
* event contract semantics.

### Transactional Outbox

* persistence;
* status;
* retry;
* lease;
* dead-letter behavior;
* idempotency.

### Dispatcher

* dispatch behavior;
* claim behavior;
* failure handling;
* worker behavior.

### Event Handlers

* handler execution model;
* failure/retry behavior;
* consequence boundaries.

### Recipient Resolution

* candidate sources;
* resolution rules;
* ownership;
* deduplication.

### Notification

* orchestration;
* notification persistence;
* delivery planning;
* channel selection;
* delivery execution.

Administration shall reference these contracts rather than redefine them.

---

# 32. Administrative Event Lifecycle

When an administrative operation produces a domain-significant event, the expected lifecycle is:

```text
Administrative Command
        ↓
Authorization
        ↓
Validation
        ↓
Administrative Application Service
        ↓
Appropriate Domain/Application Operation
        ↓
State Change
        ↓
Canonical Event
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Dispatcher
        ↓
Event Handler
        ↓
Consequences
```

Where notification is applicable:

```text
Event Handler
      ↓
Recipient Resolution
      ↓
Notification Orchestration
      ↓
Notification Persistence
      ↓
Delivery Planning
      ↓
Channel Delivery
```

The exact event and notification behavior is governed by the canonical Event and Notification contracts.

---

# 33. Notification Consequences

Administrative operations may produce notifications where the finalized Notification Policy permits them.

For example:

```text
Account Suspended
      ↓
Canonical Account Event
      ↓
Notification Policy
      ↓
Recipient Resolution
      ↓
Channel Selection
      ↓
Delivery
```

Administration must not directly construct or deliver notifications outside the canonical Notification architecture.

---

# 34. No Duplicate Infrastructure

Administration shall not introduce duplicate versions of:

* authentication;
* authorization infrastructure;
* repositories;
* Unit of Work;
* event bus;
* transactional outbox;
* dispatcher;
* notification service;
* media infrastructure;
* target resolution;
* audit infrastructure where an existing authoritative mechanism exists.

Existing SocialConnect infrastructure must be reused wherever architectural ownership permits.

---

# 35. Extensibility

The Administration subsystem must support future platform growth.

The initial platform includes social capabilities such as:

```text
Posts
Comments
Reactions
Media
```

The platform is expected to expand into:

```text
Marketplace
Vendors
Products
Orders
Reviews
Payments
Additional Social Entities
Additional Moderation Targets
Additional Events
Additional Notification Policies
```

New administrative capabilities must be addable without redesigning the foundational Administration architecture.

However, extensibility must not be achieved by making Administration the owner of every domain's business rules.

---

# 36. Administrative Module Boundaries

The Administration subsystem logically contains:

```text
Administration
│
├── Authorization & Access
├── Dashboard
├── Account Governance
├── Role Membership
├── Reporting
├── Moderation
├── Moderation Policy
├── Marketplace Governance
├── Platform Configuration
├── Maintenance Operations
├── Event Catalog Policy
├── Notification Channel Policy
├── Security Administration
└── Audit
```

These are logical boundaries within the Administration subsystem.

They do not imply that every capability requires a separate project or service.

---

# 37. Data Model Requirements

The final database model must be derived from this requirements contract and the dedicated Administration Data Model Contract.

Potential administrative concepts include:

```text
ContentReport
ModerationCase
ModerationDecision
ModerationAction
ModerationPolicy
PlatformSetting
AdministrativeAuditRecord
```

Additional Marketplace governance entities shall be derived from the finalized e-commerce requirements.

The implementation must not blindly reproduce these conceptual names as database entities without architectural review.

This document defines **required concepts and behavior**, not a final EF Core model.

---

# 38. Content Report Requirements

A Content Report conceptually contains:

```text
Report Identity
Reporter
Target Type
Target ID
Report Reason
Additional Details
Status
Created At
```

Administrative resolution information must be represented through the appropriate moderation review/case, decision, and action architecture rather than forcing the entire moderation lifecycle into a single report record.

---

# 39. Administrative History

Authorized administrative users must be able to inspect appropriate historical administrative activity.

History must support investigation of:

```text
Who performed the operation
What changed
Which target was affected
When it occurred
Why it occurred
What the outcome was
```

Historical information must respect:

* authorization;
* privacy;
* sensitive-data protection;
* audit access policy.

---

# 40. Bulk Operations

Bulk administration is not automatically implied by ordinary administrative operations.

Where bulk operations are introduced, each operation must explicitly define:

* authorization;
* validation;
* partial-failure behavior;
* transaction boundaries;
* audit behavior;
* concurrency;
* event generation;
* notification consequences.

Bulk operations must never become a shortcut around normal domain operations.

---

# 41. Error Handling

Administrative operations must provide deterministic handling of:

* unauthorized requests;
* invalid targets;
* nonexistent entities;
* stale state;
* concurrent modifications;
* invalid configuration;
* invalid role assignment;
* invalid moderation transitions;
* unavailable dependencies;
* event/outbox failures.

Administrative failures must not leave partially completed state changes.

Where an operation spans multiple required state changes, transaction behavior must be explicitly defined.

---

# 42. Validation

All administrative input must be validated at the application boundary.

This includes:

* user identifiers;
* role identifiers;
* target identifiers;
* report data;
* moderation actions;
* configuration values;
* event policy values;
* notification channel settings.

Validation shall follow the existing SocialConnect validation architecture.

The browser must never be considered a trusted source of administrative authority.

---

# 43. Administrative Transactions

Where an administrative command changes multiple pieces of state that must remain atomic, it shall use the existing Unit of Work / transaction architecture.

Where an administrative operation produces an event, the canonical Transactional Outbox requirements apply.

Every event-producing administrative command must have an explicitly defined transaction boundary during implementation.

The expected principle is:

```text
Administrative State Change
        +
Audit
        +
Event / Outbox where applicable
        ↓
Atomic Transaction
```

The exact transaction design remains subject to the canonical Event/Outbox contract and the implementation-level application contract.

---

# 44. Security and Privilege Escalation Prevention

Administration must prevent unauthorized privilege escalation through:

* role assignment;
* configuration changes;
* target manipulation;
* event policy manipulation;
* notification policy manipulation;
* marketplace configuration;
* crafted identifiers;
* forged requests;
* direct endpoint invocation.

Authorization must be enforced server-side.

The browser must never be treated as a trusted source of administrative authority.

---

# 45. Browser Trust Boundary

The Administration UI may expose identifiers required for legitimate operations.

However, the server must independently validate:

```text
Route Values
Form Values
Hidden Inputs
JavaScript Values
Query Parameters
TargetType
TargetId
```

No identifier received from the client is authoritative merely because it originated from an Administration page.

---

# 46. Administrative API/UI Boundary

Administration may initially be implemented primarily through MVC/Razor interactions.

If administrative APIs are introduced later, they must reuse the same Administration application services and authorization contracts.

An API must not create a second set of administrative business rules.

The architectural model remains:

```text
MVC UI ───────────┐
                  ├──→ Application Service
Admin API ────────┘
```

rather than:

```text
MVC UI → Admin Logic A
API    → Admin Logic B
```

---

# 47. Logging and Observability

Administration must provide sufficient observability for:

* failed administrative operations;
* authorization failures;
* moderation failures;
* configuration failures;
* event integration failures;
* notification integration failures;
* concurrency conflicts;
* background processing failures.

Operational logging and administrative audit remain separate concerns.

```text
Technical Diagnostics ≠ Administrative Audit
```

---

# 48. Non-Functional Requirements

Administration shall satisfy SocialConnect's non-functional requirements for:

* security;
* authorization;
* consistency;
* concurrency;
* performance;
* availability;
* auditability;
* maintainability;
* extensibility;
* accessibility;
* reliability;
* observability.

Administrative functionality must not introduce unacceptable degradation to normal user-facing workloads.

Detailed implementation-level NFR requirements must be finalized before each corresponding capability is implemented.

---

# 49. Performance Requirements

Administrative queries such as:

* user search;
* moderation queue;
* report lookup;
* audit history;
* dashboard counts;
* vendor lookup;
* product governance lists

must use appropriate server-side querying.

Ordinary administrative screens must not load unrestricted datasets into application memory.

Large datasets must be paginated and filtered at the persistence/query boundary.

---

# 50. Pagination

Administrative lists must use deterministic pagination.

This applies to:

* users;
* reports;
* moderation cases;
* audit records;
* vendors;
* products;
* other potentially large datasets.

Ordering must be deterministic.

The implementation must choose a pagination strategy appropriate to the query and expected data volume while preserving stable results under concurrent data changes as far as the chosen strategy permits.

---

# 51. Accessibility

The Administration UI must follow SocialConnect accessibility requirements.

Administrative functionality must support:

* keyboard navigation;
* semantic HTML;
* accessible form labels;
* validation messages;
* focus management;
* accessible status/error messaging.

The detailed HTML/Razor accessibility contract is defined separately.

---

# 52. Frontend JavaScript

Administration shall follow the SocialConnect JavaScript architecture.

`site.js` must not contain Administration business logic.

Feature-specific Administration JavaScript, where necessary, must be implemented as dedicated modules.

Server-side rendering remains the default.

Client-side code may coordinate:

* UI interaction;
* minimal client state;
* API requests where applicable;
* component events.

Business rules remain server-side.

---

# 53. Future SignalR Compatibility

Administration shall remain compatible with future real-time operational functionality through SignalR.

Potential future uses include:

* moderation queue updates;
* operational alerts;
* platform status;
* administrative notifications.

SignalR is a delivery/real-time mechanism.

It must not replace the canonical Event, Outbox, or Notification architecture.

---

# 54. Implementation Phasing

Administration shall be implemented incrementally.

## Phase A — Administration Foundation

* Admin authentication routing;
* administrative authorization;
* Admin layout/navigation;
* dashboard foundation;
* user management;
* role membership management;
* account governance;
* audit foundation.

## Phase B — Reporting & Moderation

* reporting;
* moderation queue;
* moderation policy;
* moderation review;
* moderation decisions;
* moderation actions;
* moderation history.

## Phase C — Platform Operations

* platform settings;
* typed configuration;
* Maintenance Mode;
* platform limits;
* operational status.

## Phase D — Event & Notification Administration

* Event Catalog operational policy;
* event enablement policy;
* notification channel enable/disable;
* approved notification operational configuration.

This phase is dependent upon the finalized Event & Notification architecture.

## Phase E — Marketplace Governance

* vendor governance;
* product governance;
* marketplace policy;
* commission/platform policy;
* marketplace moderation;
* future commerce governance.

## Phase F — Advanced Governance

Potential future capabilities include:

* advanced security administration;
* advanced analytics;
* escalation workflows;
* controlled bulk operations;
* advanced operational monitoring.

---

# 55. Implementation Dependency Rule

Administration implementation must not begin until the foundational Event and Notification architecture has reached its required final state.

Administration must reference the canonical Event and Notification contracts.

No Administration-specific alternative infrastructure may be created merely because those modules are implemented elsewhere.

The dependency is:

```text
Event Architecture
        ↓
Notification Architecture
        ↓
Administration Integration Contracts
        ↓
Administration Implementation
```

Administration may be designed in parallel at the requirements level, but implementation of dependent Event/Notification administrative functionality must wait for the canonical contracts.

---

# 56. Implementation Sequence

The eventual implementation sequence shall follow dependency order:

```text
1. Requirements Freeze
        ↓
2. Authorization Contract
        ↓
3. Administration Application Contracts
        ↓
4. Account Governance
        ↓
5. Audit Foundation
        ↓
6. Reporting
        ↓
7. Moderation Policy
        ↓
8. Moderation Workflow
        ↓
9. Platform Configuration
        ↓
10. Maintenance Mode
        ↓
11. Event Catalog Integration
        ↓
12. Notification Channel Integration
        ↓
13. Marketplace Governance
        ↓
14. HTML/Razor Contract
        ↓
15. Integration Testing
        ↓
16. Security Testing
        ↓
17. Concurrency Testing
        ↓
18. Cross-module E2E Testing
```

The sequence may be refined during implementation planning provided that the architectural dependencies and ownership boundaries remain intact.

---

# 57. Testing Requirements

Every implemented administrative capability must include appropriate testing.

## 57.1 Unit tests

Cover:

* business rules;
* authorization rules;
* validation;
* state transitions;
* configuration rules;
* moderation policy.

## 57.2 Application tests

Cover:

* commands;
* queries;
* transaction boundaries;
* audit generation;
* event generation;
* domain/application delegation.

## 57.3 Integration tests

Cover relevant integration with:

* ASP.NET Identity;
* EF Core;
* SQL Server;
* Event/Outbox infrastructure;
* Notification infrastructure.

## 57.4 Security tests

Cover:

* unauthorized access;
* role boundaries;
* permission boundaries;
* privilege escalation;
* forged identifiers;
* direct endpoint invocation;
* CSRF;
* target authorization.

## 57.5 Concurrency tests

Cover:

* simultaneous moderation;
* simultaneous account operations;
* simultaneous configuration changes;
* duplicate administrative commands.

## 57.6 End-to-end tests

Representative workflows include:

### Admin Login

```text
/Account/Login
      ↓
Authentication
      ↓
Admin
      ↓
Administration Dashboard
```

### Moderation

```text
Moderator Login
      ↓
Moderation
      ↓
Review Report
      ↓
Decision
      ↓
Action
      ↓
Audit
```

### Administrative Domain Operation

```text
Admin
      ↓
Suspend User
      ↓
Domain/Application Operation
      ↓
State Change
      ↓
Canonical Event
      ↓
Transactional Outbox
      ↓
Dispatcher
      ↓
Handler
      ↓
Notification where applicable
      ↓
Audit
```

### Event Policy

```text
Admin
      ↓
Change Event Policy
      ↓
Event Catalog
      ↓
Existing Event Infrastructure
```

---

# 58. Administrative Acceptance Criteria

The Administration subsystem shall not be considered complete until the applicable acceptance criteria are satisfied.

1. Unauthorized users cannot access Administration.
2. Moderators can access only their authorized administrative capabilities.
3. Admins can access their authorized platform-governance capabilities.
4. No SuperAdmin role exists in Version 1.
5. System roles cannot be created, renamed, or deleted through Administration.
6. User role membership is securely managed.
7. Account governance respects existing Identity/domain ownership.
8. Reporting uses `TargetType + TargetId`.
9. Reporting and moderation remain distinct concepts.
10. Moderation follows an explicit policy model.
11. Moderation actions respect domain ownership.
12. Administrative mutations are auditable.
13. Historical audit records cannot be silently modified through ordinary Administration functionality.
14. Platform settings are typed and validated.
15. Immutable application/domain contracts cannot be modified through Administration.
16. Maintenance Mode has explicitly defined platform-wide behavior.
17. Event Catalog administration controls approved operational policy only.
18. Event definitions and contracts cannot be arbitrarily modified by Admin.
19. Existing notification channels can be enabled/disabled by authorized Admins.
20. New notification channels cannot be created through Administration.
21. Existing Event/Outbox/Dispatcher infrastructure is reused.
22. Existing Notification infrastructure is reused.
23. No duplicate Administration event or notification infrastructure exists.
24. Administrative domain operations generate canonical events where required.
25. Marketplace governance is architecturally extensible.
26. Administrative queries use appropriate server-side pagination.
27. Administrative ordering is deterministic.
28. Administrative concurrency is handled safely.
29. Security and privilege-escalation tests pass.
30. Cross-module Event/Notification integration tests pass.
31. Administrative APIs, if introduced, reuse the same application services.
32. Administration does not bypass domain ownership.
33. Administration does not become the owner of unrelated domain business rules.
34. The Admin UI follows the SocialConnect MVC/Razor architecture.
35. `site.js` contains no Administration business logic.

---

# 59. Explicit Architectural Ownership Boundaries

The following ownership boundaries are mandatory.

## Administration owns

```text
Administrative authorization
Administrative workflows
Administrative governance
Administrative configuration policy
Administrative moderation
Administrative operational visibility
Administrative policy control
Administrative audit operations
```

## Existing domains own

```text
Post behavior
Comment behavior
Reaction behavior
Media behavior
Product behavior
Vendor business behavior
Order behavior
Payment behavior
Other domain-specific business rules
```

## Event infrastructure owns

```text
Event contracts
Event persistence
Transactional Outbox
Dispatch
Handler execution
Event delivery semantics
Retry / lease / dead-letter behavior
Idempotency
```

## Notification infrastructure owns

```text
Recipient resolution
Notification orchestration
Notification persistence
Channel selection
Delivery planning
Delivery execution
```

Administration may control approved policy but must not take ownership away from these systems.

---

# 60. Future Administrative Capability Rule

Whenever a new SocialConnect domain requires Administration functionality, the capability must first identify:

```text
1. Domain owner
2. Administrative authority
3. Authorization requirement
4. Application service boundary
5. Persistence requirements
6. Audit requirement
7. Event requirement
8. Notification requirement
9. Security implications
10. UI contract
11. Testing requirements
```

Only after these boundaries are established may implementation begin.

This prevents Administration from becoming a centralized "god module" that directly manipulates every database entity.

---

# 61. Canonical Administrative Command Flow

For administrative operations that affect platform or domain state, the canonical model is:

```text
Admin / Moderator
        ↓
Admin UI / API
        ↓
HTTP Request
        ↓
Authorization
        ↓
Validation
        ↓
Administrative Application Service
        ↓
Appropriate Domain/Application Service
        ↓
Transaction
        │
        ├── State Change
        ├── Audit
        └── Event + Outbox where applicable
                    ↓
                  Commit
                    ↓
                Dispatcher
                    ↓
                Event Handler
                    ↓
          Notification / Other Effects
```

The exact event and notification behavior is determined by the canonical Event and Notification contracts.

---

# 62. Required Companion Contracts

This SRS is the governing Administration requirements contract.

The following supporting contracts must be created before the corresponding implementation work is finalized.

## Contract A — Administration Authorization Contract

Defines:

* roles;
* permissions;
* capabilities;
* capability matrix;
* authorization rules;
* administrative operation permissions.

## Contract B — Administration HTML/Razor Contract

Defines:

* Admin layout;
* navigation;
* pages;
* ViewModels;
* forms;
* validation;
* accessibility;
* interaction behavior;
* feature-specific JavaScript requirements.

## Contract C — Administration Data Model Contract

Defines:

* entities;
* relationships;
* foreign keys;
* indexes;
* constraints;
* persistence rules;
* concurrency;
* audit persistence;
* transaction requirements.

## Contract D — Administration Event Integration Contract

Defines:

* which administrative operations produce canonical events;
* event mapping;
* event payload requirements;
* transaction boundaries;
* downstream consequences.

## Contract E — Administration Notification Policy Contract

Defines:

* which administrative events may produce notifications;
* recipient policy;
* notification eligibility;
* channel policy;
* administrative configuration boundaries.

## Contract F — Marketplace Governance Contract

Defines the e-commerce administration domain in sufficient detail before Marketplace Administration implementation begins.

---

# 63. Non-Functional / Security / Concurrency Reference

**[NFR-SEC-CONCURRENCY-01]**

Administration must satisfy SocialConnect's non-functional requirements for:

* security;
* authorization;
* consistency;
* concurrency;
* performance;
* auditability;
* reliability;
* accessibility;
* maintainability;
* observability.

Detailed implementation-level requirements for each concern must be finalized before the corresponding feature is implemented and must be validated through dedicated tests.

---

# 64. Documentation and Traceability

Administration implementation must remain traceable to this requirements contract.

Each implemented capability should identify:

```text
Requirement
    ↓
Application Contract
    ↓
Implementation
    ↓
Tests
```

Module documentation must be maintained as implementation progresses.

A future Administration documentation structure may include:

```text
docs/modules/administration/
├── Requirements.md
├── Authorization.md
├── Architecture.md
├── Data-Model.md
├── Implementation.md
├── API.md
├── UI-Razor.md
├── Workflows.md
├── Event-Integration.md
├── Notification-Policy.md
├── Testing.md
└── CHANGELOG.md
```

Only documents that contain finalized material should be created.

---

# 65. Document Governance

This document is the authoritative Administration requirements contract once committed as the canonical version.

After canonicalization:

* implementation must follow this contract;
* deviations require architectural review;
* new administrative functionality must not silently change established boundaries;
* Event/Notification contracts remain separately authoritative;
* domain ownership remains non-negotiable;
* security requirements cannot be weakened for implementation convenience;
* immutable domain/application contracts cannot be converted into administrative settings merely for convenience.

Changes to this document must preserve architectural consistency with the broader SocialConnect architecture.

---

# 66. Final Architectural Statement

SocialConnect Administration is **not merely an administrative CRUD interface**.

It is the platform's:

> **Governance and Operational Control Layer**

Its responsibility is to provide authorized administrative capabilities over:

```text
Users
Accounts
Roles
Reports
Moderation
Policies
Marketplace Governance
Platform Configuration
Maintenance
Event Policies
Notification Policies
Security
Audit
Operational Visibility
```

while preserving the ownership boundaries of the existing SocialConnect domains.

The governing architectural rules are:

> **Administration controls and governs the platform; it does not replace the platform's domain architecture.**

> **Administrative policy may control approved operational behavior, but it may never redefine immutable application/domain contracts.**

> **Every administrative operation must respect authorization, domain ownership, auditability, transaction integrity, and—where applicable—the canonical Event → Transactional Outbox → Dispatcher → Handler → Notification architecture.**

> **Administration must reuse existing platform infrastructure and must not create parallel implementations of Event, Outbox, Notification, Target Resolution, Authentication, or other already-owned capabilities.**

---

# 67. Current Status

| Area                                  | Status                                                      |
| ------------------------------------- | ----------------------------------------------------------- |
| Requirements                          | **Defined**                                                 |
| Architectural Principles              | **Locked**                                                  |
| Administrative Roles                  | **Locked — Admin, Moderator, Vendor, User**                 |
| SuperAdmin                            | **Explicitly excluded from Version 1**                      |
| ASP.NET MVC Areas                     | **Explicitly excluded**                                     |
| Authentication                        | **Unified `/Account/Login` gateway**                        |
| Admin Redirect                        | **Approved**                                                |
| Authorization Model                   | **Capability/permission-based architecture required**       |
| Domain Ownership                      | **Golden Rule — Locked**                                    |
| Target Mechanism                      | **Existing `TargetType + TargetId` — Locked**               |
| Reporting Model                       | **Locked**                                                  |
| Moderation Model                      | **Locked**                                                  |
| Moderation Policy                     | **Required**                                                |
| Marketplace Governance                | **Required architectural boundary**                         |
| Platform Configuration Classification | **Locked**                                                  |
| Maintenance Mode                      | **Required**                                                |
| Audit Architecture                    | **Required**                                                |
| Event Catalog Administration          | **Policy only — Event contracts remain immutable**          |
| Notification Channel Administration   | **Enable/disable existing channels only**                   |
| Event/Outbox Infrastructure           | **Existing canonical infrastructure must be reused**        |
| Notification Infrastructure           | **Existing canonical infrastructure must be reused**        |
| Implementation                        | **Not started**                                             |
| Event/Notification Dependency         | **Must be finalized before dependent Admin implementation** |
| HTML/Razor Contract                   | **Required companion contract**                             |
| Authorization Contract                | **Required companion contract**                             |
| Data Model Contract                   | **Required companion contract**                             |
| Event Integration Contract            | **Required companion contract**                             |
| Notification Policy Contract          | **Required companion contract**                             |
| Marketplace Governance Contract       | **Required before Marketplace Admin implementation**        |
| Security / Concurrency / NFR          | **Required and must be validated through dedicated tests**  |

## Next Status Transition

```text
CANONICAL REQUIREMENTS CONTRACT
             ↓
Administration Authorization Contract
             ↓
Administration Application Contracts
             ↓
Administration Data Model Contract
             ↓
Administration Event / Notification Integration Contracts
             ↓
Implementation
             ↓
Integration / Security / Concurrency / E2E Validation
```

**No Administration implementation should begin until the required prerequisite contracts and the canonical Event/Notification dependency have reached the required finalized state.**
