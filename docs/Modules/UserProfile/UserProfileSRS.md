# SocialConnect — User Profile Functional & Domain Requirements

**Document Status:** Canonical / Final Requirements Contract
**Document Type:** User Profile Requirements
**System:** SocialConnect
**Profile Benchmark:** Facebook-class personal social profile
**Identity Owner:** `ApplicationUser`
**Profile Owner:** `UserProfile`
**Architecture:** Layered Modular Monolith
**Persistence:** EF Core / SQL Server

---

# 1. Purpose

This document defines the final functional, structural, privacy, presentation, and interaction requirements for the **SocialConnect User Profile**.

The objective is to establish a personal profile experience comparable in breadth and usability to a modern Facebook personal profile while preserving SocialConnect's own architecture, domain boundaries, geographic context, marketplace integration, and extensibility.

The profile is not merely an account information screen.

It is the user's:

* personal identity;
* social presentation;
* public-facing introduction;
* profile media surface;
* location context;
* social discovery entry point;
* activity entry point;
* relationship entry point;
* marketplace identity bridge;
* personal information presentation layer.

The profile must therefore be designed as a **first-class SocialConnect domain concept**.

---

# 2. Fundamental Architectural Distinction

The following distinction is mandatory.

```text
ApplicationUser
        │
        │ Identity / Account
        ▼
   UserProfile
        │
        ├── Personal Information
        ├── Profile Presentation
        ├── Profile Media References
        ├── Location
        ├── About / Introduction
        ├── Privacy Preferences
        └── Social Profile Configuration
```

The profile does **not** replace `ApplicationUser`.

---

# 3. ApplicationUser vs UserProfile

## ApplicationUser

`ApplicationUser` owns Identity/account concerns.

Examples:

```text
Authentication
Email
Password
Security
2FA
Account state
Identity roles
Login information
Verification/account state
```

## UserProfile

`UserProfile` owns SocialConnect personal-profile concerns.

Examples:

```text
About me
Profile presentation
Profile picture reference
Cover photo reference
Location presentation
Personal details
Profile privacy
Profile sections
Social presentation
```

Therefore:

```text
ApplicationUser
    ≠
UserProfile
```

but:

```text
ApplicationUser
    1 : 1
UserProfile
```

---

# 4. Profile Objective

The SocialConnect profile must answer the following questions when another authorized user visits it:

```text
Who is this person?

What do they want other people to know about them?

Where are they generally located?

What does their profile look like?

What social activity do they share?

What relationships do we have?

What content have they chosen to expose?

What shops or marketplace presence do they have?

How can I interact with them?
```

The profile must provide a coherent answer without requiring the visitor to navigate through unrelated account-management screens.

---

# 5. Facebook-Class Profile Benchmark

The minimum functional benchmark is a **full personal social profile**, conceptually comparable to the profile experience offered by Facebook.

That means the SocialConnect profile should support the same broad categories of functionality:

```text
Profile identity
Profile picture
Cover image
Intro / bio
About information
Location
Personal details
Profile activity
Posts
Media
Friends / relationships
Followers / following where applicable
Profile privacy
Profile editing
Profile presentation
Featured content
Social interactions
```

SocialConnect may implement these differently, but the resulting user experience must not be reduced to a basic CRUD profile.

---

# 6. Profile Identity

The profile must prominently identify the person.

The primary identity presentation is:

```text
FirstName + LastName
```

The profile should provide a consistent display name derived from the user's account/profile data.

The current `ApplicationUser.FullName` convenience property may be used for presentation, but profile rendering must not create duplicate identity logic.

---

# 7. Profile Picture

Every user profile must support a profile picture.

The profile picture is the primary visual representation of the user.

The profile system must support:

```text
Upload
Replace
Remove
Display
Privacy-aware viewing
```

Profile media must use the canonical SocialConnect Media architecture.

It must not create a second upload pipeline.

The architecture remains:

```text
Profile UI
    ↓
MediaUploader
    ↓
Media Upload API
    ↓
MediaUploadService
    ↓
Media Domain
    ↓
Profile assignment
```

---

# 8. Profile Picture Ownership

The profile picture is associated with the user's profile.

