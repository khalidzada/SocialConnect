# SocialConnect — Application User Governance & Final Requirements

**Document Status:** Canonical / Final Requirements Contract
**Document Type:** Application User, Account & Governance Requirements
**System:** SocialConnect
**Platform:** ASP.NET Core MVC / .NET 8
**Identity:** ASP.NET Core Identity
**Architecture:** Layered Modular Monolith
**Persistence:** Entity Framework Core / SQL Server

---

# 1. Purpose

This document defines the final requirements for the **SocialConnect Application User** from the perspective of the complete platform.

It defines how SocialConnect users:

* register;
* authenticate;
* verify their identity;
* manage credentials;
* manage account security;
* maintain their profile;
* participate socially;
* buy products;
* operate shops;
* sell products;
* follow shops;
* save products;
* interact with other users;
* report content;
* receive notifications;
* manage their account lifecycle;
* become subject to platform governance;
* become administrators or moderators where authorized;
* are suspended, restricted, reactivated, or otherwise governed;
* delete or deactivate their account;
* interact with the marketplace;
* interact with platform policies and security controls.

The document also establishes the distinction between:

```text
Identity Account
Business User Profile
Platform Role
Business Capability
Governance Authority
```

These concepts must not be collapsed into one model or one service.

This document is intended to be the **implementation baseline for the complete SocialConnect Application User lifecycle**.

---

# 2. Architectural Position of the Application User

The SocialConnect `ApplicationUser` is an ASP.NET Core Identity entity.

It is intentionally **outside the SocialConnect domain entity inheritance hierarchy**.

The canonical model is:

```text
ASP.NET Core Identity
        │
        ▼
ApplicationUser
        │
        │ 1 : 1
        ▼
UserProfile
        │
        ▼
SocialConnect Business Domain
```

The SocialConnect domain hierarchy remains:

```text
BaseEntity
    ↓
AuditableEntity
    ↓
SoftDeleteEntity
    ↓
Reactable
```

where applicable.

Therefore:

```text
ApplicationUser
    ≠
BaseEntity
```

and:

```text
ApplicationUser
    ≠
Reactable
```

even though `ApplicationUser` may have navigation relationships to domain entities.

---

# 3. Application User Concept

An Application User represents a person/account that can interact with SocialConnect.

The account consists conceptually of several distinct dimensions:

```text
ApplicationUser
│
├── Identity
├── Authentication
├── Security
├── Account State
├── Verification
├── Roles
├── Capabilities
├── UserProfile
├── Social Relationships
├── Marketplace Participation
└── Governance State
```

These dimensions must remain conceptually separate.

---

# 4. ApplicationUser Responsibilities

`ApplicationUser` is responsible primarily for account and Identity concerns.

The model may contain Identity-related and account-level information such as:

```csharp
public class ApplicationUser : IdentityUser
{
    public string FirstName { get; set; } = string.Empty;

    public string LastName { get; set; } = string.Empty;

    [NotMapped]
    public string FullName => $"{FirstName} {LastName}";

    public bool IsActive { get; set; } = true;

    public bool IsVerified { get; set; } = false;

    public DateTime RegisteredAt { get; set; } = DateTime.UtcNow;

    public DateTime? LastLoginAt { get; set; }

    public virtual UserProfile? Profile { get; set; }
}
```

The final implementation must preserve the distinction between Identity responsibilities and SocialConnect business responsibilities.

---

# 5. ApplicationUser Must Not Become the Entire User Domain

`ApplicationUser` must not become a God entity.

Business concepts must remain in their appropriate models.

For example:

```text
ApplicationUser
    → authentication/account

UserProfile
    → profile/business user information

Post
    → post/content

Comment
    → comment/discussion

Reaction
    → reaction

Shop
    → merchant/shop

Product
    → product

Order
    → transaction/order

Notification
    → notification

ContentReport
    → report

ModerationCase
    → moderation
```

The user identity connects these concepts, but does not own their internal business rules.

---

# 6. UserProfile Boundary

The relationship is:

```text
ApplicationUser
       │
       │ one-to-one
       ▼
UserProfile
```

`ApplicationUser` represents:

> Who is authenticated?

`UserProfile` represents:

> What SocialConnect profile information does this user have?

The two must not be merged simply because they represent the same person.

---

# 7. First Registration / First Authentication

SocialConnect must support normal Identity authentication and external authentication.

Initial authentication may originate from:

```text
Email/password
Google
Facebook
```

The exact provider configuration belongs to the Identity/authentication implementation.

The application-level workflow must ensure that a successful first authentication creates or establishes the corresponding SocialConnect account state.

The intended sequence is:

```text
Authentication
    ↓
Identity account established
    ↓
ApplicationUser established
    ↓
UserProfile ensured
    ↓
Initial account state established
    ↓
User enters SocialConnect
```

---

# 8. First Successful Authentication Must Create an Empty Profile

On first successful authentication, SocialConnect must ensure that an associated `UserProfile` exists.

The profile may initially be empty.

The user may then complete or update it.

The platform must not require the user to provide every optional profile attribute during registration unless a specific security/business policy explicitly requires it.

Canonical principle:

```text
Successful account creation
        ↓
UserProfile exists
        ↓
Profile completion can occur later
```

---

# 9. Registration State

A newly registered user has a defined initial lifecycle.

Conceptually:

```text
REGISTERING
    ↓
AUTHENTICATED
    ↓
VERIFICATION PENDING / VERIFIED
    ↓
ACTIVE
```

The exact persisted representation must be implemented using the established Identity/account properties and policies rather than introducing unnecessary duplicate state.

---

# 10. User Account States

The final account-governance model must distinguish relevant account states.

At minimum, the platform must be able to distinguish:

```text
Active
Inactive / Deactivated
Suspended
Deleted / Deletion Requested
Verification Pending
```

Not every state must necessarily become a separate enum property.

Where Identity already provides an appropriate mechanism, that mechanism should be used.

The important requirement is that the application can reliably determine:

```text
Can authenticate?
Can access platform?
Can perform social actions?
Can buy?
Can sell?
Can operate a shop?
Can administer?
```

---

# 11. IsActive

The existing:

```csharp
public bool IsActive { get; set; } = true;
```

represents the user's application-level active state.

It must not be treated as a universal replacement for:

* Identity lockout;
* suspension;
* deletion;
* email verification;
* authorization;
* role membership.

