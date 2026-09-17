# SocialConnect

## Hybrid Social Network & E-Commerce Platform

**SocialConnect** is a long-term software engineering project for designing and developing a modular social-network and e-commerce platform using the **Microsoft .NET ecosystem**.

The project covers the complete lifecycle of a modern application — from requirements and architecture through domain modeling, implementation, database design, APIs, reusable UI components, media processing, event-driven workflows, notifications, alerting, testing, and technical documentation.

> **Project status:** Active development and continuous architectural refinement.

---

## 🎯 Project Overview

SocialConnect combines social interaction and commerce capabilities within a single platform.

The platform is designed around concepts such as:

* User identity and profiles
* Social relationships
* Posts and content publishing
* Comments and threaded discussions
* Reactions
* Sharing
* Media management
* Feed generation and discovery
* Notifications
* Event-driven processing
* Marketplace and e-commerce capabilities
* Products, shops, and vendors
* Administrative governance and moderation
* Auditing and soft deletion
* Extensible alerting
* API-based application integration
* Local and global discovery

The long-term product direction is to bridge social discovery and marketplace commerce, combining community interaction with buying and selling capabilities. The marketplace direction is intended to support both **local discovery** and broader/global discovery while remaining extensible for future commerce capabilities.

The project emphasizes **maintainability, explicit ownership, separation of responsibilities, reusable components, well-defined contracts, testability, and production-oriented engineering practices**.

---

# 🏗️ Current Technology Stack

### Backend

* **C#**
* **.NET 8**
* **ASP.NET Core MVC**
* **Entity Framework Core**
* **ASP.NET Core Identity**
* **REST APIs**
* **FluentValidation**

### Database

* **Microsoft SQL Server**
* Entity Framework Core
* Relational data modeling
* Database constraints and relationships
* Auditing and soft-delete concepts

### Frontend

* **Razor Views**
* **Razor ViewComponents**
* **Bootstrap 5**
* HTML5
* CSS
* JavaScript

### Architecture & Engineering

* Layered modular monolith
* DTO-based application boundaries
* Generic Repository
* Unit of Work
* Application services
* Domain-oriented services
* Entity loader abstractions
* Context builders and orchestration
* Event-driven architecture
* Transactional Outbox
* Notification orchestration
* Generic alerting
* Reusable UI components
* Manual mapping
* Automated testing and validation

---

# 🧭 Architecture Overview

SocialConnect is currently designed as a **layered modular monolith**.

The system maintains explicit modular boundaries while operating within a single ASP.NET Core MVC application.

At a high level:

```text
UI / MVC
   │
   ▼
Controller / ViewComponent
   │
   ▼
Application Orchestration
   │
   ├── Context Builders
   ├── Loaders / Resolvers
   ├── Factories
   ├── Policies
   └── ViewModels / DTOs
   │
   ▼
Business / Domain Services
   │
   ▼
Persistence & Infrastructure
   │
   ├── Entity Framework Core
   ├── SQL Server
   ├── Identity
   └── External / Integration Adapters
```

Controllers remain thin.

Application orchestration coordinates use cases and workflows.

Business services own business rules.

Persistence infrastructure owns database access.

Reusable UI components own their own UI state and behavior.

External providers are isolated behind infrastructure adapter boundaries.

---

# 🧩 Core Architectural Principles

SocialConnect follows a set of explicit architectural rules.

### Ownership

A reusable component owns everything within its domain.

Consumers configure it, call its public API, subscribe to its events, and react to its results.

Consumers do not implement or control the component's internal behavior.

### Separation of responsibilities

Controllers, application services, business services, repositories, infrastructure, and UI components have distinct responsibilities.

### Explicit contracts

Important architectural and application contracts are defined before implementation where appropriate.

### Server authority

The server owns authoritative validation, authorization, database state, workflows, and business rules.

### Minimal client responsibility

JavaScript primarily provides UI interaction, API communication, lifecycle management, and minimal client-side state.

### Reusability

Reusable behavior is implemented through components and stable public contracts rather than duplicated feature-specific code.

### Extensibility

New functionality should normally be introduced through configuration, policies, contracts, or new modules rather than modifying unrelated component internals.

### Small services

Services should remain focused and readable. Large workflows should be decomposed into orchestration and specialized services rather than creating large "God services."