The profile should reference the appropriate finalized Media entity rather than storing binary image data inside `UserProfile`.

Conceptually:

```text
UserProfile
    │
    └── ProfileMediaId
            │
            ▼
          Media
```

The exact relationship must follow the finalized Media domain contract.

---

# 9. Cover Photo

The profile must support a large cover/banner image.

The cover image provides the primary visual background/header of the profile.

The profile must support:

```text
Upload cover image
Replace cover image
Remove cover image
Display cover image
Privacy-aware access
```

Cover media must use the canonical Media module.

---

# 10. Profile Header

The profile header should combine:

```text
Cover Photo
    +
Profile Picture
    +
Display Name
    +
Optional verification/presentation indicators
    +
Introductory information
    +
Profile actions
```

The header is the primary identity surface of the profile.

---

# 11. Profile Introduction / Bio

The profile must support a short personal introduction.

The user should be able to answer:

> "Who am I?"

through a concise biography/about text.

The bio should support reasonable length limits and validation.

It must not be used as a replacement for structured profile data.

---

# 12. Intro Section

The profile should have a dedicated introduction section.

The intro may present selected high-value information such as:

```text
Bio
Location
Occupation
Education
Relationship information
Website
Other selected personal details
```

The exact fields are governed by the profile privacy contract.

---

# 13. About Section

The profile must provide an expanded **About** experience.

The About section should allow users to manage supported structured information rather than forcing all information into a single biography field.

Conceptually:

```text
About
├── Basic Information
├── Contact Information
├── Location
├── Work
├── Education
├── Relationship / Family
├── Personal Information
└── Other supported profile information
```

Only fields actually approved for SocialConnect V1 should be implemented.

---

# 14. Structured Personal Information

The profile architecture should support structured information where it provides meaningful social value.

Examples include:

```text
Current city
Hometown
Country
Occupation
Employer
Education
School
Website
Languages
Interests
```

The system must not blindly copy every historical Facebook profile field.

The requirement is **Facebook-class profile richness**, not exact field duplication.

---

# 15. Location

Location is a first-class SocialConnect profile concept.

At minimum, the profile should support:

```text
Country
City
```

using the existing geographic model where applicable.

This directly supports SocialConnect's broader product philosophy:

```text
Social Graph
        +
Geographic Context
        +
Product Graph
        +
Shop Graph
```

---

# 16. Location Privacy

Location must be privacy-aware.

The profile must distinguish between:

```text
Stored location
Displayed location
Discoverable location
```

A user must not be forced to expose precise location merely because location is stored.

The profile should prefer appropriate geographic granularity such as:

```text
Country
City
Neighborhood
```

where supported by the final privacy/location contract.

---

# 17. Profile Privacy

Privacy is a fundamental profile requirement.

The user must be able to control the visibility of supported profile information.

Conceptually, visibility may include:

```text
Public
Authenticated Users
Friends / Connections
Only Me
```

The exact visibility model must be finalized before implementation of advanced profile privacy.

The system must not hard-code visibility assumptions inside Razor views.

---

# 18. Privacy Is Server-Side

Profile privacy is an authorization/business rule.

The client must never determine whether profile information is visible.

The correct flow is:

```text
Viewer
    ↓
Profile Query
    ↓
Privacy Policy
    ↓
Authorized Profile Projection
    ↓
ViewModel
    ↓
Razor
```

---

# 19. Public vs Private Profile Projection

The profile query layer should support different projections according to viewer context.

Conceptually:

```text
Owner View
    ↓
Full permitted profile

Friend/Connection View
    ↓
Connection-visible profile

Authenticated Visitor
    ↓
Visitor-visible profile

Public Visitor
    ↓
Public profile
```

This avoids loading sensitive profile information and hiding it only in the UI.

---

# 20. Profile Editing

The profile owner must be able to edit their profile.

Profile editing must support:

```text
Basic information
Bio
About information
Location
Profile media
Cover media
Supported personal details
Privacy
```

The editing experience should be organized into coherent sections rather than one enormous form.

---

# 21. Profile Completion

SocialConnect should support profile completion without forcing every field to be mandatory.

A newly created profile may be:

```text
Empty
Partially completed
Complete
```

The profile system may calculate a completion state for UX purposes.

