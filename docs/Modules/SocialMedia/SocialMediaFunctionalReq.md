# SocialConnect — Social Media Domain & Functional Requirements

**Status:** Canonical Requirements Contract
**Scope:** Social Media subsystem
**Version:** 1.0
**Platform:** ASP.NET Core MVC / .NET 8 / EF Core / SQL Server
**Architecture:** Layered Modular Monolith
**Primary Domains:** Posts, Media, Reactions, Comments, Sharing, Feed Generation, Social Graph Integration
**Cross-Cutting Platforms:** Domain Events, Transactional Outbox, Event Dispatcher, Notification, Alerting, Audit, Authorization

---

# 1. Purpose

The Social Media subsystem is the core social-network capability of SocialConnect.

It provides the ability for users to:

* create and publish posts;
* attach media to posts;
* react to posts and other supported targets;
* create comments and replies;
* attach media to comments;
* share posts;
* consume personalized feeds;
* interact with content presented inside feeds;
* participate in social relationships that influence feed eligibility;
* receive downstream notification and alert effects from social actions.

The subsystem must provide a **Facebook-class social experience and behavioral model** while remaining a native SocialConnect architecture.

The objective is **functional parity at the social capability level**, not source-code, implementation, database, or proprietary algorithm replication.

Facebook publicly describes Feed as a personalized stream whose candidate stories are selected and then ranked using signals such as relationships, recency, engagement, and predicted user interest.

SocialConnect therefore adopts the same fundamental architectural principle:

```text
Social Business Operation
        ↓
Domain State Change
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Event Dispatcher
        ↓
Social Event Handler
        ↓
Feed / Notification / Alert / Other Impact
```

The business operation itself remains independent from downstream consumers.

---

# 2. Architectural Position

The Social Media subsystem is composed of the following major capabilities:

```text
Social Media
│
├── Post
│   ├── Creation
│   ├── Publication
│   ├── Editing
│   ├── Visibility
│   ├── Location
│   └── Deletion / Restoration
│
├── Media
│   ├── Upload Gateway
│   ├── Temporary Storage
│   ├── Finalization
│   ├── Assignment
│   └── Removal
│
├── Reaction
│   ├── Add
│   ├── Change
│   └── Remove
│
├── Comment
│   ├── Create
│   ├── Reply
│   ├── Edit
│   ├── Delete
│   └── Tree Rendering
│
├── Sharing
│   ├── Internal Share
│   └── External Share
│
├── Feed
│   ├── Candidate Inventory
│   ├── Eligibility
│   ├── Distribution
│   ├── Ranking
│   ├── Timeline Projection
│   └── Timeline Rendering
│
└── Social Graph Integration
    ├── Friends
    ├── Followers
    ├── Following
    ├── Blocks
    └── Other Relationship Signals
```

These are separate responsibilities.

No single service should become responsible for all of them.

---

# 3. Fundamental Architectural Principle

## 3.1 Business operation is the source of truth

Every meaningful social business transaction must be modeled as a business operation.

Examples:

```text
Create Post
Add Reaction
Change Reaction
Remove Reaction
Create Comment
Create Reply
Edit Comment
Delete Comment
Share Post
Delete Share
Finalize Media
Assign Media
Remove Media Assignment
Change Post Visibility
Delete Post
Restore Post
```

The operation changes the authoritative domain state.

The operation then raises the appropriate domain event.

The event becomes the durable integration point for downstream processing.

---

# 4. Event-Driven Social Architecture

The canonical SocialConnect social workflow is:

```text
Business Operation
        ↓
Application / Domain Service
        ↓
Domain State Change
        ↓
Domain Event
        ↓
Transactional Outbox Persistence
        ↓
Commit
        ↓
Outbox Dispatcher
        ↓
Event Handler
        ↓
Recipient / Target / Audience Resolution
        ↓
Business Impact
        ├── Feed generation
        ├── Feed update
        ├── Notification
        ├── Alert
        ├── Audit
        └── Other domain consumers
        ↓
Tests / Observability / Retry
```

The domain event must be persisted in the same transaction as the business state change whenever the event represents that committed business fact.

Therefore:

> A post must never be considered successfully created while its required domain event is missing from the transactional boundary.

Likewise:

> A reaction must never be persisted successfully while its required event is lost.

---

# 5. Separation of Post Creation and Feed Generation

This is one of the most important requirements of the Social Media architecture.

## 5.1 Post creation is not feed generation

Post creation owns:

* authorization;
* validation;
* content creation;
* post state;
* visibility;
* optional location;
* media references;
* post persistence;
* post domain event.

Post creation does **not** own:

* calculating every recipient;
* creating every feed item;
* ranking;
* feed pagination;
* feed rendering;
* notification delivery;
* alert delivery.

Therefore:

```text
PostCreationService
        ↓
Create Post
        ↓
PostCreated / PostPublished
        ↓
Transactional Outbox
        ↓
Commit
```

is complete as a business transaction.

Feed propagation occurs downstream.

---

# 6. Asynchronous Feed Distribution

SocialConnect Feed Generation is event-driven and may execute asynchronously.

Conceptually:

```text
User creates Post
        ↓
Post Creation Transaction
        ↓
Post persisted
        ↓
PostCreated event persisted
        ↓
Transaction committed
        ↓
Background dispatcher
        ↓
Feed event handler
        ↓
Audience / candidate resolution
        ↓
Feed distribution
        ↓
Feed projections / candidate records
```

This permits the platform to scale independently of post creation.

A successful post creation must not require the HTTP request to synchronously calculate and persist every user's feed.

---

# 7. Feed "Dropbox" / Background Processing Concept

SocialConnect adopts a durable background-processing concept for social distribution.

A newly created business object is first committed as authoritative domain state.

Its event is then placed into the transactional outbox.

The outbox effectively becomes the durable hand-off point between:

```text
Business Transaction
```

and:

```text
Background Social Processing
```

The architecture therefore behaves conceptually like:

```text
                    ┌──────────────────┐
                    │  Post Creation   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Domain Database  │
                    │  + Outbox Event  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Background       │
                    │ Dispatcher       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Feed Processing  │
                    └────────┬─────────┘
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
             User A       User B       User C
             Feed         Feed         Feed
```

The important requirement is durability.

The system must not depend on an in-memory queue that can lose the social operation after the database transaction succeeds.

The transactional Outbox already established by SocialConnect is the canonical durable bridge.

---

# 8. Feed Is a Projection / Distribution System

The Feed subsystem does not become the owner of the Post domain.

A Feed entry represents the distribution/presentation of social content to a user.

Therefore:

```text
Post
```

remains the authoritative content.

The Feed contains a representation/reference to that content.

It does not duplicate the Post as a second authoritative domain object.

---

# 9. Feed Must Not Own Reactions

A critical requirement:

> Feed does not have its own independent reaction system.