### Documentation as an engineering artifact

Requirements, architecture, module contracts, implementation decisions, workflows, and testing strategies are documented alongside the system.

---

# 📦 Major Engineering Domains

SocialConnect is developed as a collection of cooperating application domains and reusable capabilities.

## 👤 Identity & User Profile

Identity is responsible for authentication and identity concerns.

Business profile information is maintained separately from the Identity representation.

`ApplicationUser` remains focused on Identity concerns, while business profile information belongs to the application domain.

---

## 📝 Post Creation

Post creation is implemented as an explicit application workflow:

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

The workflow keeps post creation orchestration separate from generic post persistence responsibilities.

---

## 🖼️ Media Management

Media processing follows a dedicated pipeline:

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
      ├───────────────┐
      ▼               ▼
Finalization      Assignment
      │               │
      ▼               ▼
Final Media      Entity Association
```

Finalization and assignment are separate responsibilities.

The MediaUploader is designed as a consumer-independent reusable component.

---

## 💬 Comments

The comment subsystem supports:

* Comment creation
* Replies
* Editing
* Threaded discussions
* Lazy loading
* Reaction integration
* Target-based ownership
* Reusable UI through `CommentManager`

---

## ❤️ Reactions

Reactions use a target-oriented mechanism that can operate across supported target types.

The target mechanism is based on:

```text
TargetType + TargetId
```

This allows supported features to share a consistent target resolution model.

---

## 🔁 Sharing

Internal sharing is represented through a relationship between posts rather than duplication of the original content or media.

```text
Original Post
      │
      ▼
SharedPostId
      │
      ▼
New Post
```

Shared content is resolved through the feed/application layer.

---

# 📣 Event & Notification Platform

SocialConnect contains a dedicated event and notification platform.

The architecture distinguishes domain events from integration/outbox events.

The overall processing model is:

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

The transactional outbox architecture includes:

* Reliable event persistence
* Dispatch status
* Retry handling
* Lease/claim processing
* Dead-letter handling
* Idempotency
* SQL Server-safe worker processing

Notification persistence is the source of truth.

Real-time mechanisms such as SignalR are treated as delivery mechanisms rather than the notification source of truth.

---

# 🚨 Alerting Platform

SocialConnect includes a generic alerting architecture intended to support operational conditions across multiple modules.

The alerting platform is deliberately designed so that individual operational conditions do not become a giant hard-coded switch inside a central alert service.

The conceptual boundary is:

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

Potential alert sources may include:

* Posts
* Comments
* Reactions
* Marketplace
* Orders
* Payments
* Media
* Administration
* Other operational modules

Each source is intended to integrate through the generic alerting contract rather than introducing source-specific behavior into the generic platform.

---

# 🛡️ Administration, Governance & Moderation

Administration is designed as a privileged subsystem inside the existing MVC application.

The architecture intentionally does not use ASP.NET Core Areas for Administration.

The Administration domain includes:

* Administrative governance
* Moderation
* Reporting
* Review and case management
* Decisions and actions
* Marketplace governance
* Notification configuration
* Event policy configuration
* Operational configuration
* Maintenance mode
* Auditing

Administrative operations use application/domain services and respect domain ownership rather than bypassing business rules through direct table manipulation.

---

# 🧱 Reusable UI Architecture

Reusable UI follows a consistent ViewComponent-based architecture:

```text
ViewComponent
      │
      ├── Defaults
      ├── ViewModel
      ├── Builder
      ├── Razor View
      ├── CSS
      ├── JavaScript Module
      └── Documentation