Profile completion must not become an authorization substitute unless a specific business policy requires completion.

---

# 22. Profile Activity

A user's profile must provide access to their social activity.

This should include appropriate content such as:

```text
Posts
Shared posts
Media activity
Comments where appropriate
Reactions where appropriate
```

The profile should not duplicate the underlying content.

It should query and present content owned by the respective domains.

---

# 23. Posts on Profile

A user's posts should appear in their profile activity/timeline where visibility permits.

The profile does not own Posts.

The relationship is:

```text
UserProfile
    │
    ▼
ApplicationUser
    │
    ▼
Post.UserId
```

Post creation and post business rules remain owned by the Post module.

---

# 24. Profile Timeline

A Facebook-class profile should provide a chronological/activity-oriented view of the user's content.

The profile timeline may contain:

```text
User-created posts
Shared posts
Media posts
Relevant activity
```

The final Feed/Timeline architecture determines exactly which activities are included.

The profile module should orchestrate presentation, not recreate feed business logic.

---

# 25. Featured Content

The profile should support a **Featured** presentation area.

Featured content may allow the user to highlight selected profile-relevant content.

Examples:

```text
Selected photos
Selected posts
Selected media
Selected profile information
```

The underlying content remains owned by its source module.

The profile stores only the selection/configuration required to present it.

---

# 26. Profile Photos

The profile should provide a dedicated photo/media experience.

Users should be able to access appropriate profile-related media.

The Media domain remains the owner of:

```text
Media
Storage
Upload
Finalization
Assignment
Deletion
Media metadata
```

The Profile domain determines how profile media is presented.

---

# 27. Photo Albums / Collections

A future-capable profile architecture should permit organized media collections/albums.

Where implemented:

```text
Profile
    ↓
Album / Collection
    ↓
Media
```

The exact Album domain should be defined separately if it becomes a substantial feature.

The initial UserProfile model must not become responsible for media collection business logic.

---

# 28. Friends / Social Connections

The profile should expose the user's social relationships where policy permits.

The existing SocialConnect architecture already contains:

```text
Friendship
```

Therefore the profile should query the Friendship domain rather than maintaining a duplicate friend list.

The profile may display:

```text
Friend count
Friend previews
Friends section
Mutual friends
```

where supported.

---

# 29. Friend Relationship State

When viewing another profile, the profile UI should be capable of representing the relationship between viewer and profile owner.

Examples:

```text
No relationship
Friend
Friend request sent
Friend request received
Blocked
Mutually connected
```

The relationship state belongs to the social relationship domain.

---

# 30. Mutual Connections

Where supported, a profile may display mutual connections.

The system must calculate this through the Friendship domain.

The profile must not maintain a duplicate graph.

---

# 31. Followers / Following

Where SocialConnect supports follower relationships, the profile may expose:

```text
Follower count
Following count
```

and appropriate lists according to privacy.

This must be backed by the relevant domain relationship rather than profile-owned collections.

---

# 32. Shop Presence on Profile

Because SocialConnect combines social networking with commerce, the profile must be capable of showing the user's marketplace presence.

For a user who owns a shop, the profile may present:

```text
Shop
Shop name
Shop description
Selected products
Shop status
```

subject to marketplace/privacy rules.

The profile does not become the Shop domain.

---

# 33. Vendor Identity

A Vendor's personal profile and commercial identity must remain distinguishable.

```text
Personal Profile
    ↓
Person

Shop
    ↓
Commercial identity
```

This allows one person to participate socially while also operating one or more shops.

---

# 34. Buyer Identity

A buyer does not require a separate profile entity.

The same `ApplicationUser + UserProfile` represents the person.

Buyer behavior belongs to the marketplace/order domain.

---

# 35. Profile and Saved Products

The profile may expose appropriate marketplace personalization such as saved products where the user chooses to expose it.

`SavedProduct` remains the owner of the saved-product relationship.

The profile only presents the information.

---

# 36. Profile and Shop Following

The profile may show followed shops where policy permits.

The underlying relationship remains:

```text
ShopFollow
```

The profile is only a presentation surface.

---

# 37. Profile Actions

When viewing another user's profile, the system should provide context-sensitive actions.

Examples:

```text
Add Friend
Friends
Follow
Following
Message
Block
Report
```

Only actions supported by the relevant SocialConnect modules should appear.

The profile must not implement these business operations itself.

---

# 38. Profile Action Ownership

For example:

```text
Add Friend
    → Friendship service

Block
    → User relationship service

Message
    → Messaging service

Report
    → Reporting service

Follow Shop
    → ShopFollow service
```

The profile is the composition/presentation surface.

---

# 39. Profile Verification Indicator

Where the account/profile is verified, the profile may display a verification indicator.

The indicator must derive from the authoritative verification state.

The profile must not invent its own verification flag.

---

# 40. Profile Name and Identity Integrity

The profile display name must use authoritative account/profile information.

The system must define rules for:

```text
Name changes
Validation
History where required
Administrative restrictions
Identity/security implications
```

Profile presentation must not allow arbitrary display names to bypass account identity policy.

---

# 41. Profile Username / Handle

If SocialConnect introduces usernames/handles, they must be treated as a separate identity/profile concept.

A username should be:

```text
Unique
Validated
Reserved appropriately
Case-normalized according to policy
Change-controlled
URL-safe
```

A username must not silently replace the canonical Identity user ID.

---

# 42. Profile URL

A user profile should have a stable navigable identity.

Conceptually:

```text
/social/users/{identifier}
```

or the final project route defined by the routing contract.

The route identifier should remain stable even if display information changes.

---

# 43. Profile Searchability

Users may be discoverable through profile search according to privacy and account state.

Search results should be able to expose appropriate information such as:

```text
Profile picture
Name
Location
Verification indicator
Mutual connections
Relevant public information
```

Sensitive profile information must not be included in search results unless authorized.

---

# 44. Profile Discoverability

The user should have appropriate control over discoverability where supported.

Possible dimensions include:

```text
Profile discoverability
Search discoverability
Location discoverability
Friend list visibility
Activity visibility
```

The final privacy model must define these explicitly.

---

# 45. Profile Blocking

A blocked user must not receive profile access beyond the behavior defined by the User Block policy.

The profile query must respect block relationships.

Example:

```text
Viewer
    ↓
Block relationship
    ↓
Profile visibility policy
    ↓
Restricted profile result
```

---

# 46. Profile Reporting

A profile should be reportable.

The profile report must use the canonical Reporting architecture.

Conceptually:

```text
Profile
    ↓
Report
    ↓
TargetType + TargetId
```

where the target identifies the user/profile according to the finalized target-type catalog.

---

# 47. TargetType and UserProfile

The polymorphic targeting mechanism must remain consistent.

Where a user/profile itself is a reportable or governable target, the final TargetType contract must define the appropriate target identity.

The profile must not invent a second polymorphic target mechanism.

---

# 48. Profile Media and TargetType

Profile media assignment must respect the canonical Media ownership/role model.

The Profile domain should not create custom media storage logic.

The final Media contract determines the appropriate:

```text
OwnerType
OwnerId
MediaRole
```

for profile media.

---

# 49. Profile Audit

Changes to sensitive profile information may require audit depending on policy.

Potentially auditable changes include:

```text
Identity-sensitive information
Security-sensitive information
Verification-related information
Administrative profile changes
Privacy changes
```

Ordinary profile editing does not automatically mean every field requires an audit record.

The final audit policy determines the exact scope.

---

# 50. Profile Change Events

Profile changes that have cross-module consequences may produce domain/integration events.

Examples could include:

```text
ProfileCreated
ProfileUpdated
ProfileLocationChanged
ProfilePictureChanged
ProfilePrivacyChanged
```

These are requirements-level concepts only.

Exact event contracts belong to the Event module.

---

# 51. Profile and Notification

Where a profile change requires notification, the canonical Notification system must be used.

The Profile module must not directly implement another notification system.

---

# 52. Profile and Feed

The profile must integrate with the canonical Feed/Timeline architecture.

It must not duplicate feed-generation rules.

Conceptually:

```text
Profile
    ↓
Profile Timeline Context
    ↓
Feed/Post query mechanisms
    ↓
Post/ViewModel presentation
```

---

# 53. Profile and Comment Activity

Comments remain owned by the Comment module.

