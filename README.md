# SocialConnect

## Hybrid Social Network & E-Commerce Platform

**SocialConnect** is a long-term engineering project for designing and developing a modular social-network and e-commerce platform using the **Microsoft .NET ecosystem**.

The project explores the complete lifecycle of a modern application — from requirements and architecture through implementation, database design, APIs, reusable UI components, media processing, event-driven workflows, notifications, testing, and technical documentation.

> **Project status:** Active development and continuous architectural refinement.

---

## 🎯 Project Overview

SocialConnect combines social interaction and commerce capabilities within a single platform.

The platform is designed around concepts such as:

* User profiles and social relationships
* Posts and content publishing
* Comments and threaded discussions
* Reactions
* Sharing
* Media management
* Feed generation
* Notifications
* Event-driven processing
* E-commerce and marketplace capabilities
* Products, shops and vendors
* Administrative governance and moderation
* Auditing and soft deletion
* Extensible alerting
* API-based application integration

The project is being developed with an emphasis on **maintainability, separation of responsibilities, explicit architectural boundaries, reusable components, and production-oriented engineering practices**.

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
* Loader abstractions
* Context builders / orchestrators
* Event-driven architecture
* Transactional Outbox
* Notification orchestration
* Reusable UI components
* Manual mapping
* Automated testing and validation

---

# 🧭 Architecture Overview

The current architecture is intentionally designed around **clear ownership and separation of responsibilities**.

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
   └── External / Integration Infrastructure
```

Controllers remain thin and application orchestration owns the coordination of a use case.

The architecture deliberately avoids placing business workflows, repository access, or feature-specific responsibilities inside controllers and UI components.

---

# 🧩 Major Engineering Domains

The project has evolved into a collection of interconnected engineering domains.

## 👤 Identity & User Profile

Identity is responsible for authentication and identity concerns, while business profile information remains within the application domain.

---

## 📝 Post Creation

The post creation workflow is designed as an explicit application pipeline:

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

This keeps post creation orchestration separate from generic post persistence responsibilities.

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

Finalization and assignment remain separate responsibilities.

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

Reactions are implemented through a reusable target-oriented mechanism.

The architecture supports reaction operations across supported target types while keeping target resolution explicit.

---

## 🔁 Sharing

Internal sharing is represented as a relationship between posts rather than by duplicating the original post content or media.

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

# 📣 Event & Notification Architecture

SocialConnect contains a dedicated event and notification architecture.

The design distinguishes between:

```text
Domain Events
      │
      ▼
Integration / Outbox Events
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

The transactional outbox design includes concepts such as:

* Reliable event persistence
* Dispatch status
* Retry handling
* Lease / claim processing
* Dead-letter handling
* Idempotency
* SQL Server-safe worker processing

Notification persistence remains the source of truth, while real-time technologies such as SignalR are treated as delivery mechanisms rather than the notification system itself.

---

# 🚨 Alerting Architecture

The alerting platform is designed as a **generic and extensible capability**.

Operational conditions are not intended to become a giant hard-coded switch inside a central alert service.

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

This allows future modules to introduce operational alert sources without destabilizing the generic alerting infrastructure.

Potential source domains include:

* Posts
* Comments
* Reactions
* Marketplace
* Orders
* Payments
* Media
* Administration
* Other operational modules

---

# 🛡️ Administration, Governance & Moderation

Administration is designed as a privileged subsystem inside the existing MVC application.

The design intentionally avoids ASP.NET Core Areas.

The administration architecture includes concepts such as:

* Administrative governance
* Moderation
* Reporting
* Review / case management
* Decisions and actions
* Marketplace governance
* Notification configuration
* Event policy configuration
* Operational configuration
* Maintenance mode
* Auditing

Domain ownership remains enforced through application and domain services rather than direct administrative table manipulation.

---

# 🧱 Reusable UI Architecture

Reusable UI components follow a consistent structure.

A component can include:

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

The JavaScript architecture uses application modules with explicit lifecycle concepts such as:

```text
register
init
bind
destroy
```

Feature-specific API and business logic is kept outside the global `site.js` foundation.

---

# 📚 Documentation

The repository contains **two generations of documentation**:

1. **Current engineering documentation** — the architecture and implementation decisions that describe the evolving/current system.
2. **Original / historical project documentation** — the earlier project requirements, designs and planning documents that remain valuable as part of the project's development history.

This distinction is intentional.

> **Important:** historical documents should not automatically be interpreted as the current implementation contract.

---

# 📘 Current Engineering Documentation

New documentation should be added primarily in **Markdown (`.md`)**.

As the project continues to evolve, the current documentation index will grow.

### Architecture

* [Current Architecture](docs/architecture/Architecture.md)
* [Event & Notification Architecture](docs/architecture/Event-Notification-Architecture.md)
* [Alerting Architecture](docs/architecture/Alerting-Architecture.md)
* [Administration Architecture](docs/architecture/Administration-Architecture.md)

### Domain & Application Engineering

* [Media Architecture](docs/engineering/Media-Architecture.md)
* [Post Creation](docs/engineering/Post-Creation.md)
* [Recipient Resolution](docs/engineering/Recipient-Resolution.md)
* [Notification Module](docs/engineering/Notification-Module.md)

### Testing & Validation

* [Testing Strategy](docs/testing/Testing-Strategy.md)

> Additional documents will be added here as their implementation and architecture are finalized.

---

# 📜 Original / Historical Documentation