Each concept has its own meaning.

---

# 12. Verification

SocialConnect distinguishes account authentication from verification.

Conceptually:

```text
Authenticated
    ≠
Email Verified
    ≠
Platform Verified
```

The final implementation must define and enforce the relevant verification policies.

`IsVerified` must not be interpreted as automatically meaning every possible verification type.

---

# 13. Email Verification

Email verification is part of account security.

The platform must support verification of the user's email address.

Where platform policy requires verified email for an operation, the application service must enforce that requirement.

Examples of operations that may be subject to verification policy include:

```text
Account-sensitive operations
Marketplace participation
Seller/Vendor activation
Financial operations
Security-sensitive actions
```

The final policy for each operation must be explicitly documented by its owning module.

---

# 14. Authentication

SocialConnect authentication is handled through ASP.NET Core Identity and configured external providers.

The application must support:

```text
Local Identity authentication
Google authentication
Facebook authentication
```

Authentication must establish the authenticated principal.

Business authorization must then determine what that user is allowed to do.

Therefore:

```text
Authentication
    → Who are you?

Authorization
    → What are you allowed to do?
```

---

# 15. External Authentication

External providers are authentication mechanisms, not SocialConnect business roles.

For example:

```text
Google
Facebook
```

must not become business-domain dependencies.

The application must translate successful external authentication into the normal SocialConnect account lifecycle.

---

# 16. Credential Management

Users must be able to manage their own supported credentials through the appropriate account/security workflows.

This includes, where applicable:

* password management;
* password change;
* password reset;
* external login management;
* credential activation/deactivation where supported;
* two-factor authentication;
* recovery/security mechanisms.

Controllers must not directly manipulate Identity persistence.

---

# 17. Two-Factor Authentication

SocialConnect must support two-factor authentication through ASP.NET Core Identity's established security mechanisms.

Users must be able to:

* enable 2FA;
* disable 2FA;
* configure supported authentication methods;
* manage recovery mechanisms where provided;
* view relevant security state.

Security-sensitive changes must require appropriate authentication/re-authentication where required by the security policy.

---

# 18. Login Tracking

The current model includes:

```csharp
public DateTime? LastLoginAt { get; set; }
```

The platform must update this value according to the finalized authentication/login workflow.

It must represent the most recent successful login recognized by SocialConnect.

The value is informational/account-management state and must not replace Identity security mechanisms.

---

# 19. User Self-Service Account Management

An ordinary user must be able to manage their own account within the permissions granted by platform policy.

Self-service capabilities include:

```text
Profile management
Credential management
Security settings
2FA
External login management
Account activation/deactivation where supported
Account deletion request
Privacy-related account controls
```

Every operation must pass through the appropriate application service.

---

# 20. Account Deactivation

A user may deactivate their account where the account policy permits self-service deactivation.

Deactivation means:

```text
Account remains known to SocialConnect
but normal user participation is disabled.
```

It must not automatically mean:

```text
Database deletion
```

or:

```text
Soft deletion of every related domain record
```

The consequences of deactivation must be explicitly defined for:

* authentication;
* posts;
* comments;
* reactions;
* friendships;
* shop ownership;
* products/listings;
* orders;
* payments;
* notifications;
* saved products;
* followed shops.

No cascading business behavior should be invented by the account service.

---

# 21. Account Deletion

The platform must support account deletion according to the final account/legal/data-retention policy.

Account deletion is a governed workflow, not a direct database delete.

Conceptually:

```text
User requests deletion
        ↓
Validation / security checks
        ↓
Account deletion policy
        ↓
Domain-specific consequences
        ↓
Required retention handling
        ↓
Identity/account state transition
        ↓
Audit where applicable
        ↓
Events where applicable
```

The system must not assume that deleting an Identity row means that every related SocialConnect record should be physically deleted.

---

# 22. Administrative Self-Deletion Restriction

Admin and Moderator accounts must not use the normal self-service account deletion workflow to delete themselves.

The system must prevent this through the appropriate application authorization/governance boundary.

This rule exists independently of UI hiding.

The server-side application service must enforce it.

---

# 23. User Roles

The canonical platform roles are:

```text
Admin
Moderator
Vendor
User
```

There is no V1 `SuperAdmin` role.

The system must not introduce a parallel user class for each role.

---

# 24. Roles vs Capabilities

A role identifies an authorization/governance position.

A capability identifies what the user is allowed to do.

The authorization model is:

```text
System Role
    ↓
Administrative Capability / Permission
    ↓
Operation
```

For ordinary commerce/social participation, capabilities may be derived from account state, marketplace state, ownership, policy, and role.

---

# 25. User Perspective Model

Every Application User should be understood through several possible perspectives.

```text
                    Application User
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
       ▼                   ▼                    ▼
   Social User          Buyer              Seller/Vendor
                                               │
                                               ▼
                                             Shop
                                               │
                           ┌───────────────────┘
                           ▼
                    Admin / Moderator
                    where authorized
```

These are **perspectives and capabilities**, not separate identity entities.

One person/account can participate in multiple perspectives over time.

---

# 26. Ordinary User Perspective

The ordinary `User` is the baseline SocialConnect participant.

A User may:

* maintain a profile;
* create posts;
* comment;
* react;
* share;
* follow shops;
* save products;
* report content;
* communicate through supported platform mechanisms;
* discover local/social content;
* buy products;
* become eligible for vendor/shop capabilities;
* manage their own account and security.

All actions remain subject to:

```text
Account State
Authorization
Content Policy
Community Policy
Marketplace Policy
Security Policy
```

---

# 27. Social User Governance

A social user operates within community rules.

Governed operations include:

```text
Create content
Edit content
Delete content
Comment
Reply
React
Share
Block users
Follow shops
Report content
```

The owning modules define the detailed business rules.

The user-governance layer provides the account identity and authorization context.

---

# 28. Buyer Perspective

A buyer is an Application User exercising marketplace purchasing capabilities.

A buyer may:

```text
Browse products
Search products
Filter by location
Inspect product details
Save products
Contact seller through supported mechanisms
Create orders
Make supported payments
Track orders
Receive order notifications
Complete/confirm applicable delivery lifecycle
```

Actual availability depends on:

```text
Account state
Marketplace eligibility
Product availability
Seller/listing state
Payment state
Order state
Platform policy
```

---

# 29. Buyer Governance

Buyer operations are governed by marketplace rules.