The profile may present relevant user activity but should not load or manage comments as profile-owned entities.

---

# 54. Profile and Reaction Activity

Reactions remain owned by the Reaction module.

The profile may present reaction-related activity where appropriate.

The profile must not calculate or mutate reaction aggregates itself.

---

# 55. Profile and Reactionable

The `UserProfile` itself should not automatically inherit from:

```text
Reactable
```

unless a deliberate business requirement establishes that profiles themselves are reactable.

This must be a conscious domain decision.

The existence of reactions elsewhere does not justify making every user profile reactable.

---

# 56. Domain Hierarchy Position

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

`UserProfile` should inherit from the appropriate level based on its actual requirements.

It should not inherit from `Reactable` merely because other social entities do.

The final model decision must be based on whether a profile itself legitimately supports reactions.

---

# 57. Soft Deletion

If `UserProfile` is a SocialConnect domain entity requiring soft deletion, it must follow the established:

```text
SoftDeleteEntity
```

contract.

Profile deletion must be distinguished from:

```text
ApplicationUser deletion
```

and:

```text
Profile visibility
```

These are separate concepts.

---

# 58. Profile Ownership

The profile belongs to exactly one Application User.

Conceptually:

```text
ApplicationUser.Id
        │
        ▼
UserProfile.UserId
```

The profile must not exist as an independent person's identity without its owning account.

---

# 59. One-to-One Invariant

The system should maintain:

```text
One ApplicationUser
        ↕
One UserProfile
```

for every SocialConnect account requiring a profile.

The registration/first-authentication workflow is responsible for ensuring this invariant.

---

# 60. Profile Data Model Philosophy

The profile model should contain **profile-owned information**, not every piece of information that happens to be associated with a user.

Good profile fields:

```text
Bio
Location reference
Profile media references
Cover media reference
Personal profile information
Profile presentation configuration
Privacy-related profile configuration
```

Poor profile fields:

```text
Posts
Comments
Reactions
Orders
Payments
Notifications
Inventory
Moderation cases
Friendship rows
Media binaries
```

Those remain separate domain entities.

---

# 61. Navigation Properties

Navigation properties may exist where they provide legitimate ORM relationships.

However, excessive navigation collections must not turn `UserProfile` into a giant aggregate graph.

For example, this should not become:

```text
UserProfile
    ├── AllPosts
    ├── AllComments
    ├── AllReactions
    ├── AllOrders
    ├── AllNotifications
    ├── AllProducts
    ├── AllMedia
    └── ...
```

Queries should retrieve the specific information required by each use case.

---

# 62. Profile Aggregate Boundary

The profile should be treated as a focused domain boundary.

Its responsibilities are:

```text
Profile identity/presentation
Profile-owned information
Profile media references
Profile preferences
Profile visibility configuration
```

Other domains remain external owners.

---

# 63. Profile Builder

The UI architecture must use a dedicated Profile Builder where the profile page requires composed data.

Conceptually:

```text
Profile Request
    ↓
Profile Context
    ↓
Loaders / Lookup Services
    ↓
Resolvers / Policies
    ↓
Profile Builder
    ↓
Profile ViewModel
    ↓
Profile View
```

The Builder configures the ViewModel.

It must not become a God service.

---

# 64. Profile ViewModel

The Profile ViewModel represents the page/screen presentation.

It may contain:

```text
Profile identity
Profile header
Profile picture
Cover photo
Intro
About summary
Location summary
Relationship state
Friend summary
Follower summary
Activity summary
Featured content
Shop summary
Available actions
Privacy-aware data
```

It must not be persisted.

---

# 65. Profile View

The main MVC View acts as the page shell.

It should compose reusable components such as:

```text
ProfileHeader
ProfileIntro
ProfileAbout
ProfileFriends
ProfilePhotos
ProfileFeatured
ProfileTimeline
ProfileShops
```

where these components are justified and finalized.

The page shell must not contain large amounts of business logic.

---

# 66. ViewComponent Architecture

Profile-specific reusable UI should follow the established SocialConnect pattern:

```text
ViewComponent
    ↓
Defaults
    ↓
ViewModel
    ↓
Builder
    ↓
Default.cshtml
    ↓
CSS / JS
```