A reaction displayed inside a Feed card is still a reaction to the underlying target.

For example:

```text
Feed
 └── Post
      └── Reaction
```

not:

```text
Feed
 ├── FeedReaction
 └── PostReaction
```

The Feed transports interaction to the target.

Therefore:

```text
User clicks Like in Feed
        ↓
Feed identifies target
        ↓
Reaction endpoint
        ↓
ReactionService
        ↓
TargetType + TargetId + UserId
        ↓
Reaction state
```

The Feed UI is merely one presentation surface for the reaction capability.

The same Reaction system can therefore be used from:

* Post detail;
* Feed;
* Comment;
* Media;
* other supported reactable targets.

---

# 10. Reaction Requirements

## 10.1 Single reaction gateway

The Reaction subsystem must expose one canonical mutation endpoint.

Conceptually:

```text
POST /api/reaction/toggle
```

The endpoint receives the target and desired reaction context.

The Reaction service determines whether the operation means:

```text
Add
Change
Remove
```

based on the user's existing reaction state.

The consumer does not need separate endpoints for:

```text
/add
/remove
/change
```

---

# 11. Reaction Decision Model

The canonical reaction transition is:

```text
No existing reaction
        +
New reaction
        ↓
ADD
```

```text
Existing reaction A
        +
New reaction B
        ↓
CHANGE
```

```text
Existing reaction A
        +
Same reaction A
        ↓
REMOVE
```

Therefore the single mutation gateway effectively implements:

```text
Toggle / Resolve Reaction
```

rather than exposing implementation-specific operations to the UI.

---

# 12. Reaction Targets

Reaction targeting uses:

```text
TargetType + TargetId
```

This remains the canonical polymorphic target mechanism.

The Reaction subsystem must not introduce independent target-resolution mechanisms for each consumer.

Supported targets are determined by the canonical Reaction Target catalog.

Known Social Media targets include:

* Post;
* Comment;
* supported Media target;
* other explicitly registered reactable domain objects.

Feed is not itself a reaction target.

---

# 13. Reaction Aggregate

Reactable domain entities may contain the canonical aggregate:

```text
TotalReactionsCount
ReactionAggregate
 ├── LikeCount
 ├── LoveCount
 ├── HahaCount
 ├── WowCount
 ├── SadCount
 └── AngryCount
```

The aggregate belongs to the target domain object.

The Feed merely displays it.

---

# 14. Reaction Business Events

The canonical event model must distinguish the business transition.

At minimum:

```text
ReactionAdded
ReactionChanged
ReactionRemoved
```

Each event represents a committed business fact.

Example:

```text
User selects Love on Post
        ↓
Reaction operation
        ↓
Existing reaction = Like
        ↓
ReactionChanged
        ↓
Transactional Outbox
        ↓
Dispatcher
        ↓
Consumers
```

Consumers may include:

* Feed ranking/signal update;
* notification;
* alert;
* analytics;
* audit;
* future recommendation systems.

Reaction events must not directly manipulate Feed UI.

---

# 15. Comment Requirements

Comments are first-class social objects.

A Comment must support:

* author;
* target;
* content;
* optional media;
* parent comment;
* reply relationship;
* timestamps;
* audit;
* soft deletion;
* reaction support where enabled;
* visibility/authorization;
* notification/alert effects.

Comments may target supported social content using the canonical:

```text
TargetType + TargetId
```

mechanism.

---

# 16. Comment Tree

Comments are represented as a tree.

Conceptually:

```text
Post
│
├── Comment A
│   ├── Reply A1
│   │   └── Reply A1.1
│   └── Reply A2
│
├── Comment B
│   └── Reply B1
│
└── Comment C
```

The domain model must preserve parent-child relationships.

---

# 17. Comment Tree Depth

The maximum comment/reply depth is an administrative policy.

The platform must not hard-code the presentation depth into the reusable CommentManager.

The effective depth is determined by the configured Social/Comment policy.

This permits administrators to control:

* maximum reply depth;
* whether deeper replies are permitted;
* whether deeper levels are collapsed;
* tree rendering behavior;
* pagination/lazy loading behavior;
* moderation constraints.

The current reusable CommentManager already establishes a reply-depth constraint of two for its present UI contract; the final policy must therefore distinguish **domain-allowed depth** from **current component rendering depth**.

---

# 18. Comment Tree Rendering

Comment tree retrieval and rendering must support:

* root comments;
* child replies;
* deterministic ordering;
* pagination;
* lazy expansion;
* collapsed branches;
* authorization;
* deleted comments;
* moderation state;
* maximum configured depth.

The client must not independently reconstruct the authoritative comment tree from arbitrary API calls.

The server provides the correct ViewModel structure.

JavaScript controls interaction such as:

* expand;
* collapse;
* load more;
* reply;
* edit;
* delete.

It does not own business rules.

---

# 19. Comment Media

A Comment may contain **at most one media attachment**.

Therefore:

```text
Comment
 └── 0..1 CommentMedia
```

A comment may contain:

```text
text only
```

or:

```text
text + one media
```

or, where the business contract permits:

```text
media-only comment
```

The exact content validation policy remains server-owned.

---

# 20. Comment Media Must Use MediaUploader

Comment creation must never implement its own upload pipeline.

The reusable:

```text
MediaUploader
```

is the single UI upload gateway.

The CommentManager consumes its public contract.

It does not know:

* temporary storage implementation;
* file-processing implementation;
* physical storage path;
* media finalization internals;
* media assignment internals.

---

# 21. Media Architecture

Media is a cross-cutting platform capability.

It must be generic enough to serve:

* Posts;
* Comments;
* User Profiles;
* Shops;
* Products;
* other supported domains.

The Media module must therefore not be designed specifically around Posts.

---

# 22. MediaUploader as the Unique Upload Gateway

The canonical architecture is:

```text
Consumer
   ↓
MediaUploader
   ↓
/api/media/upload-async
   ↓
MediaUploadService
   ↓
Temporary Storage
   ↓
Media Domain
```

The consumer is never allowed to bypass MediaUploader for ordinary UI upload workflows.

The MediaUploader remains consumer-independent.

---

# 23. Automatic Media Upload

For automatic upload:

```text
User selects file
        ↓
MediaUploader
        ↓
Upload endpoint
        ↓
MediaUploadService
        ↓
Temporary storage
        ↓
Media identifier
        ↓
UI receives upload result
```

The uploaded media is not yet considered permanently part of the final business transaction.

It is temporary/staged media.

---

# 24. Manual Media Upload

The MediaUploader must also support controlled/manual workflows.

The same gateway must support:

```text
Automatic upload
```

and:

```text
Manual upload
```

without requiring consumers to implement a second upload architecture.

The distinction belongs to the consumer workflow/configuration, not to separate media infrastructure.

---

# 25. Media Finalization