The buyer must not be able to:

* purchase unavailable inventory;
* bypass payment requirements;
* manipulate another user's order;
* modify seller-owned inventory;
* alter marketplace financial records;
* bypass order authorization.

Buyer actions must pass through the appropriate marketplace/order/payment application services.

---

# 30. Seller/Vendor Perspective

A Vendor is an Application User with the appropriate marketplace capability and authorization to operate commercial functionality.

A Vendor may, subject to eligibility:

```text
Create/manage a Shop
Create/manage products
Create/manage listings
Manage prices
Manage inventory
Manage warehouse/location information
Manage orders
Configure supported payment methods
View seller-related information
```

Vendor capability is not equivalent to simply having an account.

---

# 31. Vendor Eligibility

Vendor activation may depend on platform policies such as:

```text
Verified email
Account active
Required profile information
Marketplace acceptance
Payment setup
Required business information
Platform eligibility
```

The exact conditions belong to the Marketplace Governance contract.

The user-governance layer must provide the account/security foundation needed by that contract.

---

# 32. Shop Ownership

A Shop belongs to an owner.

The current architecture establishes:

```text
ApplicationUser
    1
    │
    └── many
          Shop
```

with the current persistence relationship using restricted deletion.

This relationship must be preserved unless the final Shop model contract explicitly changes it.

A user account deletion/deactivation workflow must therefore consider shop ownership before completing account lifecycle changes.

---

# 33. Vendor Account Governance

Vendor operations are subject to both:

```text
User Governance
        +
Marketplace Governance
```

Therefore a vendor can be:

```text
authenticated
but not marketplace eligible
```

or:

```text
marketplace eligible
but currently suspended from selling
```

These states must not be represented by one generic `IsActive` flag.

---

# 34. Admin Perspective

An Admin is an Application User with privileged administrative authorization.

Admin functionality is not a different user entity.

The Admin uses the same authenticated Identity account but receives additional capabilities through authorization.

Admin capabilities include appropriate access to:

```text
Dashboard
Account Governance
Role Governance
Reporting
Moderation
Moderation Policy
Marketplace Governance
Platform Configuration
Maintenance Operations
Event Policy
Notification Channel Policy
Security Administration
Audit
```

Each capability must be explicitly authorized.

---

# 35. Moderator Perspective

A Moderator is an Application User with limited administrative authority.

A Moderator does not automatically have every Admin capability.

Moderator permissions must be explicitly defined.

Typical responsibilities include governed access to:

```text
Reports
Moderation Cases
Content Review
Moderation Decisions
Permitted Moderation Actions
Escalation
```

A Moderator must not bypass administrative capability boundaries.

---

# 36. Admin/Moderator Authorization

The authorization model is:

```text
Authenticated User
        ↓
System Role
        ↓
Administrative Capability
        ↓
Specific Operation
        ↓
Target Authorization
```

This prevents:

```text
Role = Admin
    →
unrestricted arbitrary database access
```

Authorization must also consider the target being operated upon.

---

# 37. Target Authorization

Where an administrative or governance operation targets a SocialConnect entity, the operation may use:

```text
TargetType + TargetId
```

to identify the target.

Conceptually:

```text
Admin/Moderator
      ↓
Governance Operation
      ↓
TargetType + TargetId
      ↓
Target Resolver
      ↓
Target Entity
      ↓
Authorization / Policy
      ↓
Domain/Application Operation
```

The target mechanism identifies the target.

It does not grant permission to modify it.

---

# 38. Domain Ownership During Governance

Administration and moderation must respect domain ownership.

The following is prohibited:

```text
Admin Controller
    ↓
DbContext
    ↓
Post table
    ↓
direct UPDATE
```

The correct architecture is:

```text
Admin
    ↓
Admin Application Service
    ↓
appropriate domain/application operation
    ↓
transaction
    ↓
state change
    ↓
audit/event where applicable
```

The same principle applies to:

```text
Comment
Reaction
Media
Product
Shop
Order
Payment
```

and every other business domain.

---

# 39. Account Suspension

Suspension is a governed account-state operation.

A suspension must have a defined:

```text
Target User
Reason
Initiating Authority
Start
Duration / End where applicable
Scope
Consequences
Audit
Event
Notification policy
```

Suspension must not simply be:

```text
IsActive = false
```

without recording the governance context required by the final policy.

---

# 40. Suspension Consequences

The exact consequences depend on the suspension type/policy.

Potentially governed capabilities include:

```text
Login
Post creation
Commenting
Reactions
Messaging
Marketplace buying
Marketplace selling
Shop management
Order operations
Administrative access
```

The system must evaluate the affected capability rather than assuming every suspension has identical consequences.

---

# 41. Role Suspension vs Account Suspension

The architecture must distinguish between:

```text
Account suspension
```

and:

```text
Marketplace/vendor restriction
```

and:

```text
Administrative capability restriction
```

For example:

```text
User remains active
but vendor capability is suspended.
```

This must not require disabling the entire Identity account.

---

# 42. Role Membership Governance

Admin-authorized workflows may govern role membership according to policy.

The role system must support the canonical roles:

```text
Admin
Moderator
Vendor
User
```

Role changes must be:

* authorized;
* validated;
* auditable;
* concurrency-safe;
* performed through the appropriate application service.

No controller should directly manipulate Identity role tables.

---

# 43. Admin and Moderator Role Protection

Administrative role changes require stronger authorization than ordinary profile updates.

The platform must prevent unauthorized escalation such as:

```text
User
    ↓
self-assign Admin
```

or:

```text
Vendor
    ↓
self-assign Moderator
```

Role assignment must pass through the administrative authorization boundary.

---

# 44. User Blocking

Users may block other users according to the SocialConnect relationship policy.

The existing model contains:

```text
SentBlocks
ReceivedBlocks
```

The blocking workflow belongs to the user/social relationship domain.

Blocking must have defined consequences for applicable operations such as:

```text
Content visibility
Interaction
Messaging
Social discovery
Profile access
```

The exact behavior must be enforced by the owning domain services.

---

# 45. Friendships / Social Relationships

User-to-user relationships are business-domain concepts and must not be encoded as Identity roles or account properties.

Examples include:

```text
Friendship
UserBlock
```

These remain domain entities with their own lifecycle and business rules.

`ApplicationUser` provides identity.

The relationship entities provide social meaning.

---

# 46. User-Owned Content