```

The Builder configures the ViewModel.

The ViewComponent renders.

JavaScript provides interaction and lifecycle management.

The global `site.js` foundation contains shared infrastructure such as:

```text
App.Events
App.Modules
Utilities
Bootstrap/Foundation
```

Feature-specific API and business logic does not belong in `site.js`.

---

# 📚 Documentation

The SocialConnect documentation is organized into **project-level documentation** and **module-level documentation**.

The root `README.md` is the primary documentation entry point for the repository.

The documentation structure is intentionally designed to grow incrementally as each architectural and implementation domain is finalized.

## Documentation Principles

Documentation should:

* Be maintained primarily in **Markdown**
* Reflect the current architectural contract
* Clearly distinguish requirements, architecture, implementation, and testing
* Be organized around stable project-level and module-level boundaries
* Preserve traceability between requirements and implementation
* Document important decisions for future maintenance and development
* Be added only when the corresponding content has been properly analyzed and finalized

---

# 📘 Project-Level Documentation

Project-level documentation describes the system as a whole.

It provides the foundation that individual module documentation builds upon.

### Requirements & Product Definition

* [Project Overview](docs/requirements/Project-Overview.md)
* [Product-Level Commerce SRS](docs/requirements/CommerceSRS.md)
* [Social Connect Feasibility](docs/requirements/Market-Value-and-Business-Model.md)
* [Software Requirements Specification](docs/requirements/SRS.md)

### System Architecture

- [System Architecture](docs/architecture/Architecture.md)
- [Detailed System Design](docs/architecture/Detailed-Design.md)
- [Domain Model Architecture](docs/architecture/DomainModelsArch.md)
* [Event & Notification Architecture](docs/architecture/Event-Notification-Architecture.md)
* [Alerting Architecture](docs/architecture/Alerting-Architecture.md)
* [Administration Architecture](docs/architecture/Administration-Architecture.md)
* [Administration  Panel SRS](docs/Modules/Administratioin/Admin-SRS.md)
* [SocialConnect Application User and Identity ](docs/Modules/ApplicationUser/AppUserModel.md)
* [SocialConnect Application User Profile](docs/Modules/UserProfile/UserProfileSRS.md)

### Cross-Cutting Engineering

Project-wide engineering documentation will cover shared mechanisms such as:

* Architecture conventions
* Shared contracts
* Reusable component conventions
* Media infrastructure
* Event and notification infrastructure
* Alerting infrastructure
* Security conventions
* Persistence conventions
* External integration boundaries

Additional documents will be linked here as they are finalized.

---

# 🧩 Module Documentation

Each major SocialConnect module will have its own documentation set.

Module documentation is intended to preserve both the **requirements** and the **engineering implementation knowledge** necessary to understand, maintain, extend, and audit the module in the future.

A module documentation set may contain:

```text
Module
│
├── Requirements
├── Architecture / Design
├── Implementation Contract
├── Application Workflows
├── Domain Rules
├── API Contract
├── UI / Component Documentation
├── Events & Notifications
├── Integration Boundaries
├── Testing
└── Implementation Notes
```

Not every module requires every document.

Only the documents relevant to the module should be created.

### Current Module Documentation

Module documentation will be added progressively as each module is analyzed, finalized, implemented, and validated.

Current and future module areas include:

* Identity & Profiles
* Feed
* Posts
* Post Creation
* Comments
* Reactions
* Sharing
* Media
* Notifications
* Events
* Alerting
* Marketplace
* Products
* Shops
* Orders
* Payments
* Administration
* Moderation
* Reporting

The links for each module will be added here when the corresponding documentation has been finalized.

---

# 🔧 Engineering & Implementation Documentation

Engineering documentation records implementation-level knowledge that is useful beyond a single feature.

Examples include:

* Reusable UI components
* Media pipeline
* Loader abstractions
* Context builders
* Application orchestration
* Repository and Unit of Work conventions
* Event processing
* Notification orchestration
* Recipient resolution
* Alert source integration
* External integration adapters
* Shared JavaScript module conventions

These documents complement the module documentation rather than replacing it.

---

# 🌐 API Documentation

API documentation will be maintained separately from the system-level requirements.

API documentation will describe finalized contracts including:

* Routes
* HTTP methods
* Request DTOs
* Response models
* Authorization requirements
* Validation rules
* Error behavior
* Ownership rules
* Integration contracts

Only finalized API contracts should be documented as authoritative.

---

# 🧪 Testing & Validation Documentation

Testing documentation will contain both project-wide strategy and module-specific validation.

### Project-Level Testing

* [Testing Strategy](docs/testing/Testing-Strategy.md)

### Module-Level Testing

Individual modules may contain documentation covering:

* Unit tests
* Integration tests
* Application workflow tests
* API tests
* UI/component validation
* Persistence tests
* Event/outbox tests
* Notification tests
* Alerting tests
* End-to-end workflows

Module-specific testing documentation will be linked from the relevant module documentation section.

---

# 🗄️ Database Documentation

Database documentation will describe finalized persistence architecture, including:

* Entity relationships
* Constraints
* Keys
* Indexes
* Ownership boundaries
* Soft-delete behavior
* Auditing
* Transactional requirements
* Module-specific persistence design

Database documentation will be updated as the corresponding domain/module contracts are finalized.

---

# 🔌 External Integrations

External integrations follow a strict adapter-boundary principle.

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

Application and domain services must not depend directly on external provider SDKs, payload models, endpoints, or authentication mechanisms.

Each external integration will receive its own documentation when implemented.

---

# 🌍 Localization, Globalization & Geographic Discovery

Geographic context is intended to be a shared capability across SocialConnect's social and marketplace domains.

The product direction includes:

* Country and city context
* Local discovery
* Regional marketplace behavior
* Broader/global discovery
* Locale-aware user experiences
* Future language and cultural localization
* Future currency, taxation, shipping, and regional commerce rules

Implemented capabilities will be documented separately from future globalization and marketplace capabilities.

---

# 📈 Development & Documentation Lifecycle

SocialConnect documentation follows the same engineering lifecycle as the software.

```text
Requirement
     │
     ▼