Finalization occurs only after the owning business transaction is successfully established.

Canonical flow:

```text
Temporary Media
       ↓
Business Transaction
       ↓
Finalize Media
       ↓
Final Storage
```

Finalization means:

```text
Temporary → Final
```

It does not mean assigning the media to a business entity.

---

# 26. Media Assignment

Assignment is a separate responsibility.

```text
MediaFinalizationService
        ↓
Temporary → Final
```

and:

```text
MediaAssignmentService
        ↓
Finalized Media → Business Owner
```

Therefore:

> Finalization never assigns media.

and:

> Assignment never finalizes temporary media.

This separation is mandatory.

---

# 27. Media Lifecycle

The canonical lifecycle is:

```text
Selected
   ↓
Uploading
   ↓
Uploaded to Temporary Storage
   ↓
Media Domain Record
   ↓
Business Transaction
   ↓
Finalized
   ↓
Assigned
```

Possible failure/cancellation states must be handled without corrupting the owning business transaction.

---

# 28. Media Events

Media lifecycle events may include:

```text
MediaUploaded
MediaFinalized
MediaAssigned
MediaAssignmentRemoved
MediaDeleted
```

However, these events must not automatically be interpreted as Feed-generation events.

For example:

```text
MediaUploaded
```

does not mean:

```text
Create Feed Item
```

The Post business transaction determines whether the media contributes to a published Post.

---

# 29. Post Creation

Post creation is a complete business workflow.

Canonical flow:

```text
Post Composer
      ↓
CreatePostDto
      ↓
PostCreationContext
      ↓
PostCreationService
      ↓
Validate
      ↓
Resolve User/Profile/Location
      ↓
Create Post
      ↓
Finalize Media
      ↓
Assign Media
      ↓
Publish Post
      ↓
Raise Domain Event
      ↓
Transactional Outbox
      ↓
Commit
```

This follows the already locked SocialConnect Post Creation architecture.

---

# 30. Post Content

A Post may support:

* text;
* image;
* video;
* multiple media;
* location;
* visibility;
* comments enabled/disabled;
* reactions enabled/disabled;
* sharing according to policy.

The Post domain owns these business properties.

---

# 31. Post Location

When post location sharing is enabled, SocialConnect may inherit:

```text
CountryId
CityId
```

from the user's profile/context.

When location sharing is disabled:

```text
CountryId = NULL
CityId = NULL
```

The system must not silently expose private profile location through posts.

---

# 32. Post Publication

Creation and publication must be conceptually separated even if the V1 implementation completes both within one application workflow.

The business state must make it possible to distinguish:

```text
Draft / Incomplete
```

from:

```text
Published
```

where future product requirements require staged publication.

Only a publishable/published Post becomes eligible for Feed distribution.

---

# 33. Post Business Events

The Post subsystem must define explicit business events.

Core events include:

```text
PostCreated
PostPublished
PostUpdated
PostVisibilityChanged
PostDeleted
PostRestored
```

Where creation and publication are one atomic V1 business operation, the event contract must still clearly define whether:

```text
PostCreated
```

means merely persisted or:

```text
PostCreated + Published
```

The final event contract must avoid ambiguous semantics.

---

# 34. Recommended V1 Published Post Event

For feed generation, the authoritative feed-producing business fact should be:

```text
PostPublished
```

because Feed should consume content that is actually eligible for distribution.

Conceptually:

```text
Create Post
      ↓
Validate
      ↓
Persist
      ↓
Publish
      ↓
PostPublished
      ↓
Outbox
      ↓
Feed Distribution
```

This prevents Feed from treating an unpublished/incomplete post as distributable content.

---

# 35. Post Update Events

A post update does not necessarily mean a new Feed story.

The Feed handler must distinguish:

```text
Content Update
```

from:

```text
New Distribution Event
```

For example:

```text
PostUpdated
```

may cause existing Feed projections to refresh their representation.

It should not blindly create another Feed item.

---

# 36. Post Deletion

When a Post is deleted:

```text
PostDeleted
        ↓
Outbox
        ↓
Feed Handler
        ↓
Remove / suppress affected Feed representation
```

The Feed must not retain an active presentation of deleted content.

The Post remains the authoritative domain source.

---

# 37. Post Visibility Changes

A visibility change can affect Feed eligibility.

Example:

```text
Public
   ↓
FriendsOnly
```

or:

```text
FriendsOnly
   ↓
Private
```

The resulting event must cause Feed processing to reevaluate the affected distribution.

Therefore:

```text
PostVisibilityChanged
```

is a Feed-relevant event.

---

# 38. Sharing

SocialConnect distinguishes:

```text
Internal Share
External Share
```

## Internal Share

An internal share creates a new Post referencing the original Post.

The canonical model is:

```text
New Post
    └── SharedPostId → Original Post
```

It does not duplicate:

* original content;
* original media;
* original Post identity.

The relationship uses the existing:

```text
ShareType
```

contract.

---

# 39. Internal Share Event

Internal sharing is itself a business transaction.

Therefore it produces a business event.

Conceptually:

```text
User shares Post
       ↓
Create Share/Post
       ↓
SharedPostId
       ↓
PostPublished / PostShared
       ↓
Outbox
       ↓
Feed Distribution
```

The exact final event naming must remain consistent with the canonical Event Catalog.

The important semantic distinction is:

> A share is not merely a UI button click. It is a committed social business operation.

---

# 40. Shared Post Feed Representation

The Feed must be capable of rendering:

```text
User A shared User B's Post
```

without duplicating the original Post content in the database.

The Feed card may therefore contain:

```text
Sharer
+
Share context
+
Original Post reference
+
Original Post presentation
```

The underlying Post remains authoritative.

---

# 41. Shared Post Graph

Shared Posts may reference other shared Posts.

The Feed system must protect itself against:

* cycles;
* unlimited nesting;
* recursive rendering;
* pathological shared graphs.

The existing FeedTimelineContextBuilder requirement therefore remains:

```text
Maximum Share Depth
+
Cycle Protection
```

---

# 42. External Sharing

External sharing is not equivalent to internal Feed distribution.

External sharing may produce:

```text
ExternalShareCreated
```

for audit/analytics/business purposes where required.

It does not automatically create an internal Feed item unless the business operation itself creates an internal SocialConnect Post.

---

# 43. Feed Generation Principles

Feed generation is based on business events.

The Feed subsystem must not continuously poll every Post table and attempt to infer what happened.

Instead:

```text
Business Event
      ↓
Feed Impact
```

is the canonical model.

This makes social consequences traceable.

---

# 44. Feed Event Categories

Feed-related events fall into four conceptual categories.

## Category A — New Content

Events that introduce new candidate content:

```text
PostPublished
InternalPostShared
```

## Category B — Content State

Events that alter eligibility or representation:

```text
PostUpdated
PostVisibilityChanged
PostDeleted
PostRestored
```