The repository's original HTML documentation is intentionally retained.

These documents provide valuable historical context for the evolution of SocialConnect.

## Requirements

* [Sprint Implementation](SprintImplementation.html)
* [Product Requirements Document](prd.html)
* [Software Requirements Specification](SRS.html)
* [API Specification](APISpecDoc.html)

## Design

* [High-Level Architecture](highlevelarchite.html)
* [Detailed Design](DetailDesignDoc.html)
* [Technical Design Document](TDD.html)
* [Wireframes](Wireframe.html)
* [Database Design](DbDesignDoc.html)

## Planning

* [Project Plan](projectplan.html)
* [DevOps Strategy](DevopStr.html)
* [Test Plan](testplan.html)
* [Test Cases](testcase.html)

## Supporting Documentation

* [Project Objectives](objectives.html)
* [Technology Overview](technology.html)

---

# 🌐 Visual Documentation Portal

The repository also contains an original browser-based documentation portal.

### [Open SocialConnect Documentation Portal](index.html)

The portal provides a graphical navigation layer for the original HTML documentation.

It organizes documentation into areas such as:

* Requirements
* Design
* Planning
* Architecture
* Project objectives
* Technology
* Testing

The original portal is preserved rather than replaced because it represents an important part of the project's documentation history.

---

# 🖼️ Architecture & Design Diagrams

The repository also contains visual architecture and design artifacts.

* [Complete Blueprint](completeblueprint.png)
* [Complete Overview Blueprint / UML](completeoverviewblueprintuml.png)
* [SocialConnect Class Diagram](socialconnectclassdiagram.png)

Additional diagrams will be added as the architecture develops.

---

# 🔄 Documentation Lifecycle

Documentation is treated as part of the engineering process rather than as an afterthought.

The preferred lifecycle is:

```text
Requirement
    │
    ▼
Architecture
    │
    ▼
Design
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
Architecture Refinement
```

When an architectural decision becomes important enough to affect future implementation, it should be documented explicitly.

---

# 🗂️ Recommended Repository Documentation Structure

As the repository grows, current documentation will gradually be organized under:

```text
docs/
│
├── architecture/
│   ├── Architecture.md
│   ├── Event-Notification-Architecture.md
│   ├── Alerting-Architecture.md
│   └── Administration-Architecture.md
│
├── engineering/
│   ├── Media-Architecture.md
│   ├── Post-Creation.md
│   ├── Recipient-Resolution.md
│   └── Notification-Module.md
│
├── api/
│
├── testing/
│   └── Testing-Strategy.md
│
└── historical/
```

The original HTML documents can remain at their current locations until there is a specific reason to reorganize them.

---

# 🧪 Engineering Principles

The project follows several core engineering principles:

### Explicit ownership

Every important operation should have a clear owner.

### Separation of responsibilities

Controllers, UI components, application orchestration, business services, persistence, and infrastructure should not become mixed responsibilities.

### Small services

Services should remain focused and readable rather than becoming large "God services".

### Explicit contracts

Important architectural decisions and application contracts should be defined before implementation.

### Server-first application state

The server remains responsible for authoritative application state. JavaScript primarily handles interaction and minimal client-side state.

### Reusable components

Feature UI should be designed as reusable components rather than duplicated page-specific implementations.

### Documentation as part of development

Architecture, implementation decisions, workflows, and important boundaries should be documented as the system evolves.

---

# 🛠️ Development Philosophy

SocialConnect is developed incrementally.

Architectural decisions are reviewed and locked before implementation where appropriate. Existing contracts should not be redesigned simply for stylistic reasons; changes should be driven by a genuine architectural requirement or contradiction.

The goal is not simply to make the application work.

The goal is to build a system that is:

* Understandable
* Maintainable
* Testable
* Extensible
* Explicitly documented
* Consistent in its architectural boundaries

---

# 📈 Project Evolution

SocialConnect began with traditional requirements, design, planning and feasibility documentation.

Over time, the project has evolved toward a more structured architecture with stronger separation between:

```text
Domain
Application
Infrastructure
Persistence
UI
Events
Notifications
Alerting
Administration
```

The repository therefore intentionally contains both **historical design material and newer engineering documentation**.

This makes the repository useful not only as a project archive, but also as a record of the system's architectural evolution.

---

# 🚀 Future Documentation

As development continues, this README will remain the **documentation index**.

New documents should be added here when they become sufficiently stable and useful.

Examples include:

* Domain architecture documents
* Module implementation contracts
* Event catalogs
* Notification workflows
* Alert source documentation
* API documentation
* Database documentation
* Testing documentation
* Deployment documentation
* Operational runbooks
* Component documentation
* Architecture decision records
* Implementation checkpoints

---

# 👨‍💻 Author

**Khalid Zada**

Computer Engineer | IT & Systems | Software Development | Infrastructure & Troubleshooting

GitHub: [@khalidzada](https://github.com/khalidzada)

---

## 📌 Repository Navigation

| Area         | Purpose                                     |
| ------------ | ------------------------------------------- |
| `README.md`  | Current GitHub documentation entry point    |
| `docs/`      | Current engineering documentation           |
| `index.html` | Original visual documentation portal        |
| `*.html`     | Original / historical project documentation |
| `*.png`      | Architecture and design diagrams            |

---

> **SocialConnect is a continuously evolving engineering project.**
>
> The documentation is maintained alongside the architecture and implementation so that important technical decisions remain visible, traceable, and understandable.