A user may own business entities such as:

```text
Post
Comment
Reaction
Shop
SavedProduct
```

Ownership must be explicit.

Where a direct FK relationship is appropriate, a normal foreign key must be used.

Where a polymorphic target is genuinely required, use:

```text
TargetType + TargetId
```

---

# 47. User and Reactions

Users may create reactions on supported Reactable entities.

The relationship is conceptually:

```text
ApplicationUser
       │
       ▼
Reaction
       │
       ▼
Reactable Target
```

The Reaction module owns:

* reaction validation;
* uniqueness;
* replacement/removal;
* aggregate updates;
* reaction events.

The user-governance layer supplies identity and authorization context.

---

# 48. User and Posts

Users may create Posts according to the Post Creation contract.

The canonical workflow remains:

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

The Application User is the authenticated actor/owner context.

The User service must not absorb Post Creation business logic.

---

# 49. User and Comments

Users may create comments/replies subject to:

```text
Post/Target policy
Comment policy
Account state
Block relationships
Authorization
```

Comment ownership belongs to the Comment domain.

The Comment system uses:

```text
Comment.UserId
```

for the comment author/owner relationship.

---

# 50. User and Notifications

Users may receive notifications generated by supported platform events.

Notification persistence is the source of truth.

The user identity provides the recipient.

Conceptually:

```text
Business Event
    ↓
Recipient Resolution
    ↓
ApplicationUser
    ↓
Notification persistence
    ↓
Delivery planning
```

The Notification system must not duplicate account identity.

---

# 51. User Notification Preferences

Users may have notification preferences according to the final Notification Policy.

Preferences must distinguish:

```text
Notification type
Channel
User preference
Platform policy
Mandatory/system notification
```

User preferences must not override platform-required security or legal notifications where those are mandatory.

---

# 52. User Reporting

Users must be able to report supported content/accounts according to the Reporting contract.

The conceptual workflow is:

```text
User
    ↓
Report
    ↓
Review / Case
    ↓
Decision
    ↓
Action
```

Reporting is distinct from moderation.

A user submitting a report does not directly impose a moderation action.

---

# 53. User Reports and TargetType

Reports may target different SocialConnect entities.

Where polymorphic targeting is appropriate:

```text
TargetType + TargetId
```

is used.

Example:

```text
TargetType = Post
TargetId   = <Post.Id>
```

or:

```text
TargetType = Comment
TargetId   = <Comment.Id>
```

The report service resolves and validates the target.

---

# 54. Moderation

Moderation is a privileged governance process.

A user report does not automatically mean that content is removed.

The canonical lifecycle is:

```text
Report
    ↓
Moderation Review / Case
    ↓
Decision
    ↓
Action
```

Possible governed actions include:

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

The permitted action depends on the applicable Moderation Policy.

---

# 55. User Warnings

A user may receive a formal platform warning as a moderation outcome.

A warning must be attributable to:

```text
User
Reason / policy
Moderation decision
Authority
Timestamp
```

Where notification is required, the canonical Notification platform must be used.

---

# 56. Marketplace Governance

Marketplace participation has additional governance requirements.

The system must separately govern:

```text
Buyer eligibility
Vendor eligibility
Shop eligibility
Listing eligibility
Inventory
Orders
Payments
Seller restrictions
Marketplace policy
```

A user can remain a normal SocialConnect user even if marketplace privileges are restricted.

---

# 57. Buyer Restriction

A marketplace restriction may affect buyer capabilities without disabling the complete SocialConnect account.

For example:

```text
ApplicationUser
    ACTIVE

Social capabilities
    AVAILABLE

Buyer capability
    RESTRICTED
```

The actual policy must be defined by Marketplace Governance.

---

# 58. Vendor Restriction

Likewise:

```text
ApplicationUser
    ACTIVE

Social capabilities
    AVAILABLE

Vendor capability
    SUSPENDED
```

The vendor restriction must be enforced by marketplace services.

The account service must not implement inventory, listing, order, or payment business rules.

---

# 59. Payment and User Governance

Payment-related operations are highly sensitive.

Users must not be able to:

* modify another user's payment information;
* bypass payment authorization;
* alter transaction settlement;
* manipulate platform commission;
* access another user's financial data.

Payment operations belong to the Payment/Order domain and its authorized providers.

The user-governance layer supplies identity and authorization context.

---

# 60. Privacy and Data Ownership

A user owns control over their own account information subject to:

```text
Platform policy
Legal/retention requirements
Security requirements
Marketplace transaction requirements
Audit requirements
```

Administrative users must only access information necessary for their authorized operation.

Sensitive information must not be exposed merely because a user has an administrative role.

---

# 61. User Profile Management

Users must be able to manage their own profile through appropriate application services.

Profile management may include:

```text
Name
Profile information
Location
Profile media
Other supported profile fields
```

Profile-specific business information belongs to `UserProfile`, not to arbitrary controllers or unrelated domain entities.

---

# 62. Location

Location is an important SocialConnect user capability.

User profile location may participate in:

```text
Social discovery
Feed personalization
Local discovery
Marketplace discovery
Product discovery
Shop discovery
Neighborhood features
```

Location information must be handled according to the profile/privacy contract.

The platform must not expose precise location merely because a coarse location is available.

---

# 63. User Privacy

Users must have appropriate controls over profile information according to the final privacy policy.

The platform must distinguish between:

```text
Public profile information
Authenticated-user information
Owner-only information
Administrative information
Security-sensitive information
```

Authorization must be enforced server-side.

---

# 64. User-Owned Saved Products

Users may save products.

The existing model includes:

```text
SavedProducts
```

The Saved Product capability belongs to the marketplace/social discovery domain.

It must not be implemented as an arbitrary collection directly inside `ApplicationUser`.

---

# 65. User-Followed Shops

Users may follow shops.

The existing model includes:

```text
FollowedShops
```

The relationship is represented by the appropriate `ShopFollow` domain entity.

Conceptually:

```text
ApplicationUser
    ↓
ShopFollow
    ↓
Shop
```

Shop-follow business rules belong to the Shop/social marketplace domain.

---

# 66. Account Deletion and Owned Business Data

Before completing account deletion, SocialConnect must determine the user's outstanding domain ownership.

Examples:

```text
Owned Shops
Owned Products/Listings
Open Orders
Financial Transactions
Posts
Comments
Reports
Notifications
Media
Social Relationships
```