## Category C — Engagement Signals

Events that alter ranking/signals:

```text
ReactionAdded
ReactionChanged
ReactionRemoved

CommentCreated
CommentReplied
CommentDeleted

ShareCreated
ShareDeleted
```

These generally do not create a new Feed story.

They update the information used by Feed ranking or re-ranking.

## Category D — Social Graph Changes

Events that alter who may receive or discover content:

```text
FriendshipAccepted
FriendshipRemoved
FollowCreated
FollowRemoved
BlockCreated
BlockRemoved
```

These events may require Feed eligibility recalculation.

---

# 45. Feed Must Distinguish Story Creation from Ranking Signal

This distinction is mandatory.

For example:

```text
PostPublished
```

may create a new Feed candidate.

But:

```text
ReactionAdded
```

does not create another Post in the Feed.

Instead:

```text
ReactionAdded
        ↓
Engagement Signal
        ↓
Existing Feed Candidate Updated
```

Likewise:

```text
CommentCreated
```

may affect ranking or action-bumping without creating a duplicate Feed story.

Facebook publicly describes engagement and conversation as signals that can influence Feed ranking, while its Feed system separately builds candidate inventory and ranks that inventory.

---

# 46. Feed Candidate Inventory

Feed processing must conceptually maintain or calculate an eligible candidate inventory.

A candidate must satisfy relevant conditions such as:

* content exists;
* content is published;
* content is not deleted;
* content is visible to the viewer;
* viewer is not blocked;
* author/content relationship is eligible;
* content policy permits distribution;
* target audience permits distribution.

The candidate inventory is then available to ranking/distribution logic.

---

# 47. Feed Eligibility

Feed eligibility must be determined independently from Feed rendering.

The architecture is:

```text
Feed Candidate
      ↓
Eligibility
      ↓
Ranking / Ordering
      ↓
Feed Timeline Context
      ↓
Feed Card
      ↓
View
```

The Feed ViewComponent must not decide eligibility.

---

# 48. Feed Ranking

Feed ranking is a separate responsibility from Feed candidate generation.

Potential signals include:

* recency;
* relationship strength;
* author relationship;
* previous interaction;
* reactions;
* comments;
* shares;
* content type;
* location relevance;
* user preferences;
* visibility;
* negative feedback;
* social graph changes;
* other approved ranking signals.

Facebook has publicly described Feed as using many signals and prediction/ranking stages rather than a single simplistic chronological list.

SocialConnect therefore reserves a dedicated Feed ranking boundary.

---

# 49. Feed Timeline Architecture

The locked SocialConnect Feed architecture remains:

```text
FeedTimelineService
        ↓
FeedTimelineContextBuilder
        ↓
FeedReactionResolver
        ↓
FeedCardFactory
```

Responsibilities:

### FeedTimelineService

Orchestrates the Feed workflow.

### FeedTimelineContextBuilder

Builds the required context and loads the relevant social graph/content data.

### FeedReactionResolver

Resolves reaction presentation from the existing reaction data/context.

It does not duplicate Reaction business logic.

### FeedCardFactory

Maps prepared context into Feed ViewModels.

It should remain pure mapping.

---

# 50. Feed Reaction Resolver Boundary

Two resolver concepts remain distinct.

```text
IReactionTargetResolver
```

is responsible for loading reactable domain targets for the Reaction subsystem.

```text
IFeedReactionTargetResolver
```

maps:

```text
EntityOwnerType
        ↓
ReactionTargetType
```

for Feed presentation.

These must not be collapsed into one service.

---

# 51. Feed Card Factory

FeedCardFactory must not:

* query repositories;
* perform authorization;
* execute reaction business logic;
* generate notifications;
* create events;
* calculate Feed eligibility;
* rank candidates.

It receives prepared context and maps it into presentation models.

---

# 52. Feed UI

The Feed UI is a presentation layer.

It renders:

* author;
* timestamp;
* content;
* media;
* location;
* sharing context;
* reaction summary;
* comment summary;
* share information;
* interaction controls.

It does not own the underlying business operations.

---

# 53. Feed and Media

Feed media is a representation of the underlying Post/Comment/Media domain.

Feed does not upload media.

Feed does not finalize media.

Feed does not assign media.

Feed consumes media that has already been processed through the canonical Media subsystem.

---

# 54. Feed and Comments

Feed may display:

* comment count;
* selected comments;
* comment composer;
* replies;
* comment interaction.

But CommentManager remains the owner of comment interaction.

Feed embeds/configures the reusable CommentManager rather than implementing another comment system.

---

# 55. Feed and Reactions

Feed may display:

```text
Like
Love
Haha
Wow
Sad
Angry
```

but Feed does not implement these operations.

The action is transported to:

```text
Reaction subsystem
```

using:

```text
TargetType + TargetId
```

---

# 56. Feed and Sharing

Feed may expose the Share operation.

The actual business operation belongs to:

```text
Sharing / Post Creation
```

not Feed.

Feed simply initiates the operation against the underlying Post.

---

# 57. Social Graph and Feed

The Feed must be able to consume social graph relationships such as:

* friends;
* followers;
* followed users;
* followed shops/pages where applicable;
* blocks;
* relationship changes.

These relationships influence candidate eligibility and/or ranking.

They do not become Feed-owned data.

---

# 58. Business Event → Feed Impact Matrix

The following matrix is the canonical conceptual contract.

| Business Event          | New Feed Story | Existing Feed Update | Ranking Signal | Eligibility Re-evaluation |
| ----------------------- | -------------: | -------------------: | -------------: | ------------------------: |
| `PostPublished`         |            Yes |             Possible |            Yes |                       Yes |
| `PostUpdated`           |             No |                  Yes |       Possible |                  Possible |
| `PostVisibilityChanged` |             No |                  Yes |       Possible |                       Yes |
| `PostDeleted`           |             No |           Yes/Remove |             No |                       Yes |
| `PostRestored`          |             No |                  Yes |       Possible |                       Yes |
| `InternalPostShared`    |            Yes |             Possible |            Yes |                       Yes |
| `ReactionAdded`         |             No |             Possible |            Yes |       No/Policy-dependent |
| `ReactionChanged`       |             No |             Possible |            Yes |       No/Policy-dependent |
| `ReactionRemoved`       |             No |             Possible |            Yes |       No/Policy-dependent |
| `CommentCreated`        |             No |             Possible |            Yes |       No/Policy-dependent |
| `CommentReplied`        |             No |             Possible |            Yes |       No/Policy-dependent |
| `CommentDeleted`        |             No |             Possible |       Possible |       No/Policy-dependent |
| `FriendshipAccepted`    |             No |             Possible |            Yes |                       Yes |
| `FriendshipRemoved`     |             No |             Possible |            Yes |                       Yes |
| `FollowCreated`         |             No |             Possible |            Yes |                       Yes |
| `FollowRemoved`         |             No |             Possible |            Yes |                       Yes |
| `BlockCreated`          |             No |                  Yes |             No |                       Yes |
| `BlockRemoved`          |             No |                  Yes |             No |                       Yes |
| `MediaUploaded`         |             No |                   No |             No |                        No |
| `MediaFinalized`        |             No |                   No |             No |                        No |
| `MediaAssigned`         |             No |             Possible |             No |                        No |