The exact component decomposition must be finalized during product/UI architecture investigation.

---

# 67. Profile JavaScript

Profile JavaScript must remain minimal.

It may manage:

```text
Edit interactions
Media selection
Tabs
Expand/collapse
Profile actions
API calls
Client-side UI state
Events
```

It must not own:

```text
Authorization
Privacy rules
Profile ownership
Business validation
Role decisions
```

---

# 68. Profile CSS

Profile styling must be feature-scoped.

Global styles belong in the existing site foundation only when genuinely global.

Profile-specific layout/styling must not pollute `site.css`.

---

# 69. Responsive Profile

The profile must be usable across:

```text
Desktop
Tablet
Mobile
```

The profile header, cover image, profile picture, action controls, About sections, media, and timeline must adapt appropriately.

---

# 70. Profile Navigation

The profile should provide clear navigation between major profile sections.

Potential sections include:

```text
Posts
About
Friends
Photos
Videos
Featured
Shops
```

Only implemented/available sections should appear.

Navigation must be driven by actual capabilities/data.

---

# 71. Empty States

The profile must handle incomplete profiles elegantly.

Examples:

```text
No bio
No cover photo
No profile photo
No posts
No photos
No friends
No shop
No featured content
```

The owner should receive useful calls to action.

Visitors should see appropriate empty-state presentation without exposing private information.

---

# 72. Profile Editing UX

The owner experience should make it obvious:

```text
What information is currently public
What information is editable
What information is private
What information is incomplete
```

Editing should be section-based rather than one massive form.

---

# 73. Profile Media Workflow

The profile media workflow must follow the existing MediaUploader contract.

It must not know or duplicate:

```text
upload internals
temporary storage mechanics
finalization mechanics
storage provider details
```

The profile supplies the appropriate configuration/role.

Media infrastructure remains reusable.

---

# 74. Profile Performance

Profile pages must not load the complete user's social graph by default.

The implementation should use targeted queries for:

```text
Profile
Friends summary
Photos summary
Timeline
Shop summary
Relationship state
```

Large collections must be paginated/lazy-loaded where appropriate.

---

# 75. Profile Caching

Profile read models may eventually be cacheable where useful.

However, caching must respect:

```text
Privacy
Authorization
Profile updates
Account suspension
Blocking
Visibility changes
```

Caching must never cause private profile information to leak to unauthorized viewers.

---

# 76. Profile Search Performance

Profile search should use:

```text
Database filtering
Projection
Deterministic ordering
Pagination
```

rather than loading complete profile graphs.

---

# 77. Profile Security

Profile operations must enforce:

```text
Authentication
Ownership
Authorization
Privacy
Account state
Block state
Verification requirements where applicable
```

The client cannot be trusted to enforce these rules.

---

# 78. Profile Administration

Administrators may inspect or govern profiles according to the Administration Authorization Contract.

They must not directly modify the profile database record when a governed application operation exists.

The correct flow is:

```text
Admin
    ↓
Admin Application Service
    ↓
Profile/User Application Service
    ↓
Profile state change
    ↓
Audit/Event where required
```

---

# 79. Moderator Access

Moderators may access profile information only to the extent required by their authorized moderation responsibilities.

Moderator access must be least-privilege.

A Moderator does not automatically gain unrestricted access to private profile information.

---

# 80. Account State and Profile Visibility

If the ApplicationUser is:

```text
Inactive
Suspended
Deleted
```

the profile presentation must follow the account governance policy.

The Profile module must not invent independent account-state semantics.

---

# 81. Deleted Accounts

A deleted account may require one of several presentation outcomes depending on the finalized retention policy:

```text
Profile unavailable
Anonymized profile
Historical attribution
Removed profile
```

The exact behavior must be determined by account deletion/data-retention requirements.

The profile module must respect that policy.

---

# 82. Suspended Accounts

A suspended account may have restricted profile visibility depending on the suspension policy.

The profile must query authoritative account state.

It must not assume:

```text IsActive == false
```

is the only possible restriction.

---

# 83. Profile Privacy and Social Graph

Profile visibility may interact with:

```text Friendship
Blocking
Following
Account state
Content visibility
```

The final profile query must evaluate these policies before constructing the ViewModel.