The account workflow must not arbitrarily delete all related records.

Each domain must define its own lifecycle consequences.

---

# 67. Account Deletion Orchestration

The final architecture should therefore treat account deletion as an orchestration process:

```text
User Deletion Request
        ↓
Account Governance Service
        ↓
Determine applicable domain consequences
        ↓
Invoke domain/application operations
        ↓
Identity/account transition
        ↓
Audit
        ↓
Events / Outbox where applicable
        ↓
Notification where applicable
```

The account service coordinates.

It does not become the owner of every domain.

---

# 68. Auditability

Governed user operations must be auditable where policy requires.

Examples include:

```text
Role changes
Account suspension
Account reactivation
Administrative changes
Security-sensitive operations
Moderation actions
Marketplace governance actions
Platform configuration changes affecting users
```

Audit records must identify, where applicable:

```text
Actor
Target
Operation
Timestamp
Reason
Result
```

The audit mechanism must remain immutable through ordinary administrative UI.

---

# 69. Actor vs Target

The governance model distinguishes:

```text
Actor
```

from:

```text
Target
```

Example:

```text
Moderator A
    ↓
suspends
    ↓
User B
```

Here:

```text
Actor = Moderator A
Target = User B
```

For entity-specific operations, the target may be represented through:

```text
TargetType + TargetId
```

This distinction is essential for auditing.

---

# 70. Self-Action vs Administrative Action

The system must distinguish:

```text
Self-service action
```

from:

```text
Administrative action
```

Example:

```text
User changes own profile
```

is different from:

```text
Admin changes another user's account status
```

The authorization and audit requirements are therefore different.

---

# 71. User Account Governance Flow

A normal self-service operation follows:

```text
Authenticated User
    ↓
User UI
    ↓
HTTP Request
    ↓
Authentication / Authorization
    ↓
Validation
    ↓
User Application Service
    ↓
Identity / Domain Service
    ↓
Transaction where required
    ↓
Audit/Event where required
    ↓
Result
```

---

# 72. Administrative User Governance Flow

An administrative operation follows:

```text
Admin / Moderator
    ↓
Admin UI
    ↓
HTTP Request
    ↓
Authorization
    ↓
Validation
    ↓
Admin Application Service
    ↓
Appropriate Domain/Application Service
    ↓
Transaction
    ├── State Change
    ├── Audit
    └── Event + Outbox where applicable
    ↓
Commit
    ↓
Dispatcher
    ↓
Handler
    ↓
Notification / Other Effects
```

---

# 73. No Direct Table Governance

The following architecture is prohibited:

```text
Controller
    ↓
DbContext
    ↓
Users / Posts / Products / Orders
```

for business governance operations.

The correct architecture is:

```text
Controller
    ↓
Application Service
    ↓
Domain/Application Owner
    ↓
Persistence
```

This preserves domain ownership.

---

# 74. User Repository Boundary

The specialized `IUserRepository/UserRepository` exists because `ApplicationUser` is an Identity entity outside the generic SocialConnect domain hierarchy.

The repository must remain focused on appropriate persistence/user retrieval responsibilities.

It must not become responsible for:

```text
Post creation
Comment creation
Marketplace operations
Notification orchestration
Moderation
Administration
Account business workflows
```

Those belong to their respective application/domain services.

---

# 75. Identity Manager Boundary

Identity-specific operations should use the appropriate ASP.NET Core Identity abstractions such as:

```text
UserManager<ApplicationUser>
SignInManager<ApplicationUser>
RoleManager
```

where applicable.

The application must not directly manipulate Identity tables when an Identity manager or application service is the correct boundary.

---

# 76. User Queries

User discovery and querying must be separated from mutation workflows.

Queries may include:

```text
Get user
Search users
Find active users
Find verified users
Find users by country
Find users by city
Paginate users
Load user profile
```

The final implementation must determine which queries belong to:

```text
Repository
Loader
Lookup Service
Query Service
```

rather than accumulating all queries in one specialized repository.

---

# 77. Tracking Rules

The established persistence convention remains:

```text
Normal queries
    → no-tracking where appropriate

Update workflows
    → tracked entities
```

The exact tracking requirement depends on the operation.

Read-only user discovery must not unnecessarily create tracked entity graphs.

---

# 78. DTO Boundary

User-facing query results should use DTOs where a service boundary requires them.

For example:

```text
UserSummaryDto
```

is a transport contract.

It is not the domain entity.

Manual mapping remains the project standard.

AutoMapper is not introduced.

---

# 79. ViewModel Boundary

The UI receives ViewModels rather than domain entities where presentation state is required.

The canonical flow is:

```text
User Application Service
    ↓
Query / Loader
    ↓
Builder / Resolver
    ↓
User ViewModel
    ↓
Razor View / ViewComponent
```

---

# 80. User Interface Architecture

User account UI follows the existing SocialConnect MVC architecture.

Standard page structure:

```text
MVC View
    ├── ViewComponents
    ├── Partial Views
    └── TagHelpers
```

Controllers remain thin.

Business operations remain server-side.

---

# 81. User JavaScript

User-related JavaScript must follow the established module registration architecture.

Feature JavaScript belongs to the user/account feature.

It must not be placed into `site.js` unless it is genuine shared foundation functionality.

Client-side JavaScript may handle:

```text
Interaction
UI state
Form enhancement
API invocation
Client validation assistance
Events
```

It must not own authoritative business rules.

---

# 82. Security Boundary

Every user operation must be evaluated against:

```text
Authentication
Authorization
Account State
Verification State
Role
Capability
Target Ownership
Target Authorization
Policy
Concurrency
Audit Requirement
```

The existence of a UI control is never considered authorization.

---

# 83. Ownership Authorization

For self-service operations:

```text
Current User
    ==
Resource Owner
```

must normally be established by the server.

A user must not be able to change ownership identifiers in a request to gain access to another user's resource.

Example:

```text
POST /profile/update
```

must use the authenticated user's identity rather than trusting:

```text
UserId = arbitrary request value
```

---

# 84. Administrative Target Authorization

Administrative authorization must verify:

```text
Actor
+
Role
+
Capability
+
Target
+
Policy
```

An Admin/Moderator must not automatically receive unrestricted authority over every target type.

---

# 85. Concurrency

User governance operations must consider concurrency.

Examples:

```text
Two administrators change the same user
User changes security settings while admin suspends account
Vendor status changes while marketplace governance operation occurs
Role membership changes concurrently
```

Where appropriate, the persistence/application layer must detect conflicting updates.

---

# 86. Security-Sensitive Operations

Security-sensitive user operations should require stronger controls where appropriate.

Examples:

```text
Password change
Email change
2FA changes
External login changes
Account deletion
Administrative role changes
Security recovery changes
```

The exact re-authentication requirements belong to the Security/Identity implementation contract.

---

# 87. Notification of Governance Actions

Where policy requires, governed account actions may generate notifications.

Examples:

```text
Account suspended
Account reactivated
Role changed
Vendor access changed
Moderation warning
Marketplace restriction
Security-sensitive change
```

The notification must use the existing canonical Event/Outbox/Notification architecture.

No parallel user-notification infrastructure may be introduced.

---

# 88. Event Integration

User governance operations that produce cross-module effects must use the canonical event architecture.

Conceptually:

```text
User Governance Operation
    ↓
Domain/Application State Change
    ↓
Event
    ↓
Transactional Outbox
    ↓
Dispatcher
    ↓
Handler
    ↓
Recipient Resolution
    ↓
Notification / Other Effects
```

The event system remains separate from direct synchronous UI behavior.

---

# 89. Account Governance Events

The final event catalog must determine the exact events, but likely governed lifecycle concepts include:

```text
UserRegistered
UserVerified
UserActivated
UserDeactivated
UserSuspended
UserReactivated
UserRoleChanged
UserDeleted / AccountDeletionCompleted
SecurityStateChanged
```

The exact event names and payload contracts must be finalized in the Event module contract.

This document defines the **business requirement**, not the event schema itself.

---

# 90. Alerting Integration

User governance conditions may become Alert sources where operational monitoring is required.

The generic Alerting platform must remain stable.

User-specific conditions must plug into the Alert source/policy boundary.

The User Governance module must not create a giant hard-coded alert switch.

---

# 91. Administration Integration

Administration consumes the finalized User Governance capabilities.

It may provide administrative operations such as:

```text
Search User
Inspect User
Inspect account state
Change permitted status
Suspend
Reactivate
Manage permitted role membership
Review history
Review reports
Review moderation state
Review marketplace governance state
```

These operations must use the appropriate application/domain services.

---

# 92. Moderator Integration

Moderators receive only the capabilities explicitly assigned to them.

A Moderator may perform permitted moderation operations but must not automatically gain:

```text
Full account administration
Role escalation
Platform configuration
Security administration
Financial administration
```

unless explicitly authorized by the final capability contract.

---

# 93. Admin Integration

Admins receive broader administrative capabilities according to the Administration Authorization Contract.

Even Admins remain subject to:

```text
Audit
Authorization
Target validation
Domain ownership
Concurrency
Security controls
```

Admin does not mean direct database ownership.

---

# 94. User Governance Matrix

The following matrix defines the intended perspective of the major user types.

| Capability              |                  User |                 Buyer |                Vendor |                     Moderator |            Admin |
| ----------------------- | --------------------: | --------------------: | --------------------: | ----------------------------: | ---------------: |
| Authenticate            |                   Yes |                   Yes |                   Yes |                           Yes |              Yes |
| Manage own profile      |                   Yes |                   Yes |                   Yes |                           Yes |              Yes |
| Manage own credentials  |                   Yes |                   Yes |                   Yes |                           Yes |              Yes |
| Configure own 2FA       |                   Yes |                   Yes |                   Yes |                           Yes |              Yes |
| Social participation    |                   Yes |                   Yes |                   Yes |              Policy-dependent | Policy-dependent |
| Create posts            |                   Yes |                   Yes |                   Yes |              Policy-dependent | Policy-dependent |
| Comment/react           |                   Yes |                   Yes |                   Yes |              Policy-dependent | Policy-dependent |
| Buy products            |              Optional |                   Yes |              Possible |              Policy-dependent | Policy-dependent |
| Operate shop            |                    No |                    No |                   Yes |                            No |  Governance only |
| Manage own listings     |                    No |                    No |                   Yes |                            No |  Governance only |
| Manage own inventory    |                    No |                    No |                   Yes |                            No |  Governance only |
| Report content          |                   Yes |                   Yes |                   Yes |                           Yes |              Yes |
| Review reports          |                    No |                    No |                    No |                    Authorized |       Authorized |
| Perform moderation      |                    No |                    No |                    No |                    Authorized |       Authorized |
| Manage user roles       |                    No |                    No |                    No | Only if explicitly authorized |       Authorized |
| Platform configuration  |                    No |                    No |                    No |                  No / limited |       Authorized |
| Security administration |             Self only |             Self only |             Self only |            Limited/authorized |       Authorized |
| Audit access            | Own permitted history | Own permitted history | Own permitted history |                    Authorized |       Authorized |

This table describes capability boundaries, not separate user classes.

---

# 95. Account Governance Matrix

| Operation                         | Self-Service User |   Vendor |                Moderator |                      Admin |
| --------------------------------- | ----------------: | -------: | -----------------------: | -------------------------: |
| Update own profile                |               Yes |      Yes |                      Yes |                        Yes |
| Change own password               |               Yes |      Yes |                      Yes |                        Yes |
| Manage own 2FA                    |               Yes |      Yes |                      Yes |                        Yes |
| Deactivate own account            |            Policy |   Policy |               Restricted |                 Restricted |
| Delete own account                |            Policy |   Policy |               Restricted |                 Restricted |
| Suspend another user              |                No |       No |               Authorized |                 Authorized |
| Reactivate another user           |                No |       No |               Authorized |                 Authorized |
| Change another user's role        |                No |       No | Explicit permission only |                 Authorized |
| Modify another user's credentials |                No |       No |                       No | Restricted/security policy |
| View sensitive user data          |          Own only | Own only |             Need-to-know |               Need-to-know |
| Direct database modification      |                No |       No |                       No |                         No |

---

# 96. Account Lifecycle

The canonical lifecycle is:

```text
Registration / External Authentication
            ↓
ApplicationUser Established
            ↓
UserProfile Ensured
            ↓
Verification
            ↓
Active
     ┌──────┴────────┐
     ↓               ↓
Deactivated       Suspended
     ↓               ↓
Reactivated       Reinstated
     └──────┬────────┘
            ↓
     Active / Final State
            ↓
      Deletion Workflow
```