The exact event names remain governed by the canonical Event Catalog.

The important architectural rule is the distinction between:

```text
New Story
```

and:

```text
Signal / Projection Update
```

---

# 59. Notification Integration

Social business events may also feed the Notification subsystem.

For example:

```text
CommentCreated
        ↓
Recipient Resolution
        ↓
Notification
```

or:

```text
ReactionAdded
        ↓
Recipient Resolution
        ↓
Notification
```

Feed and Notification are independent consumers.

Therefore:

```text
PostPublished
       ├── Feed Handler
       ├── Notification Handler
       ├── Alert Handler
       └── Other Consumers
```

No Feed service should call NotificationService directly as part of its internal logic.

---

# 60. Alert Integration

Alerting is likewise a separate consumer.

A Social Media operation may produce:

```text
Domain Event
```

which is then evaluated by:

```text
Alert Source / Policy
```

The generic Alerting platform must not contain a giant switch such as:

```text
if PostCreated...
if CommentCreated...
if ReactionCreated...
```

Instead, individual operational conditions plug into the established Alert source/policy boundary.

---

# 61. Event / Notification / Alert Separation

The architecture must remain:

```text
Business Operation
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Dispatcher
        ↓
Multiple Consumers
        ├── Feed
        ├── Notification
        ├── Alert
        ├── Audit
        └── Future Consumers
```

No consumer becomes the owner of the event.

---

# 62. Domain Event vs Technical Event

A Domain Event represents a business fact.

Examples:

```text
PostPublished
ReactionAdded
CommentCreated
InternalPostShared
PostDeleted
```

Technical operations such as:

```text
MediaUploadStarted
HTTP request received
Feed card rendered
JavaScript click occurred
```

are not automatically domain events.

This distinction prevents event infrastructure from becoming an implementation-event bus.

---

# 63. Event Payload Requirements

Each social event must contain enough information for downstream consumers without forcing consumers to reconstruct the entire operation from UI state.

A typical event should identify:

* EventId;
* aggregate/entity identifier;
* actor/user identifier where applicable;
* target identifier/type where applicable;
* operation timestamp;
* relevant state transition;
* correlation information;
* event version;
* event metadata required by the canonical Event infrastructure.

Events must remain stable contracts.

---

# 64. Event Idempotency

Every Feed consumer must be idempotent.

The same event may be delivered more than once because of:

* retries;
* dispatcher restart;
* worker failure;
* network/database interruption;
* lease expiration.

Therefore:

```text
Same EventId
+
Same Consumer
=
No duplicate business effect
```

This applies especially to Feed distribution.

---

# 65. Feed Distribution Idempotency

A Post must not generate duplicate Feed representations merely because:

```text
PostPublished
```

is delivered twice.

The Feed processing layer must have a deterministic idempotency strategy.

Conceptually:

```text
EventId
+
Consumer
+
TargetUser
+
FeedStory
```

must resolve to one logical distribution.

---

# 66. Background Worker Requirements

Feed processing may be performed by background workers.

Workers must support:

* durable event consumption;
* retries;
* lease/claim behavior;
* idempotency;
* failure recovery;
* observability;
* logging;
* concurrency safety;
* controlled throughput.

Feed workers must never bypass the canonical event/outbox infrastructure.

---

# 67. Feed Consistency Model

SocialConnect should distinguish:

### Strong consistency

Required for:

* Post persistence;
* Reaction state;
* Comment persistence;
* Media assignment;
* Share transaction;
* domain event persistence.

### Eventual consistency

Allowed for:

* Feed propagation;
* Feed ranking updates;
* notification delivery;
* alert evaluation;
* derived counters/projections where explicitly designed.

This is fundamental to scalable social architecture.

---

# 68. User Experience Requirement

The user should see the newly created Post as successfully created once the authoritative transaction succeeds.

Feed propagation may complete shortly afterward.

The system must not falsely report:

```text
Post creation failed
```

merely because background Feed distribution is still processing.

Likewise, Feed workers must not silently create authoritative Posts.

---

# 69. Facebook-Class Feed Principles

SocialConnect's Feed should follow the same high-level product principles publicly described by Facebook:

* personalized content;
* relationship-aware content;
* recency;
* engagement signals;
* meaningful interaction;
* candidate selection;
* ranking;
* feedback;
* content diversity;
* user controls;
* continuous refinement.

Facebook has explicitly described Feed as combining candidate inventory, signals, predictions and ranking, with engagement and relationship signals among the inputs.

SocialConnect should therefore avoid defining Feed as simply:

```sql
SELECT TOP 50 Posts
ORDER BY CreatedAt DESC
```

That would not satisfy the intended social-media architecture.

---

# 70. Feed Does Not Mean Chronological Timeline

A chronological view may exist as one Feed mode.

It must not become the definition of the entire Feed architecture.

The Feed engine must remain capable of:

```text
Candidate Generation
→ Eligibility
→ Ranking
→ Ordering
→ Presentation
```

This allows future support for:

* Latest;
* personalized Home;
* friends;
* following;
* nearby;
* shop/social feeds;
* recommendation surfaces.

Facebook itself distinguishes a personalized Home experience from a more recent/connection-oriented Feeds experience.

---

# 71. Social Media Content Lifecycle

The complete social content lifecycle is:

```text
Compose
   ↓
Validate
   ↓
Upload Media
   ↓
Temporary Media
   ↓
Create Business Entity
   ↓
Finalize Media
   ↓
Assign Media
   ↓
Publish
   ↓
Domain Event
   ↓
Outbox
   ↓
Background Distribution
   ↓
Feed Candidate
   ↓
Eligibility
   ↓
Ranking
   ↓
Feed Presentation
   ↓
User Interaction
   ├── Reaction
   ├── Comment
   └── Share
         ↓
     New Domain Event
         ↓
     Further Impact
```

This creates the intended event-driven social loop.

---

# 72. End-to-End Post Scenario

A complete Post scenario must be documented and tested as:

```text
User submits Post
        ↓
Post Composer
        ↓
CreatePostDto
        ↓
PostCreationContext
        ↓
PostCreationService
        ↓
Validate
        ↓
Create Post
        ↓
Finalize Media
        ↓
Assign Media
        ↓
Publish
        ↓
PostPublished
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Background Dispatcher
        ↓
Feed Handler
        ↓
Resolve Eligible Audience
        ↓
Create/Update Feed Distribution
        ↓
Feed Available
```