Analysis
     │
     ▼
Architecture
     │
     ▼
Design / Contract
     │
     ▼
Implementation
     │
     ▼
Validation / Testing
     │
     ▼
Documentation
     │
     ▼
Review / Refinement
```

For major modules, the objective is to preserve enough documentation that the implementation can be understood and maintained without relying solely on historical conversations or undocumented decisions.

---

# 🔄 Documentation Status

Documentation is continuously expanded.

A document may represent one of several states:

| Status          | Meaning                                                        |
| --------------- | -------------------------------------------------------------- |
| **Draft**       | Under analysis and subject to change                           |
| **Reviewed**    | Technically reviewed but not yet locked                        |
| **Locked**      | Architectural or implementation contract approved for use      |
| **Implemented** | Corresponding functionality implemented                        |
| **Validated**   | Implementation has been tested against its documented contract |
| **Superseded**  | Replaced by a newer authoritative document                     |

The repository should maintain **one authoritative document for each contract**.

Duplicate competing versions of the same architectural or requirements document should not be created.

---

# 🚀 Documentation Roadmap

The documentation repository will grow incrementally.

The general sequence is:

```text
Project Definition
       │
       ▼
System Architecture
       │
       ▼
Bounded Contexts / Module Boundaries
       │
       ▼
Module Requirements
       │
       ▼
Module Architecture / Design
       │
       ▼
Implementation Contract
       │
       ▼
API / UI / Event Contracts
       │
       ▼
Testing & Validation
```

As each document is finalized, its link will be added to the appropriate section of this README.

The README structure itself is intended to remain stable while its documentation index grows.

---

# 🖼️ Architecture & Design Diagrams

The repository contains visual architecture and design artifacts where appropriate.

Examples include:

* [Complete Blueprint](completeblueprint.png)
* [Complete Overview Blueprint / UML](completeoverviewblueprintuml.png)
* [SocialConnect Class Diagram](socialconnectclassdiagram.png)

Additional diagrams will be added as the corresponding architecture is finalized.

---

# 👨‍💻 Author

**Khalid Zada**

Computer Engineer | IT & Systems | Software Development | Infrastructure & Troubleshooting

GitHub: [@khalidzada](https://github.com/khalidzada)

---

# 📌 Repository Navigation

| Location             | Purpose                                                                         |
| -------------------- | ------------------------------------------------------------------------------- |
| `README.md`          | Primary project and documentation entry point                                   |
| `docs/requirements/` | Project-level requirements and product definition                               |
| `docs/architecture/` | System-wide architecture and cross-cutting architecture                         |
| `docs/engineering/`  | Engineering and implementation documentation                                    |
| `docs/api/`          | Finalized API contracts                                                         |
| `docs/testing/`      | Project-wide testing and validation documentation                               |
| `docs/modules/`      | Module-specific requirements, design, implementation, and testing documentation |

---

> **SocialConnect is a continuously evolving engineering project.**
>
> The documentation is maintained alongside the architecture and implementation so that important requirements, technical decisions, module contracts, and engineering practices remain visible, traceable, and understandable.