---

# 84. Profile Information Categories

The final profile information should be categorized into:

### Identity

```text
Name
Profile picture
Cover photo
Username/handle if implemented
```

### Introduction

```text
Bio
Intro
```

### About

```text
Location
Work
Education
Personal details
Website
Other supported fields
```

### Social

```text
Friends
Followers
Following
Mutual connections
Activity
```

### Media

```text
Photos
Videos
Featured media
```

### Commerce

```text
Shop presence
Public seller information
Selected products
```

### Privacy

```text
Visibility settings
Discoverability
Activity visibility
```

---

# 85. Profile Requirements Matrix

| Capability      | Owner                          | Profile Presents | Profile Owns          |
| --------------- | ------------------------------ | ---------------- | --------------------- |
| Name            | ApplicationUser/Profile policy | Yes              | Identity presentation |
| Bio             | UserProfile                    | Yes              | Yes                   |
| Profile picture | Media + Profile                | Yes              | Reference             |
| Cover photo     | Media + Profile                | Yes              | Reference             |
| Country         | Profile/Location               | Yes              | Reference/value       |
| City            | Profile/Location               | Yes              | Reference/value       |
| Posts           | Post                           | Yes              | No                    |
| Comments        | Comment                        | Optional         | No                    |
| Reactions       | Reaction                       | Optional         | No                    |
| Friends         | Friendship                     | Yes              | No                    |
| Blocks          | User relationship              | Policy only      | No                    |
| Photos          | Media                          | Yes              | No                    |
| Shop            | Shop                           | Yes              | No                    |
| Products        | Product                        | Yes              | No                    |
| Saved products  | SavedProduct                   | Optional         | No                    |
| Notifications   | Notification                   | No/directly      | No                    |
| Orders          | Order                          | Optional/limited | No                    |
| Payments        | Payment                        | No/directly      | No                    |
| Reports         | Reporting                      | Administrative   | No                    |
| Moderation      | Moderation                     | Administrative   | No                    |

---

# 86. Minimum V1 Profile

A SocialConnect V1 profile must not be less than:

```text
Profile Header
├── Profile Picture
├── Cover Photo
├── Full Name
├── Intro/Bio
├── Location
└── Profile Actions

About
├── Basic information
├── Location
└── Supported personal details

Social
├── Friends / relationship summary
└── Profile activity

Media
└── Profile/media presentation

Timeline
└── User posts/activity

Privacy
└── Profile visibility controls
```

This represents the minimum **Facebook-class personal profile baseline**.

---

# 87. Extended Profile

The architecture must remain capable of supporting:

```text
Featured
Albums
Followers
Following
Mutual connections
Profile video
Additional personal details
Professional information
Shop presentation
Marketplace identity
Location-aware discovery
Profile customization
```

without requiring a redesign of the core profile boundary.

---

# 88. What Must Not Be Added to UserProfile

The following must remain outside the `UserProfile` aggregate merely because they belong to a user:

```text
Posts
Comments
Reactions
Friendships
Blocks
Shop
Products
Orders
Payments
Notifications
Media binary data
Moderation cases
Reports
Events
Outbox records
Alert records
```

The profile references or presents these domains through their appropriate services.

---

# 89. Domain Boundary Summary

The canonical boundary is:

```text
ApplicationUser
    │
    │ Identity
    ▼
UserProfile
    │
    ├── Profile information
    ├── Profile presentation
    ├── Profile preferences
    └── Profile-owned references
            │
            ├──────────────► Media
            ├──────────────► Friendship
            ├──────────────► Post
            ├──────────────► Shop
            ├──────────────► Product
            └──────────────► Other domains
```

The arrows represent domain integration/retrieval relationships, not ownership transfer.

---

# 90. Final Architectural Principle

The SocialConnect User Profile must be:

> **A rich, Facebook-class personal social identity and presentation surface, backed by a focused UserProfile domain model and composed from the independent SocialConnect domain modules.**

It must **not** become:

> a database container holding every entity associated with a person.

---

# 91. Final Implementation Sequence

The User Profile implementation must proceed in this order:

```text
1. Lock User Profile Requirements
        ↓
2. Inspect current UserProfile model
        ↓
3. Inspect UserProfile configuration
        ↓
4. Inspect profile-related migrations/database
        ↓
5. Inspect profile media relationships
        ↓
6. Inspect Country/City relationships
        ↓
7. Inspect Friendship relationships
        ↓
8. Inspect UserBlock relationships
        ↓
9. Inspect Post/Profile integration
        ↓
10. Inspect Shop/Profile integration
        ↓
11. Inspect existing profile UI
        ↓
12. Inspect ViewModels
        ↓
13. Inspect Builders/Resolvers
        ↓
14. Inspect Profile ViewComponents
        ↓
15. Inspect profile JavaScript/CSS
        ↓
16. Compare implementation against this contract
        ↓
17. Classify gaps
        ↓
18. Finalize UserProfile domain model
        ↓
19. Finalize profile application services
        ↓
20. Finalize profile query/read architecture
        ↓
21. Finalize UI architecture
        ↓
22. Implement
        ↓
23. Compile
        ↓
24. Smoke test
        ↓
25. End-to-end profile verification
```

---

# 92. Final Acceptance Criteria

The SocialConnect User Profile is complete only when:

* [ ] Every ApplicationUser can have a UserProfile.
* [ ] First authentication establishes the profile.
* [ ] Profile remains separate from Identity.
* [ ] Profile supports a Facebook-class personal presentation.
* [ ] Profile picture is supported.
* [ ] Cover photo is supported.
* [ ] Bio/intro is supported.
* [ ] Structured About information is supported.
* [ ] Country/city location is supported.
* [ ] Profile privacy is enforced server-side.
* [ ] Profile editing is supported.
* [ ] Profile activity is supported.
* [ ] User posts can appear on the profile.
* [ ] Profile media is supported through the canonical Media module.
* [ ] Social relationship information can be presented.
* [ ] Blocking/privacy rules affect profile visibility.
* [ ] Profile reporting is supported.
* [ ] Shop/vendor presence can be represented.
* [ ] Buyer/vendor perspectives remain the same ApplicationUser.
* [ ] Profile does not own Post/Comment/Reaction/Shop/Product business logic.
* [ ] Profile does not duplicate Media infrastructure.
* [ ] TargetType + TargetId is used consistently where applicable.
* [ ] Profile queries are privacy-aware.
* [ ] Profile queries are performance-conscious.
* [ ] Large collections are paginated/lazy-loaded where appropriate.
* [ ] Profile UI follows View → ViewComponent → Builder → ViewModel architecture.
* [ ] Profile JavaScript follows `App.Modules`.
* [ ] `site.js` remains foundation-only.
* [ ] Controllers remain thin.
* [ ] Admin access follows Administration Authorization.
* [ ] Moderator access follows least privilege.
* [ ] Profile changes requiring events use the canonical Event/Outbox platform.
* [ ] Notifications use the canonical Notification platform.
* [ ] Security and ownership checks are server-side.
* [ ] Profile behavior respects account suspension/deactivation/deletion.
* [ ] Profile implementation does not become a God service or God entity.

---

# 93. Final Contract Statement

The SocialConnect User Profile is a **first-class personal social identity domain**, not a simple CRUD table.

Its purpose is to provide the user-facing personal identity layer that connects:

```text
Person
   ↓
Profile
   ↓
Social Graph
   ↓
Content
   ↓
Media
   ↓
Geographic Context
   ↓
Marketplace Identity
```

while maintaining strict ownership boundaries between the underlying domains.

The target experience is **Facebook-class personal profile functionality**, adapted to SocialConnect's own architecture and product model.

The implementation must therefore provide the richness users expect from a mature social-network profile while preserving:

```text
ApplicationUser
    → Identity

UserProfile
    → Personal profile

Other domain modules
    → Their own business ownership

Builders/ViewModels
    → Presentation composition

Application Services
    → Use-case orchestration

Repositories/Loaders
    → Controlled persistence access

TargetType + TargetId
    → Polymorphic target identification

Event/Outbox
    → Cross-module integration

Notification
    → User notification delivery

Administration
    → Privileged governance
```

**This document is the canonical functional baseline for the SocialConnect User Profile and must be used as the reference when the existing UserProfile implementation is subsequently audited.**