Separately:

```text
PostPublished
        ├── Notification processing
        ├── Alert processing
        └── Other consumers
```

---

# 73. End-to-End Reaction Scenario

```text
User clicks Love
        ↓
Feed/Post/Comment UI
        ↓
Reaction endpoint
        ↓
Reaction Service
        ↓
Resolve Target
        ↓
Existing Reaction?
        ├── No → Add
        ├── Same → Remove
        └── Different → Change
        ↓
Persist Reaction
        ↓
Update Reaction Aggregate
        ↓
Raise Domain Event
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Dispatcher
        ├── Feed Signal Handler
        ├── Notification Handler
        ├── Alert Handler
        └── Other Consumers
```

---

# 74. End-to-End Comment Scenario

```text
User submits Comment
        ↓
CommentManager
        ↓
Comment API
        ↓
Comment Service
        ↓
Validate target
        ↓
Validate parent/depth
        ↓
Validate content
        ↓
Validate media count ≤ 1
        ↓
Create Comment
        ↓
Assign finalized media
        ↓
Persist
        ↓
CommentCreated / CommentReplied
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Consumers
        ├── Feed signal update
        ├── Notification
        ├── Alert
        └── Audit
```

---

# 75. End-to-End Share Scenario

```text
User selects Share
        ↓
Share workflow
        ↓
Validate original Post
        ↓
Create Internal Share Post
        ↓
Set SharedPostId
        ↓
Do NOT duplicate original media/content
        ↓
Publish
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Background Feed Distribution
        ↓
Shared Post appears in eligible feeds
```

---

# 76. End-to-End Comment Media Scenario

```text
User selects one image/video
        ↓
CommentManager
        ↓
MediaUploader
        ↓
Upload Gateway
        ↓
Temporary Storage
        ↓
Media ID
        ↓
Comment Creation
        ↓
Comment persisted
        ↓
Media finalized
        ↓
Media assigned to Comment
        ↓
CommentCreated / CommentReplied
```

The CommentManager never implements:

```text
file storage
image processing
finalization
assignment
```

---

# 77. Social Media Component Boundaries

## Post Composer

Owns:

* post composition UI;
* client interaction;
* media uploader integration;
* form state;
* submit/cancel;
* rendering.

Does not own:

* Post business rules;
* Feed generation;
* notification;
* media storage.

## MediaUploader

Owns:

* upload interaction;
* upload state;
* temporary upload communication;
* file queue;
* upload lifecycle;
* upload events.

Does not own:

* Post;
* Comment;
* Feed;
* final business transaction.

## Reaction Component

Owns:

* reaction UI;
* selected state presentation;
* interaction;
* calling reaction endpoint.

Does not own:

* reaction persistence;
* target authorization;
* business transition logic.

## CommentManager

Owns:

* comment UI;
* tree interaction;
* reply/edit states;
* comment API interaction;
* lazy expansion.

Does not own:

* comment business rules;
* media storage;
* notification;
* Feed generation.

## Feed Component

Owns:

* Feed presentation;
* interaction wiring;
* Feed card rendering.

Does not own:

* Feed generation;
* ranking;
* Post creation;
* Reaction persistence;
* Comment persistence.

---

# 78. Reusable UI Contract

Every reusable Social Media component follows:

```text
ViewComponent
    ↓
ViewModel
    ↓
Builder
    ↓
Defaults
    ↓
Razor
    ↓
Component JS
    ↓
Component CSS
```

The component must be configured through its Builder.

Consumers communicate with the component only through public contracts.

---

# 79. JavaScript Requirements

Social Media JavaScript must:

* use `App.Modules.register`;
* use WeakMap where instance storage is required;
* use immutable/frozen configuration/constants where appropriate;
* expose public APIs only;
* implement component lifecycle;
* support `init`;
* support `bind`;
* support `destroy`;
* use `App.Events`.

`site.js` remains foundation-only.

No Social Media feature API or business logic is placed in `site.js`.

---

# 80. Server Ownership

The server remains authoritative for:

* authorization;
* visibility;
* privacy;
* target validation;
* relationship rules;
* media ownership;
* comment depth;
* moderation;
* reaction transitions;
* sharing rules;
* Feed eligibility;
* business events.

JavaScript must never be trusted for these decisions.

---

# 81. Security Requirements

Every social operation must validate:

* authenticated user;
* target existence;
* target visibility;
* target ownership/permission;
* blocked relationships;
* deleted state;
* moderation state;
* feature availability;
* anti-forgery where applicable;
* request validation.

A user must never be able to manipulate another user's social content simply by changing:

```text
TargetId
```

or:

```text
TargetType
```

in a request.

---

# 82. Soft Delete

Social Media entities participating in the established soft-delete architecture must use the canonical lifecycle.

Deletion must produce the appropriate business event where downstream consumers need to react.

Example:

```text
PostDeleted
```

must allow Feed and other consumers to suppress the content.

The Feed must not independently decide that a deleted Post still exists.

---

# 83. Audit

Important social mutations must participate in the established audit architecture.

Examples:

* Post deletion;
* moderation-related changes;
* visibility changes;
* comment deletion;
* administrative intervention;
* account-level social restrictions.

Audit is a cross-cutting concern and must not be implemented separately inside every social service.

---

# 84. Administration Integration

Administration controls policy.

It does not directly manipulate social tables.

For example:

```text
Admin
  ↓
Comment Policy
  ↓
Configured Maximum Reply Depth
  ↓
Comment Service
```

not:

```text
Admin Controller
  ↓
Comment Table
```

Likewise:

```text
Admin
  ↓
Moderation Policy
  ↓
Social Domain/Application Service
```

remains the canonical path.

---

# 85. Moderation Integration

Social Media must integrate with the canonical Administration moderation system.

Potential moderation actions include:

* hide;
* remove;
* restrict;
* suspend;
* escalate.

Moderation is not implemented as a special Feed rule.

A moderated Post remains governed by the authoritative Post/moderation state, and Feed eligibility responds to the resulting event/state.

---

# 86. Notification Integration Contract

Social events may produce notifications.

Examples include:

```text
PostPublished
CommentCreated
CommentReplied
ReactionAdded
ReactionChanged
InternalPostShared
```

The exact recipient rules belong to Recipient Resolution and Notification Policy.

Social services must not directly decide notification presentation.

---

# 87. Alert Integration Contract

Social events may become Alert sources.

Alerting remains generic.

The Social Media subsystem contributes source/policy definitions rather than modifying the generic Alert engine.

This preserves:

```text
Generic Alert Platform
+
Domain-specific Alert Sources
```

instead of:

```text
One Giant Alert Switch
```

---

# 88. Feed Event Processing Requirements

For every Feed-relevant event, the implementation documentation must explicitly describe:

```text
1. Business operation
2. Domain state change
3. Domain event raised
4. Transactional Outbox persistence
5. Dispatcher
6. Event handler
7. Audience / candidate resolution
8. Feed eligibility
9. Feed distribution
10. Ranking/signal update
11. Idempotency
12. Failure/retry behavior
13. Notification impact
14. Alert impact
15. Tests
```

This sequence becomes mandatory for future Social Media event documentation.

---

# 89. Required Event Specifications

Before implementation of a Social Media event, each event must receive its own event specification containing:

```text
Event Name
Purpose
Business Trigger
Aggregate
Actor
Target
Payload
Transaction Boundary
Outbox Requirements
Consumers
Feed Impact
Notification Impact
Alert Impact
Idempotency
Retry Behaviour
Authorization Context
Audit Requirements
Failure Behaviour
Tests
```

No event should be implemented merely because a UI action exists.

---

# 90. Required Social Media Event Catalog

The canonical event catalog should be established around the following groups.

## Post

```text
PostCreated
PostPublished
PostUpdated
PostVisibilityChanged
PostDeleted
PostRestored
```

## Sharing

```text
InternalPostShared
InternalPostShareRemoved
ExternalPostShared
```

Exact names remain subject to the canonical Event Catalog naming contract.

## Reaction

```text
ReactionAdded
ReactionChanged
ReactionRemoved
```

## Comment

```text
CommentCreated
CommentReplied
CommentUpdated
CommentDeleted
CommentRestored
```

## Media

```text
MediaUploaded
MediaFinalized
MediaAssigned
MediaAssignmentRemoved
MediaDeleted
```

## Social Graph

```text
FriendshipAccepted
FriendshipRemoved
FollowCreated
FollowRemoved
BlockCreated
BlockRemoved
```

Only events that represent actual business transitions should be finalized in the canonical Event Catalog.

---

# 91. Event Naming Rule

Events must describe what **happened**, not what a consumer intends to do.

Good:

```text
PostPublished
ReactionAdded
CommentCreated
```

Avoid:

```text
GenerateFeedForPost
SendNotificationForComment
CreateAlertForReaction
```

The latter are consumer commands, not domain facts.

---

# 92. Feed Consumer Naming

Feed handlers may consume domain events and perform Feed work.

For example:

```text
PostPublished
      ↓
PostPublishedFeedHandler
```

The handler is a consumer.

It must not redefine:

```text
PostPublished
```

as a Feed-specific domain event.

---

# 93. Feed Distribution Does Not Create Business Truth

A Feed record is derived from business truth.

Therefore:

```text
Feed missing
```

does not mean:

```text
Post does not exist
```

The recovery mechanism is:

```text
Post/Event
        ↓
Feed projection/distribution
```

not manual reconstruction of the Post.

---

# 94. Feed Rebuild Requirement

Because Feed is derived from authoritative domain events/state, the architecture should remain capable of rebuilding Feed projections where required.

The system must not make Feed the only source of information about social content.

This is particularly important for:

* data recovery;
* projection rebuilds;
* ranking changes;
* new Feed surfaces;
* future recommendation systems.

---

# 95. Performance Requirements

The architecture must avoid:

* synchronous fan-out to thousands of users during Post creation;
* repeated database queries per Feed card;
* duplicate target loading;
* N+1 relationship queries;
* duplicate media queries;
* repeated Reaction target resolution.

The Feed context builder must prepare the required context efficiently.

The Feed card factory must remain pure.

---

# 96. Social Media Read vs Write Separation

Write operations:

```text
Post Creation
Comment Creation
Reaction Mutation
Share
Media Assignment
```

must be handled by application/domain services.

Read operations:

```text
Feed Timeline
Comments
Reaction summaries
Post details
Profile timeline
```

must use appropriate query/read services.

Controllers remain thin.

---

# 97. Repository Boundary

Business services must not directly manipulate repositories as part of business orchestration where the locked loader/service architecture applies.

Use:

```text
Application Service
        ↓
Domain/Application operation
        ↓
Loader / Lookup / Repository infrastructure
```

according to the established SocialConnect architecture.

Repositories remain persistence-oriented.

---

# 98. No God Services

The Social Media subsystem must not create a service such as:

```text
SocialMediaService
```

that handles:

* Posts;
* Comments;
* Reactions;
* Media;
* Sharing;
* Feed;
* Notifications;
* Alerts.

Instead, responsibilities remain separated.

---

# 99. Canonical Responsibility Map

| Capability                 | Owner                              |
| -------------------------- | ---------------------------------- |
| Post creation              | Post Creation application workflow |
| Post business state        | Post domain                        |
| Media upload               | MediaUploader / MediaUploadService |
| Media finalization         | MediaFinalizationService           |
| Media assignment           | MediaAssignmentService             |
| Reaction mutation          | Reaction service                   |
| Reaction target resolution | Reaction target resolver           |
| Comment mutation           | Comment service                    |
| Comment tree policy        | Comment/Admin policy               |
| Comment rendering          | CommentManager                     |
| Sharing                    | Share/Post workflow                |
| Feed orchestration         | FeedTimelineService                |
| Feed context               | FeedTimelineContextBuilder         |
| Feed reaction mapping      | FeedReactionResolver               |
| Feed card mapping          | FeedCardFactory                    |
| Feed distribution          | Feed processing/handler            |
| Feed ranking               | Feed ranking boundary              |
| Notification               | Notification subsystem             |
| Alerting                   | Alerting subsystem                 |
| Audit                      | Canonical audit infrastructure     |

---

# 100. Acceptance Criteria

The Social Media subsystem is considered architecturally complete only when:

### Posts

* Posts can be created through the canonical Post workflow.
* Posts can contain text and supported media.
* Visibility is enforced server-side.
* Location is handled according to policy.
* Post publication produces the appropriate event.
* Post deletion affects downstream Feed state.

### Media

* MediaUploader is the single reusable upload gateway.
* Automatic upload is supported.
* Manual upload is supported.
* Temporary storage is distinct from final storage.
* Finalization is separate from assignment.
* Consumers do not implement their own upload pipelines.
* Comments support at most one media attachment.

### Reactions

* One canonical mutation endpoint exists.
* Add/change/remove transitions are resolved by the service.
* TargetType + TargetId is used.
* Feed does not own reaction persistence.
* Reaction events are emitted.

### Comments

* Comment trees are supported.
* Parent-child relationships are authoritative.
* Maximum depth is policy-controlled.
* Tree rendering supports expansion/collapse.
* CommentManager remains reusable.
* Comment media uses MediaUploader.
* One comment has at most one media attachment.

### Sharing

* Internal shares create a new Post/reference.
* Shared content/media are not duplicated.
* Shared Post graphs have depth/cycle protection.
* Sharing produces a business event.

### Feed