Actual state transitions must be governed by explicit policies.

---

# 97. Lifecycle Invariants

The following invariants must hold:

### Identity invariant

Every authenticated SocialConnect account corresponds to a valid Identity account.

### Profile invariant

A successfully established SocialConnect user has an associated `UserProfile` according to the registration contract.

### Authorization invariant

Authentication alone does not grant business authorization.

### Ownership invariant

Users may modify only resources they own unless an authorized administrative/domain policy permits otherwise.

### Governance invariant

Administrative actions require explicit authorization.

### Domain ownership invariant

Administrative code cannot bypass the owner of a business domain.

### Audit invariant

Governed privileged mutations are auditable where required.

### Security invariant

Security-sensitive operations cannot rely solely on client-side controls.

---

# 98. User Governance Does Not Own Every User-Related Domain

This document governs the Application User.

It does not absorb every domain that references the user.

The boundaries remain:

```text
User Governance
    → account identity, security, lifecycle, authorization context

Social Domain
    → posts, comments, reactions, friendships, blocks

Media Domain
    → media lifecycle

Marketplace
    → shops, products, listings, inventory

Order Domain
    → orders

Payment Domain
    → payments/settlement

Notification Domain
    → notifications

Event Domain
    → events/outbox

Moderation
    → reports/cases/decisions/actions

Administration
    → privileged governance
```

This prevents the Application User subsystem from becoming a God module.

---

# 99. Repository and Service Responsibility

The final implementation must preserve:

```text
Controller
    → HTTP boundary

Application Service
    → use-case/business orchestration

Domain Service
    → domain rules where required

Loader
    → controlled entity retrieval

Lookup Service
    → reusable read gateway

Repository
    → persistence abstraction

UnitOfWork
    → persistence coordination/transaction

Identity Manager
    → Identity operations
```

No layer may casually absorb another layer's responsibilities.

---

# 100. Unit of Work

User-related operations that modify multiple persistent structures may require a Unit of Work transaction.

For example:

```text
User Governance
    ↓
Account state change
    +
Audit
    +
Domain state change
    +
Event/outbox
```

where those operations must commit atomically.

The Unit of Work coordinates persistence.

It does not decide business policy.

---

# 101. Error and Validation Requirements

User governance operations must validate:

```text
Required fields
Account state
Target existence
Target state
Authorization
Verification
Role
Capability
Business policy
Concurrency
```

Validation must be server-side.

Client validation is supplementary only.

---

# 102. Security Logging vs Audit

The implementation must distinguish operational/security logging from business audit.

```text
Logging
    → technical diagnostics

Audit
    → authoritative record of governed business/administrative action
```

A log entry must not be treated as a substitute for an audit record where an audit requirement exists.

---

# 103. User Data Access

User data access must follow least privilege.

A user should receive only the data necessary for the operation.

Administrative interfaces should expose sensitive information only where the administrator/moderator has an explicit business need and authorization.

---

# 104. No Role-Based UI as Security

The UI may hide controls according to role/capability.

However:

```text
Hidden button
    ≠
Authorization
```

Every operation must be protected on the server.

---

# 105. No Client-Controlled Ownership

The client must not be trusted to determine:

```text
Actor UserId
Account Owner
Role
Administrative Authority
Vendor Authority
Moderation Authority
```

These must be derived from the authenticated/security context and validated server-side.

---

# 106. Performance Requirements

User operations must avoid unnecessary entity graphs.

Examples:

```text
User search
    → projection / DTO where appropriate

Profile display
    → only required profile data

Administrative search
    → deterministic pagination

User discovery
    → no unnecessary tracking

Relationship queries
    → targeted queries
```

The repository/loader design must support these requirements.

---

# 107. Pagination

User discovery and administrative user search must use deterministic pagination.

Ordering must be stable.

The exact pagination strategy must follow the established SocialConnect pagination contract.

User search must not load all users into memory.

---

# 108. User Search

User search may support criteria such as:

```text
Search term
Country
City
Verified state
Active state
Other explicitly supported filters
```

The final query contract must define:

* filtering;
* sorting;
* pagination;
* authorization;
* privacy restrictions;
* projection.

---

# 109. User Governance Testing

The complete Application User implementation must be tested at multiple levels.

### Identity tests

```text
Registration
Login
External login
Password
Email verification
2FA
```

### Account lifecycle tests

```text
Activation
Deactivation
Suspension
Reactivation
Deletion
```

### Authorization tests

```text
Self access
Other-user access
Admin access
Moderator access
Unauthorized escalation
```

### Role tests

```text
User
Vendor
Moderator
Admin
```

### Security tests

```text
Privilege escalation
IDOR/ownership bypass
Credential changes
Self-administrative deletion
Unauthorized role assignment
```

### Integration tests

```text
Events
Outbox
Notifications
Moderation
Marketplace
```

---

# 110. Critical Security Test Cases

The final implementation must explicitly test that:

```text
User A cannot modify User B's account.
```

```text
User cannot assign themselves Admin.
```

```text
User cannot assign themselves Moderator.
```

```text
Vendor cannot access another Vendor's shop management.
```

```text
Moderator cannot perform Admin-only operations.
```

```text
Admin/Moderator cannot self-delete through normal deletion.
```

```text
Suspended users cannot perform prohibited operations.
```

```text
Client-supplied UserId cannot override authenticated identity.
```

```text
TargetType + TargetId cannot bypass authorization.
```

---

# 111. Architectural Anti-Patterns

The following are prohibited:

### God User Service

```text
UserService
    → posts
    → comments
    → reactions
    → products
    → orders
    → notifications
    → moderation
    → administration
```

### God User Repository

A repository containing every possible user-related query and business workflow is prohibited.

### Direct Identity Table Access

Controllers/services must not bypass Identity abstractions unnecessarily.

### Direct Cross-Domain Table Manipulation

Administration must not directly update domain tables when a domain/application operation exists.

### Target Switch Explosion

`TargetType + TargetId` must not produce one giant switch handling every feature.

### UI-Owned Governance

JavaScript must not determine whether a user is authorized.

---

# 112. Final Application User Architecture

The complete conceptual architecture is:

```text
                         ┌───────────────────────┐
                         │   ASP.NET Identity    │
                         │                       │
                         │   ApplicationUser     │
                         └───────────┬───────────┘
                                     │
                         identity/account boundary
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │     UserProfile       │
                         └───────────┬───────────┘
                                     │
             ┌───────────────────────┼───────────────────────┐
             │                       │                       │
             ▼                       ▼                       ▼
       Social User                Buyer                 Vendor
             │                       │                       │
             │                       │                       ▼
             │                       │                     Shop
             │                       │                       │
             │                       │                       ▼
             │                       │                 Products /
             │                       │                 Listings /
             │                       │                 Inventory
             │                       │
             └───────────────────────┴───────────────────────┐
                                                             │
                                                             ▼
                                                    Platform Governance
                                                             │
                                           ┌─────────────────┴───────────────┐
                                           ▼                                 ▼
                                       Moderator                           Admin
                                           │                                 │
                                           ▼                                 ▼
                                      Moderation                     Administration
```

Across all of these perspectives:

```text
TargetType + TargetId
```

provides the canonical polymorphic targeting mechanism where required.

---

# 113. Final User Governance Principle

The SocialConnect Application User is **one identity with multiple possible platform perspectives**.

The system must not create separate user entities for:

```text
User
Buyer
Seller
Vendor
Moderator
Admin
```

Instead:

```text
ApplicationUser
      +
UserProfile
      +
Identity Roles
      +
Capabilities
      +
Domain Ownership
      +
Governance State
```

determine the user's effective participation in the platform.

---

# 114. Final Separation of Concepts

The following distinctions are mandatory:

```text
ApplicationUser
    → Identity account

UserProfile
    → SocialConnect profile/business user information

Role
    → authorization/governance position

Capability
    → permitted business operation

Ownership
    → relationship to a business resource

Account State
    → active/deactivated/suspended/etc.

Verification
    → security/platform verification state

TargetType + TargetId
    → polymorphic target identification

Governance Action
    → privileged operation over an account/resource

Audit
    → authoritative record of governed action
```

No single property or class should be used as a substitute for all of these concepts.

---

# 115. Final Implementation Contract

The implementation of the SocialConnect Application User must satisfy the following sequence:

```text
Identity
    ↓
ApplicationUser
    ↓
UserProfile
    ↓
Account Security
    ↓
Verification
    ↓
Roles
    ↓
Capabilities
    ↓
Domain Ownership
    ↓
Social Participation
    ↓
Buyer Participation
    ↓
Vendor Participation
    ↓
Administration / Moderation where authorized
    ↓
Governance
    ↓
Audit / Events / Notifications where required
    ↓
Account Lifecycle
```

Every stage must have a defined application/service ownership boundary.

---

# 116. Final Requirements Checklist

Before the Application User subsystem is considered complete, the implementation must provide:

* [ ] `ApplicationUser` remains outside the SocialConnect `BaseEntity` hierarchy.
* [ ] `UserProfile` remains separate from `ApplicationUser`.
* [ ] First successful authentication ensures a UserProfile exists.
* [ ] Local Identity authentication works.
* [ ] Google external authentication works.
* [ ] Facebook external authentication works.
* [ ] Email verification is implemented according to policy.
* [ ] Credential management is implemented.
* [ ] 2FA is implemented.
* [ ] Login tracking is implemented.
* [ ] Account activation/deactivation is implemented.
* [ ] Account deletion workflow is implemented.
* [ ] Admin self-deletion is prevented.
* [ ] Moderator self-deletion is prevented.
* [ ] Role membership is governed.
* [ ] Admin authorization is explicit.
* [ ] Moderator authorization is limited and explicit.
* [ ] Vendor eligibility is governed.
* [ ] Buyer capability is governed.
* [ ] Ownership authorization is enforced server-side.
* [ ] User blocking follows the SocialConnect relationship contract.
* [ ] User reporting integrates with Reporting/Moderation.
* [ ] User governance actions are auditable where required.
* [ ] TargetType + TargetId is used consistently where polymorphic targeting is required.
* [ ] Target resolution does not bypass domain ownership.
* [ ] User repository remains persistence-focused.
* [ ] Identity operations use the appropriate Identity abstractions.
* [ ] Loaders/lookup services remain separate from repositories.
* [ ] Controllers remain thin.
* [ ] ViewModels remain presentation-only.
* [ ] Builders remain responsible for ViewModel construction.
* [ ] JavaScript remains client interaction only.
* [ ] User-specific business logic does not move into `site.js`.
* [ ] Cross-domain events use the canonical Event/Outbox platform.
* [ ] Notifications use the canonical Notification platform.
* [ ] Alerting uses the canonical Alert source/policy architecture.
* [ ] Administration invokes domain/application operations rather than directly manipulating domain tables.
* [ ] Security-sensitive operations have appropriate authorization/re-authentication.
* [ ] Concurrency is handled for privileged mutations.
* [ ] Critical authorization/security tests exist.
* [ ] End-to-end user lifecycle testing is completed.

---

# 117. Final Status

This document is the **canonical Application User Governance & Final Requirements Contract** for SocialConnect.

It establishes the user/account foundation upon which the following areas will be implemented and verified:

```text
Identity
User Profile
Authentication
Security
Account Lifecycle
Roles
Capabilities
Social Participation
Buyer
Vendor
Shop Ownership
Reporting
Moderation
Administration
Marketplace Governance
Events
Notifications
Alerting
Audit
```

The implementation phase must now follow this contract rather than independently redefining user behavior inside individual modules.

The next architectural step is therefore **not to immediately modify `ApplicationUser`**.

The correct sequence is:

```text
1. Lock this User Governance Contract
        ↓
2. Inspect current ApplicationUser
        ↓
3. Inspect ApplicationUserConfiguration
        ↓
4. Inspect UserProfile
        ↓
5. Inspect Identity configuration
        ↓
6. Inspect IUserRepository/UserRepository
        ↓
7. Inspect UnitOfWork interaction
        ↓
8. Inspect account/authentication services
        ↓
9. Inspect role/authorization implementation
        ↓
10. Inspect every user-owned relationship
        ↓
11. Compare implementation against this contract
        ↓
12. Classify gaps
        ↓
13. Approve required migrations
        ↓
14. Implement
        ↓
15. Compile
        ↓
16. Smoke test
        ↓
17. End-to-end user governance verification
```

**Architectural principle:** one `ApplicationUser`, multiple platform perspectives, explicit capabilities, explicit ownership, explicit governance, and strict separation between Identity, SocialConnect domain entities, and administrative authority.