* Feed generation is separate from Post creation.
* Feed processing is event-driven.
* Feed can execute asynchronously.
* Feed is not the authoritative content store.
* Feed does not own reactions.
* Feed does not own comments.
* Feed consumes social events.
* Ranking is separate from candidate generation.
* Feed distribution is idempotent.

### Events

* Social business operations produce domain events.
* Events are persisted transactionally through Outbox.
* Dispatcher processing is durable.
* Handlers are idempotent.
* Feed/Notification/Alert are independent consumers.
* Event contracts are explicitly documented.
* Each Feed-relevant event has an end-to-end specification.

---

# 101. Canonical Social Media Architecture

The complete architecture can therefore be summarized as:

```text
                         SOCIALCONNECT
                         SOCIAL MEDIA
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
        ▼                     ▼                      ▼
      POST                 COMMENT                REACTION
        │                     │                      │
        │                     │                      │
        └──────────────┬──────┴──────────────┬───────┘
                       │                     │
                       ▼                     ▼
                    MEDIA                SHARING
                       │                     │
                       └──────────┬──────────┘
                                  │
                                  ▼
                         BUSINESS TRANSACTION
                                  │
                                  ▼
                           DOMAIN EVENT
                                  │
                                  ▼
                         TRANSACTIONAL OUTBOX
                                  │
                                  ▼
                             DISPATCHER
                                  │
                ┌─────────────────┼──────────────────┐
                │                 │                  │
                ▼                 ▼                  ▼
              FEED          NOTIFICATION          ALERT
                │
                ▼
       CANDIDATE INVENTORY
                │
                ▼
           ELIGIBILITY
                │
                ▼
            RANKING
                │
                ▼
        FEED DISTRIBUTION
                │
                ▼
        FEED TIMELINE CONTEXT
                │
                ▼
          FEED CARD FACTORY
                │
                ▼
               UI
                │
       ┌────────┼─────────┐
       ▼        ▼         ▼
    React    Comment     Share
       │        │         │
       └────────┴─────────┘
                │
                ▼
        NEW BUSINESS EVENTS
```

---

# 102. Final Architectural Contract

The SocialConnect Social Media subsystem is governed by the following principles:

1. **Posts are authoritative social content.**
2. **Media is a generic cross-cutting platform capability.**
3. **MediaUploader is the unique reusable upload gateway.**
4. **Temporary media is not final business media.**
5. **Finalization and assignment are separate responsibilities.**
6. **Comments use the same generic MediaUploader.**
7. **A Comment may contain at most one media attachment.**
8. **Reactions have one canonical mutation gateway.**
9. **Reaction state is determined server-side as add/change/remove.**
10. **TargetType + TargetId remains the canonical polymorphic target mechanism.**
11. **Feed does not own reactions.**
12. **Feed does not own comments.**
13. **Feed is a distribution/presentation projection of authoritative social content.**
14. **Post creation and Feed generation are separate responsibilities.**
15. **Feed generation is event-driven.**
16. **Feed propagation may execute asynchronously through durable background processing.**
17. **Transactional Outbox is the durable bridge between business transactions and downstream social processing.**
18. **Every important social business operation produces an explicit domain event.**
19. **Domain events describe business facts, not consumer commands.**
20. **Feed handlers consume events; they do not redefine domain ownership.**
21. **Engagement events update Feed signals rather than creating duplicate Feed stories.**
22. **Post publication creates Feed candidates; reactions/comments generally modify signals.**
23. **Social graph changes can alter Feed eligibility.**
24. **Feed ranking is separate from Feed candidate generation.**
25. **Feed rendering is separate from Feed generation.**
26. **Comment tree policy is administratively configurable.**
27. **Reusable components own their interaction domain but never own business authority.**
28. **Notification and Alerting consume social events independently of Feed.**
29. **No social subsystem may become a God service.**
30. **Every Feed-relevant event must be documented through the complete business-event lifecycle.**
31. **All downstream consumers must be idempotent.**
32. **Authoritative social state always remains in its owning domain, never in Feed.**
33. **Facebook-class functionality is the product benchmark; SocialConnect's implementation remains governed by its own canonical architecture and contracts.**

---

# 103. Implementation Sequence

The implementation should follow the architecture rather than mixing all social features together.

## Phase 1 — Contract Finalization

Finalize:

* Social Media requirements;
* Post requirements;
* Media requirements;
* Reaction requirements;
* Comment requirements;
* Sharing requirements;
* Feed requirements;
* Social Event Catalog;
* Feed Event Impact Matrix.

## Phase 2 — Media

Finalize and verify:

```text
MediaUploader
MediaUploadService
Temporary Storage
MediaFinalizationService
MediaAssignmentService
```

## Phase 3 — Post

Finalize:

```text
Post Creation
Post Publication
Post Visibility
Post Media
Post Events
```

## Phase 4 — Reaction

Finalize:

```text
Reaction Target Resolution
Reaction Mutation
Reaction Aggregate
Reaction Events
```

## Phase 5 — Comment

Finalize:

```text
Comment Creation
Comment Reply
Comment Tree
Comment Policy
Comment Media
Comment Events
```

## Phase 6 — Sharing

Finalize:

```text
Internal Share
SharedPost Graph
Cycle Protection
Share Events
```

## Phase 7 — Feed

Finalize:

```text
Candidate Generation
Eligibility
Distribution
Ranking Boundary
FeedTimelineContextBuilder
FeedReactionResolver
FeedCardFactory
FeedTimelineService
```

## Phase 8 — Event Vertical Slices

For every Feed-relevant event:

```text
Business Operation
→ Event
→ Outbox
→ Dispatcher
→ Handler
→ Candidate/Audience Resolution
→ Feed Impact
→ Tests
```

## Phase 9 — Notification / Alert Vertical Slices

Then prove:

```text
Business Operation
→ Event
→ Outbox
→ Dispatcher
→ Event Handler
→ Recipient Resolution
→ Notification
→ Alert
→ Feed Impact
→ Tests
```

## Phase 10 — End-to-End Social Verification

Finally verify complete vertical slices:

```text
Post Creation
→ PostPublished
→ Outbox
→ Feed Distribution
→ Feed Rendering
→ Reaction
→ Reaction Event
→ Feed Signal
→ Comment
→ Comment Event
→ Notification
→ Alert
```

---

# 104. Definition of Done

The Social Media subsystem is not considered complete merely because:

```text
Post CRUD works
```

or:

```text
Feed displays Posts
```

It is complete when the **business-event-driven social lifecycle** is demonstrably working.

The final proof must show:

```text
Business Operation
        ↓
Domain State
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Dispatcher
        ↓
Handler
        ↓
Feed / Notification / Alert Impact
        ↓
Idempotency
        ↓
Failure / Retry
        ↓
Correct UI Projection
```

That sequence is the core architectural contract of SocialConnect's Social Media subsystem.
