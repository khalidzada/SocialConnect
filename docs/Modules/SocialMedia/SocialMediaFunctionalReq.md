# SocialConnect — Social Media Domain & Functional Requirements

**Status:** Canonical Requirements Contract
**Scope:** Social Media subsystem
**Version:** 1.0
**Platform:** ASP.NET Core MVC / .NET 8 / EF Core / SQL Server
**Architecture:** Layered Modular Monolith
**Primary Domains:** Posts, Media, Reactions, Comments, Sharing, Feed Generation, Social Graph Integration
**Cross-Cutting Platforms:** Domain Events, Transactional Outbox, Event Dispatcher, Notification, Alerting, Audit, Authorization, Worker Health, External Dependency Adapters

---

# 1. Purpose

The Social Media subsystem is the core social-network capability of SocialConnect.

It provides users with the ability to:

* create social posts;
* attach media to posts;
* react to supported social targets;
* create comments and replies;
* attach media to comments;
* share posts;
* consume personalized feed content;
* interact with content presented through feeds;
* participate in social relationships that influence content eligibility and ranking;
* receive downstream notification effects from social activity;
* participate in a social environment governed by authorization, moderation, privacy, and administrative policy.

The subsystem is intended to provide a **Facebook-class social capability and behavioral model** at the product-function level while remaining a native SocialConnect implementation.

This means SocialConnect may adopt established social-media concepts such as:

* personalized feeds;
* social-graph-driven content eligibility;
* engagement signals;
* ranking;
* comments and replies;
* reactions;
* sharing;
* media-rich posts;
* privacy and visibility;
* downstream notifications.

It does **not** mean reproducing another platform's proprietary source code, database schema, implementation, ranking algorithm, infrastructure, or proprietary internal behavior.

The Social Media subsystem is governed by the following fundamental principle:

```text
Business Operation
        ↓
Authoritative Domain State Change
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Event Dispatcher
        ↓
Event Consumer / Handler
        ↓
Business Impact
        ├── Feed
        ├── Notification
        ├── Alerting
        ├── Audit
        ├── Projection / Cache
        └── Other Approved Consumer
```

A business operation is therefore independent from its downstream consequences.

---

# 2. Scope

The Social Media subsystem covers:

1. Post creation and lifecycle;
2. Post visibility;
3. Post location;
4. Post media;
5. Reactions;
6. Comments and replies;
7. Comment media;
8. Internal and external sharing;
9. Feed candidate generation;
10. Feed eligibility;
11. Feed distribution;
12. Feed ranking;
13. Feed timeline projection;
14. Feed rendering;
15. Social graph integration;
16. downstream Notification integration;
17. downstream Alerting integration;
18. moderation and administrative policy integration;
19. social-domain event integration;
20. background processing required for scalable Feed distribution.

The subsystem does **not** own:

* the generic Event infrastructure;
* the Transactional Outbox infrastructure;
* the generic Notification platform;
* the generic Alerting platform;
* application-wide Worker Health;
* external vendor integrations;
* Identity infrastructure;
* Administration infrastructure;
* generic media storage infrastructure outside the Media module.

Those capabilities are consumed through their established contracts.

---

# 3. Architectural Position

The Social Media subsystem consists of distinct capabilities:

```text
Social Media
│
├── Post
│   ├── Creation
│   ├── Lifecycle
│   ├── Visibility
│   ├── Location
│   ├── Media
│   └── Deletion
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
│   ├── Creation
│   ├── Reply
│   ├── Editing
│   ├── Deletion
│   └── Tree Rendering
│
├── Sharing
│   ├── Internal Share
│   └── External Share
│
├── Feed
│   ├── Candidate Generation
│   ├── Eligibility
│   ├── Distribution
│   ├── Ranking
│   ├── Projection
│   └── Rendering
│
└── Social Graph Integration
    ├── Friends
    ├── Followers
    ├── Following
    ├── Blocks
    └── Other Relationship Signals
```

These capabilities have separate responsibilities.

No single Social Media service may become the owner of all of them.

---

# 4. Fundamental Business Principle

Every meaningful social business operation must have an authoritative business owner.

Examples include:

```text
Create Post
Update Post
Change Post Visibility
Delete Post

Add Reaction
Change Reaction
Remove Reaction

Create Comment
Edit Comment
Delete Comment

Create Internal Share
Create External Share

Upload Media
Finalize Media
Assign Media
Remove Media Assignment
```

The operation:

1. validates the request;
2. authorizes the operation;
3. changes authoritative domain state;
4. raises the appropriate domain event where the event represents a committed business fact;
5. persists the event through the Transactional Outbox when required by the Event contract;
6. commits the authoritative transaction.

Downstream consumers then determine what impact the business fact has.

---

# 5. Business Event Does Not Mean Feed Item

This is a foundational SocialConnect rule.

> A business event does not automatically mean that a Feed item must be created.

For example:

```text
Post.Created
```

may make a Post eligible for Feed processing.

But:

```text
Reaction.Added
```

does not create another Post or another Feed story.

Instead:

```text
Reaction.Added
        ↓
Engagement Signal
        ↓
Existing Feed Candidate / Projection
```

Likewise:

```text
Comment.Created
```

may affect:

* engagement;
* ranking;
* notification;
* activity;
* moderation;
* analytics;

without creating another Feed story.

The Event Impact Policy determines the downstream effect.

---

# 6. Canonical Event-Driven Social Architecture

The canonical SocialConnect processing model is:

```text
Business Operation
        ↓
Application / Domain Service
        ↓
Authoritative State Change
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Transaction Commit
        ↓
Outbox Processing / Dispatcher
        ↓
Event Handler
        ↓
Impact Policy
        ├── Feed
        ├── Notification
        ├── Alerting
        ├── Audit
        ├── Projection
        └── Other Consumer
```

The Event infrastructure remains a platform capability.

Social Media does not create a parallel event bus.

Social Media does not create a parallel Outbox.

Social Media does not create a parallel dispatcher.

Social Media does not create a parallel notification engine.

Social Media does not create a parallel alert engine.

---

# 7. Transactional Event Requirement

When a domain event represents a committed business fact, its required Outbox record must be persisted within the same transaction as the authoritative business state.

Therefore:

```text
Business State Change
+
Required Outbox Event
=
One Transactional Boundary
```

The system must not reach a committed state such as:

```text
Post exists
but
Post.Created event was lost
```

when that event is part of the canonical event contract.

The same principle applies to reactions, comments, shares, and other event-producing social operations.

---

# 8. Separation of Post Creation and Feed Generation

Post creation and Feed generation are explicitly separate responsibilities.

## Post Creation owns

* authorization;
* validation;
* content;
* Post state;
* visibility;
* location;
* media references;
* Post persistence;
* required Post domain event.

## Post Creation does not own

* calculating every Feed recipient;
* Feed candidate distribution;
* Feed ranking;
* Feed pagination;
* Feed rendering;
* Notification delivery;
* Alert delivery.

Therefore:

```text
Post Creation
      ↓
Post State
      ↓
Social.Post.Created
      ↓
Transactional Outbox
      ↓
Commit
```

is the authoritative Post business transaction.

Feed processing occurs downstream.

---

# 9. Asynchronous Feed Distribution

Feed propagation may execute asynchronously.

The canonical conceptual workflow is:

```text
User Creates Post
        ↓
Post Creation Transaction
        ↓
Post Persisted
        ↓
Social.Post.Created
        ↓
Outbox Persisted
        ↓
Transaction Committed
        ↓
Background Event Processing
        ↓
Feed Impact Handler
        ↓
Audience / Candidate Resolution
        ↓
Eligibility
        ↓
Distribution
        ↓
Feed Projection
```

Post creation must not synchronously fan out to every eligible Feed recipient.

This allows Feed processing to scale independently of the request that created the Post.

---

# 10. Durable Background Processing

The existing Transactional Outbox is the canonical durable hand-off between the authoritative business transaction and asynchronous processing.

Conceptually:

```text
                 ┌────────────────────┐
                 │   Post Operation   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Domain State +     │
                 │ Transactional      │
                 │ Outbox             │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Event Processing   │
                 │ / Dispatcher       │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Feed Handler       │
                 └─────────┬──────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           User A       User B       User C
           Feed         Feed         Feed
```

An in-memory queue must not become the authoritative bridge between a committed social operation and its downstream consequences.

---

# 11. Feed Is a Derived Distribution / Projection System

The Post remains authoritative social content.

Feed is a derived representation of content distributed to a viewer.

Therefore:

```text
Post
```

is authoritative.

The Feed may contain:

```text
PostId
ViewerId
Distribution / Candidate Information
Ranking / Ordering Information
Projection Metadata
```

or another approved derived representation.

The exact persistence model is an implementation concern governed by the Feed architecture.

The fundamental requirement is:

> Feed must never become the authoritative owner of Post content.

---

# 12. Feed Does Not Own Reactions

Feed has no independent reaction system.

A reaction displayed in a Feed card is a reaction to the underlying target.

Correct model:

```text
Feed
 └── Post
      └── Reaction
```

Incorrect model:

```text
Feed
 ├── FeedReaction
 └── PostReaction
```

When the user reacts from Feed:

```text
Feed UI
    ↓
Underlying Target
    ↓
Reaction Gateway
    ↓
Reaction Service
    ↓
Reaction State
```

The same Reaction subsystem may therefore be used from:

* Feed;
* Post detail;
* Comment;
* other explicitly supported reactable targets.

---

# 13. Reaction Requirements

The Reaction subsystem must expose one canonical mutation gateway.

The established endpoint is:

```text
POST /api/reaction/toggle
```

The consumer supplies the target and requested reaction context.

The server determines whether the resulting business transition is:

```text
Add
Change
Remove
```

Consumers must not need separate mutation endpoints for each transition.

---

# 14. Reaction Transition Rules

The canonical state transition is:

```text
No Existing Reaction
        +
Requested Reaction
        ↓
Reaction.Added
```

```text
Existing Reaction A
        +
Requested Reaction B
        ↓
Reaction.Changed
```

```text
Existing Reaction A
        +
Requested Reaction A
        ↓
Reaction.Removed
```

The server is authoritative for this decision.

The client must not determine the authoritative transition.

---

# 15. Reaction Targeting

Reaction targeting uses the canonical:

```text
TargetType + TargetId
```

mechanism.

No consumer-specific polymorphic target mechanism may be introduced.

The Reaction target catalog determines which domain objects may be reactable.

Known Social Media targets include:

* Post;
* Comment;
* explicitly supported Media targets;
* other explicitly registered reactable objects.

Feed itself is not a Reaction target.

---

# 16. Reaction Aggregate

Reactable entities may expose the established aggregate:

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

The aggregate belongs to the authoritative reactable target.

Feed only presents the resulting information.

Feed must not maintain a competing reaction aggregate.

---

# 17. Reaction Business Events

The canonical Social Reaction events are:

```text
Social.Reaction.Added
Social.Reaction.Changed
Social.Reaction.Removed
```

Each event represents a committed business transition.

Example:

```text
User selects Love
        ↓
Reaction Operation
        ↓
Existing Reaction = Like
        ↓
Reaction.Changed
        ↓
Transactional Outbox
        ↓
Dispatcher
        ↓
Consumers
```

Possible downstream consumers include:

* Feed engagement/ranking signals;
* Notification;
* Alerting;
* Audit;
* analytics;
* future recommendation systems.

Reaction events do not directly manipulate Feed UI.

---

# 18. Comment Requirements

Comments are first-class social objects.

A Comment may contain:

* author;
* target;
* content;
* optional media;
* parent comment;
* reply relationship;
* timestamps;
* audit information;
* soft-delete state;
* reaction support where enabled;
* visibility/moderation state;
* downstream notification and other event impacts.

Comment targets use:

```text
TargetType + TargetId
```

where the target is supported by the Comment contract.

---

# 19. Comment Tree

Comments form an authoritative parent-child structure.

Example:

```text
Post
│
├── Comment A
│   ├── Reply A1
│   └── Reply A2
│
├── Comment B
│   └── Reply B1
│
└── Comment C
```

The domain model must preserve the parent relationship.

The presentation layer must not invent parent-child relationships independently.

---

# 20. Comment Depth Policy

Maximum permitted comment/reply depth is a server-side policy.

The reusable CommentManager must not become the owner of the authoritative domain depth rule.

The effective policy may govern:

* maximum permitted depth;
* whether deeper replies are permitted;
* rendering depth;
* collapsed levels;
* lazy loading;
* pagination;
* moderation constraints.

The current CommentManager UI contract has a reply-depth constraint of two.

That UI constraint must remain distinguishable from the broader domain policy.

Therefore:

```text
Domain Policy
      ↓
Permitted Comment Depth
      ↓
Server ViewModel / Component Configuration
      ↓
CommentManager Rendering
```

---

# 21. Comment Tree Rendering

Comment retrieval and presentation must support:

* root comments;
* child replies;
* deterministic ordering;
* pagination;
* lazy expansion;
* collapsed branches;
* authorization;
* deleted comments;
* moderation state;
* configured depth;
* appropriate loading boundaries.

The server provides the authoritative ViewModel structure.

JavaScript may control:

* expand;
* collapse;
* load more;
* reply;
* edit;
* delete.

JavaScript does not own:

* authorization;
* depth policy;
* moderation rules;
* target ownership;
* deletion authority;
* persistence rules.

---

# 22. Comment Media

A Comment supports at most one media attachment.

Therefore:

```text
Comment
 └── 0..1 Media Assignment
```

Permitted content may be:

```text
Text only
```

or:

```text
Text + One Media
```

and, if explicitly permitted by the Comment content policy:

```text
Media only
```

The server owns the final content-validation rule.

---

# 23. Comment Media Uses the Canonical Media Platform

Comment creation must never implement a separate upload pipeline.

The Comment UI uses:

```text
MediaUploader
```

through its public contract.

CommentManager must not know:

* temporary storage implementation;
* physical storage path;
* file-processing internals;
* finalization internals;
* assignment internals.

---

# 24. Media as a Cross-Cutting Platform Capability

Media is not a Post-specific capability.

The Media subsystem must be reusable by:

* Posts;
* Comments;
* User Profiles;
* Shops;
* Products;
* other approved domain objects.

The Media platform therefore remains independent of the Social Media consumers that use it.

---

# 25. MediaUploader as the Unique Upload Gateway

The canonical UI upload architecture is:

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

The MediaUploader is the unique reusable upload gateway for ordinary UI upload workflows.

Consumers must not create independent upload implementations.

---

# 26. MediaUploader Contract

The locked MediaUploader v1.0.0 contract includes:

### Configuration

```text
data-media-uploader-parent-dropzone
```

### Upload request

```text
Role
Files
```

### Public API

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

### Lifecycle states

```text
idle
uploading
completed
cancelled
failed
```

### Event family

```text
media.uploader.*
```

The uploader remains consumer-independent.

Post Composer and CommentManager configure and consume it; they do not alter its internal behavior.

---

# 27. Automatic Media Upload

Automatic upload follows:

```text
User Selects File
        ↓
MediaUploader
        ↓
Upload Endpoint
        ↓
MediaUploadService
        ↓
Temporary Storage
        ↓
Media Identifier
        ↓
UI Upload Result
```

At this stage the media is staged/temporary.

It is not yet automatically considered permanently assigned business media.

---

# 28. Manual Media Upload

The same MediaUploader must support controlled/manual workflows.

The architecture supports:

```text
Automatic Upload
```

and:

```text
Manual Upload
```

through the same Media gateway.

Consumers may control when the upload is initiated through configuration/public APIs.

They must not create a second upload infrastructure.

---

# 29. Media Finalization

Finalization is a distinct responsibility:

```text
Temporary Media
        ↓
MediaFinalizationService
        ↓
Final Media
```

Finalization means:

```text
Temporary → Final
```

It does not mean:

```text
Final → Assigned
```

and it does not assign the media to a Post, Comment, Product, Shop, or User.

---

# 30. Media Assignment

Assignment is separate:

```text
Finalized Media
        ↓
MediaAssignmentService
        ↓
Business Owner / Target
```

Therefore:

> Finalization never performs business ownership assignment.

And:

> Assignment never finalizes temporary media.

This separation is mandatory.

---

# 31. Media Lifecycle

The canonical lifecycle is:

```text
Selected
   ↓
Uploading
   ↓
Temporary Upload
   ↓
Media Domain Record
   ↓
Business Transaction
   ↓
Finalized
   ↓
Assigned
```

Failure and cancellation must not corrupt the authoritative owning business transaction.

---

# 32. Media Events and Event Catalog Governance

Media lifecycle transitions may be operationally important.

However, a Media lifecycle transition must not automatically become a Social Media domain event.

For example:

```text
Media Uploaded
```

does not inherently mean:

```text
Social Feed Story Created
```

The Social Media business transaction determines whether the media participates in a Post or Comment.

Any Media event intended for the generic Event platform must be explicitly established in the canonical Event Catalog rather than being invented by a consumer.

---

# 33. Post Creation Workflow

The locked Post Creation workflow is:

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
Resolve Required Context
      ↓
Create Post
      ↓
Finalize Media
      ↓
Assign Media
      ↓
Complete Post Business State
      ↓
Raise Social.Post.Created
      ↓
Transactional Outbox
      ↓
Commit
```

PostService is not the workflow owner.

The Post Creation workflow owns orchestration.

---

# 34. Post Content

A Post may support:

* text;
* one or more supported media items;
* image;
* video;
* location;
* visibility;
* comments enabled/disabled;
* reactions enabled/disabled;
* sharing according to policy.

The Post domain owns these business properties.

---

# 35. Post Location

When location sharing is enabled, Post creation may use:

```text
CountryId
CityId
```

from the appropriate user/context information.

When location sharing is disabled:

```text
CountryId = NULL
CityId = NULL
```

Private profile location must never be exposed implicitly merely because the Post is being created.

The exact location privacy policy is server-owned.

---

# 36. Post Lifecycle

The current canonical Event Catalog establishes:

```text
Social.Post.Created
Social.Post.Updated
Social.Post.Deleted
Social.Post.Shared
Social.Post.VisibilityChanged
```

These events must retain their canonical semantic meaning.

A new event must not be introduced merely to simplify a Feed implementation.

If future requirements require a distinct publication state with a separate business transition, the Event Catalog must be deliberately revised before implementation.

---

# 37. Post Created and Feed Eligibility

`Social.Post.Created` is a business fact.

It does not mean that every created Post must immediately become a Feed story.

The Feed impact policy evaluates:

```text
Post State
+
Visibility
+
Audience
+
Moderation State
+
Deletion State
+
Social Graph
+
Other Feed Policy
```

before distribution.

Conceptually:

```text
Social.Post.Created
        ↓
Feed Impact Policy
        ↓
Is Post eligible?
        ├── No → No Feed Distribution
        └── Yes
             ↓
        Candidate Generation
             ↓
        Distribution
```

This preserves the distinction between:

```text
Post Created
```

and:

```text
Feed Eligible
```

without requiring a separate `PostPublished` event.

---

# 38. Post Update

`Social.Post.Updated` does not automatically mean a new Feed story.

Depending on the changed state, it may cause:

* existing Feed projection refresh;
* ranking signal update;
* eligibility reevaluation;
* no Feed action.

The Feed consumer must interpret the event according to the approved impact policy.

---

# 39. Post Visibility Change

`Social.Post.VisibilityChanged` may change Feed eligibility.

For example:

```text
Public
   ↓
Friends Only
```

or:

```text
Friends Only
   ↓
Private
```

may require existing distributions to be reevaluated or suppressed.

The Feed system must never continue displaying content that the current authoritative visibility policy prohibits.

---

# 40. Post Deletion

`Social.Post.Deleted` is Feed-relevant.

Conceptually:

```text
Post Deleted
      ↓
Social.Post.Deleted
      ↓
Outbox
      ↓
Dispatcher
      ↓
Feed Consumer
      ↓
Suppress / Remove / Invalidate Distribution
```

The Feed must not continue presenting deleted Post content as active content.

The exact projection cleanup strategy is a Feed implementation concern.

---

# 41. Post Restoration

Restoration is governed by the authoritative Post/moderation/domain lifecycle.

A separate `PostRestored` event is **not currently part of the locked Social Event Catalog**.

If restoration later becomes an independently observable business transition requiring asynchronous consumers, the Event Catalog must explicitly introduce and document an appropriate event.

Until such a contract exists, implementation must not invent one.

---

# 42. Sharing

SocialConnect distinguishes:

```text
ShareType.None
ShareType.Internal
ShareType.External
```

Internal sharing is a SocialConnect business operation.

External sharing is an external-distribution operation.

---

# 43. Internal Share

An Internal Share creates a new Post referencing the original Post.

The canonical relationship is:

```text
New Post
    └── SharedPostId → Original Post
```

The new Post does not duplicate:

* original Post identity;
* original content as authoritative content;
* original media as independent media ownership.

The original Post remains authoritative.

The existing Share model governs the relationship.

---

# 44. Internal Share Event

The established Event Catalog contains:

```text
Social.Post.Shared
```

Therefore Internal Share processing must use the canonical event contract rather than inventing a separate event name such as:

```text
InternalPostShared
```

Conceptually:

```text
User Shares Post
       ↓
Share / Post Business Workflow
       ↓
New Share/Post State
       ↓
Social.Post.Shared
       ↓
Transactional Outbox
       ↓
Dispatcher
       ↓
Feed / Notification / Other Consumers
```

The exact payload must follow the canonical Event Contract.

---

# 45. Shared Post Feed Representation

The Feed must be able to represent:

```text
User A shared User B's Post
```

without duplicating the original Post as a second authoritative content object.

The Feed presentation may contain:

```text
Sharer
+
Share Context
+
Original Post Reference
+
Original Post Presentation
```

The original Post remains authoritative.

---

# 46. Shared Post Graph Protection

A shared Post may reference another shared Post.

Feed presentation must therefore protect against:

* cycles;
* unlimited nesting;
* recursive rendering;
* pathological graphs.

The existing Feed architecture requires:

```text
Maximum Share Depth
+
Cycle Protection
```

The `FeedTimelineContextBuilder` is responsible for preparing a safe presentation context.

---

# 47. External Sharing

External sharing is not the same as internal Feed distribution.

An external share may be recorded for:

* audit;
* analytics;
* business metrics;
* other explicitly required purposes.

It must not automatically create an internal Feed story.

If the user creates an internal SocialConnect Post as part of a sharing workflow, that internal Post follows the normal Post business/event/Feed lifecycle.

---

# 48. Feed Generation Principles

Feed generation is event-driven.

The Feed subsystem must not rely on continuously scanning all Post records and attempting to infer historical business activity as its primary processing mechanism.

The canonical model is:

```text
Business Event
      ↓
Feed Impact Policy
      ↓
Feed Action
```

This makes downstream social consequences traceable.

---

# 49. Feed Impact Categories

Feed impacts are divided conceptually into four categories.

## Category A — New Candidate Content

Business events that may introduce new Feed candidates:

```text
Social.Post.Created
Social.Post.Shared
```

Eligibility remains mandatory.

Neither event means unconditional distribution.

---

## Category B — Existing Content State

Events that may alter existing Feed representation or eligibility:

```text
Social.Post.Updated
Social.Post.VisibilityChanged
Social.Post.Deleted
```

---

## Category C — Engagement Signals

Events that may influence existing Feed candidates or ranking:

```text
Social.Reaction.Added
Social.Reaction.Changed
Social.Reaction.Removed
```

and:

```text
Social.Comment.Created
Social.Comment.Updated
Social.Comment.Deleted
```

where applicable.

These generally do not create another Feed story.

---

## Category D — Social Graph Changes

Social graph business events may influence Feed eligibility or ranking.

Examples include:

```text
Friendship Accepted
Friendship Removed
Follow Created
Follow Removed
Block Created
Block Removed
```

These events are governed by the Social Graph/Event Catalog contracts.

---

# 50. Feed Impact Matrix

The following is the canonical **conceptual** impact matrix.

| Business Event                  | New Feed Candidate | Existing Feed Update | Ranking / Engagement Signal | Eligibility Re-evaluation |
| ------------------------------- | -----------------: | -------------------: | --------------------------: | ------------------------: |
| `Social.Post.Created`           |   Policy-dependent |             Possible |                         Yes |                       Yes |
| `Social.Post.Updated`           |                 No |                  Yes |                    Possible |                  Possible |
| `Social.Post.VisibilityChanged` |                 No |                  Yes |                    Possible |                       Yes |
| `Social.Post.Deleted`           |                 No |       Yes / Suppress |                          No |                       Yes |
| `Social.Post.Shared`            |   Policy-dependent |             Possible |                         Yes |                       Yes |
| `Social.Reaction.Added`         |                 No |             Possible |                         Yes |          Policy-dependent |
| `Social.Reaction.Changed`       |                 No |             Possible |                         Yes |          Policy-dependent |
| `Social.Reaction.Removed`       |                 No |             Possible |                         Yes |          Policy-dependent |
| `Social.Comment.Created`        |                 No |             Possible |                         Yes |          Policy-dependent |
| `Social.Comment.Updated`        |                 No |             Possible |                    Possible |          Policy-dependent |
| `Social.Comment.Deleted`        |                 No |             Possible |                    Possible |          Policy-dependent |
| Social Graph relationship event |                 No |             Possible |                         Yes |                       Yes |
| Block relationship event        |                 No |       Yes / Suppress |                          No |                       Yes |

**Important:** This matrix describes architectural impact categories, not a promise that every event will execute every possible effect.

The final implementation policy for each event must explicitly determine the actual effects.

---

# 51. Feed Story Creation vs Signal Update

This distinction is mandatory.

Example:

```text
Social.Post.Created
        ↓
Eligible?
        ↓
Create Candidate
```

while:

```text
Social.Reaction.Added
        ↓
Engagement Signal
        ↓
Update Existing Candidate / Ranking Data
```

Similarly:

```text
Social.Comment.Created
        ↓
Engagement / Conversation Signal
```

does not mean:

```text
Create Another Feed Story
```

unless a future Feed policy explicitly defines such behavior.

---

# 52. Feed Candidate Inventory

Feed processing must conceptually maintain or calculate an eligible candidate inventory.

A candidate must satisfy relevant conditions such as:

* content exists;
* content is in a distributable state;
* content is not deleted;
* content is visible to the viewer;
* viewer is permitted to see it;
* blocking rules do not prohibit distribution;
* relationship rules permit distribution;
* moderation policy permits distribution;
* audience rules permit distribution;
* other Feed eligibility rules are satisfied.

The implementation may use stored candidate records, projections, queries, or a hybrid approach.

The architectural requirement is the separation of:

```text
Candidate Generation
```

from:

```text
Eligibility
```

and:

```text
Ranking
```

---

# 53. Feed Eligibility

Feed eligibility is a business/application concern.

The Feed presentation layer must not decide whether content is eligible.

Canonical flow:

```text
Candidate
   ↓
Eligibility
   ↓
Ranking / Ordering
   ↓
Timeline Context
   ↓
Feed Card
```

The server remains authoritative.

---

# 54. Feed Ranking

Feed ranking is separate from candidate generation.

The ranking boundary may use signals including:

* recency;
* social relationship;
* author relationship;
* previous interaction;
* reactions;
* comments;
* shares;
* content type;
* location relevance;
* user preferences;
* negative feedback;
* social graph changes;
* content diversity;
* other approved ranking signals.

The exact ranking algorithm is not fixed by this requirements document.

The architecture must instead preserve a dedicated ranking boundary so that ranking can evolve independently from:

* Post creation;
* Reaction persistence;
* Comment persistence;
* Feed rendering.

---

# 55. Feed Is Not Defined as Chronological Timeline

A chronological Feed mode may exist.

It must not become the architectural definition of Feed.

The Feed architecture remains capable of:

```text
Candidate Generation
        ↓
Eligibility
        ↓
Ranking
        ↓
Ordering
        ↓
Presentation
```

This allows future Feed surfaces such as:

* personalized Home;
* Latest;
* Friends;
* Following;
* nearby;
* Shop/social surfaces;
* recommendation surfaces.

---

# 56. Feed Timeline Architecture

The locked Feed architecture remains:

```text
FeedTimelineService
        ↓
FeedTimelineContextBuilder
        ↓
FeedReactionResolver
        ↓
FeedCardFactory
```

## FeedTimelineService

Owns Feed orchestration.

It coordinates the read workflow but does not become the owner of Post, Reaction, Comment, or Media business operations.

## FeedTimelineContextBuilder

Builds the context required for Feed rendering.

It may load:

* Posts;
* authors;
* profiles;
* media;
* share graph;
* reaction presentation state;
* comment summaries;
* social graph context;
* other approved presentation data.

It must protect against:

* N+1 loading;
* duplicate target resolution;
* recursive share graphs.

## FeedReactionResolver

Resolves reaction presentation from existing Reaction information.

It does not implement Reaction business rules.

## FeedCardFactory

Maps prepared context into Feed ViewModels.

It remains presentation mapping logic.

---

# 57. Feed Reaction Resolver Boundaries

Two resolver responsibilities remain distinct.

```text
IReactionTargetResolver
```

resolves reactable domain targets for the Reaction subsystem.

```text
IFeedReactionTargetResolver
```

maps Feed entity context into the canonical Reaction target model.

They must not be collapsed into one service merely because both deal with reaction targets.

---

# 58. FeedCardFactory Restrictions

FeedCardFactory must not:

* query repositories;
* perform authorization;
* resolve business ownership;
* execute Reaction business logic;
* create Notifications;
* create Alerts;
* create domain events;
* determine Feed eligibility;
* rank candidates;
* perform Feed distribution.

It receives prepared context and maps that context to presentation ViewModels.

---

# 59. Feed and Media

Feed does not upload media.

Feed does not finalize media.

Feed does not assign media.

Feed consumes media already processed by the canonical Media subsystem.

Therefore:

```text
Media
   ↓
Post / Comment
   ↓
Feed Presentation
```

not:

```text
Feed
   ↓
Media Upload
```

---

# 60. Feed and Comments

Feed may display:

* comment count;
* selected comments;
* comment composer;
* replies;
* comment interaction.

The reusable CommentManager remains the owner of comment interaction.

Feed configures/embeds the CommentManager rather than implementing another comment system.

---

# 61. Feed and Reactions

Feed may display:

```text
Like
Love
Haha
Wow
Sad
Angry
```

but Feed does not implement Reaction persistence or transition logic.

The action is sent through the canonical Reaction gateway using:

```text
TargetType + TargetId
```

---

# 62. Feed and Sharing

Feed may expose the Share interaction.

The actual business operation belongs to the established Share/Post workflow.

Feed initiates the operation against the underlying Post.

It does not create Share state itself.

---

# 63. Social Graph Integration

Feed may consume Social Graph relationships including:

* friends;
* followers;
* following;
* followed shops/pages where supported;
* blocks;
* relationship changes;
* other approved relationship signals.

The Social Graph remains authoritative for those relationships.

Feed consumes relationship information.

It does not become the owner of Social Graph state.

---

# 64. Social Graph Events and Feed

When a Social Graph operation changes the set of users who may see content, Feed eligibility may require recalculation.

Conceptually:

```text
Social Graph Business Operation
        ↓
Social Graph Event
        ↓
Feed Impact
        ↓
Eligibility Re-evaluation
```

Examples include:

```text
Follow Created
Follow Removed
Friendship Accepted
Friendship Removed
Block Created
Block Removed
```

The exact events remain governed by the Social Graph/Event Catalog.

---

# 65. Notification Integration

Social Media events may be consumed by the Notification subsystem.

Examples include:

```text
Social.Post.Created
Social.Post.Shared
Social.Reaction.Added
Social.Reaction.Changed
Social.Comment.Created
```

where the Notification Policy defines a notification-worthy condition.

The Social Media service does not directly own notification persistence or delivery.

Canonical flow:

```text
Social Event
      ↓
Dispatcher
      ↓
Notification Consumer
      ↓
Recipient Resolution
      ↓
Notification Orchestration
      ↓
Notification Persistence
      ↓
Delivery Planning
      ↓
Channel Adapter
```

Recipient resolution remains part of the canonical Notification architecture.

---

# 66. Alerting Integration

Alerting is a separate operational platform.

A Social Media business event may provide evidence to an Alert Source/Policy.

For example:

```text
Social Event
      ↓
Operational Evidence
      ↓
Alert Source / Policy
      ↓
Alert
```

The generic Alerting platform must not contain a giant Social Media switch such as:

```text
if PostCreated...
if CommentCreated...
if ReactionAdded...
```

Instead:

```text
Generic Alert Platform
        +
Domain-specific Alert Sources / Policies
```

is the canonical architecture.

Social Media contributes operational conditions through the established Alert source/policy boundary.

---

# 67. Event / Notification / Alert Separation

These concepts remain distinct:

```text
Business Event
    =
Committed Business Fact
```

```text
Notification
    =
User-facing downstream communication
```

```text
Alert
    =
Operational condition requiring attention
```

```text
Feed
    =
Content distribution / presentation projection
```

Therefore:

```text
Business Event
       ↓
Dispatcher
       │
       ├── Feed Consumer
       ├── Notification Consumer
       ├── Alert Consumer
       ├── Audit Consumer
       └── Other Consumer
```

No consumer becomes the owner of the event.

---

# 68. Domain Event vs Technical Event

A domain event represents a meaningful business fact.

Examples:

```text
Social.Post.Created
Social.Post.Updated
Social.Post.Deleted
Social.Post.Shared
Social.Post.VisibilityChanged

Social.Reaction.Added
Social.Reaction.Changed
Social.Reaction.Removed

Social.Comment.Created
Social.Comment.Updated
Social.Comment.Deleted
```

Technical actions are not automatically domain events.

Examples:

```text
HTTP request received
JavaScript click occurred
Feed card rendered
Database query executed
Media upload progress changed
```

These do not automatically become domain events.

---

# 69. Canonical Social Event Catalog Alignment

The Social Media subsystem must align with the established Event Catalog.

The currently locked Social event surface is:

## Post

```text
Social.Post.Created
Social.Post.Updated
Social.Post.Deleted
Social.Post.Shared
Social.Post.VisibilityChanged
```

## Reaction

```text
Social.Reaction.Added
Social.Reaction.Changed
Social.Reaction.Removed
```

## Comment

```text
Social.Comment.Created
Social.Comment.Updated
Social.Comment.Deleted
Social.Comment.Moderated
```

The Event Catalog is authoritative.

This document does not silently create competing event names.

If a future business requirement requires a new event, that event must be explicitly added to the canonical Event Catalog with:

* semantic definition;
* aggregate;
* trigger;
* payload;
* version;
* transaction requirements;
* consumers;
* compatibility rules.

---

# 70. Comment Reply Event Semantics

A reply is a Comment business operation.

The current Event Catalog does not define a separate:

```text
Social.Comment.Replied
```

event.

Therefore a reply is represented through:

```text
Social.Comment.Created
```

with the relevant parent-comment information in the event contract.

Conceptually:

```text
Create Reply
      ↓
Comment.Created
      ↓
ParentCommentId / Target Context
      ↓
Notification / Feed / Other Impact
```

If future requirements justify a separate reply event, the Event Catalog must be deliberately revised.

---

# 71. Event Payload Requirements

Each Social event must provide enough information for its approved consumers to process the business fact without depending on UI state.

A canonical event contract may include:

* EventId;
* event name/type;
* aggregate/entity identifier;
* actor/user identifier where applicable;
* TargetType where applicable;
* TargetId where applicable;
* operation timestamp;
* relevant business state transition;
* correlation information;
* event version;
* metadata required by the Event infrastructure.

The exact payload is defined by the Event Catalog and event specification.

Events must remain vendor-neutral.

Events must not contain unnecessary external-provider fields such as:

```text
TwilioMessageId
SendGridTemplateId
FirebaseVendorPayload
PaymentProviderInternalObject
```

unless such information is itself a genuine business fact explicitly required by the canonical contract.

---

# 72. Event Idempotency

Every asynchronous Social consumer must be idempotent.

Duplicate event delivery may occur because of:

* retry;
* worker restart;
* process crash;
* lease expiration;
* database interruption;
* dispatcher failure;
* transient infrastructure failure.

Therefore:

```text
Same EventId
+
Same Consumer
=
No Duplicate Business Effect
```

This applies particularly to Feed distribution.

---

# 73. Feed Distribution Idempotency

Repeated processing of:

```text
Social.Post.Created
```

must not create duplicate logical Feed distributions.

The Feed processing layer must have a deterministic idempotency strategy.

Conceptually:

```text
Event
+
Consumer
+
Viewer
+
Logical Feed Distribution
```

must resolve to one logical result.

The exact persistence/key strategy belongs to Feed implementation.

---

# 74. Feed Rebuild Capability

Because Feed is derived from authoritative domain state and events, the architecture should remain capable of rebuilding Feed projections where required.

Feed must not become the only place from which social content can be reconstructed.

Rebuild capability is important for:

* projection recovery;
* data recovery;
* new Feed surfaces;
* ranking changes;
* schema/projection evolution;
* future recommendation systems.

---

# 75. Background Worker Requirements

Feed processing may use background workers.

Workers must participate in the common application-wide Worker Health architecture.

A Feed worker must support, through the established Worker Health contract:

* worker identity;
* heartbeat;
* progress evidence;
* current operation;
* last successful cycle;
* failure reporting;
* operational logging;
* concurrency safety;
* controlled throughput;
* recovery behavior.

Feed must not create an isolated Feed-only health system.

The same Worker Health platform must later support workers from:

* Event;
* Notification;
* Feed;
* Media;
* Marketplace;
* Orders;
* Payments;
* Administration;
* other future modules.

---

# 76. Worker Health vs Alerting

Worker Health and Alerting remain separate.

Worker Health produces operational evidence.

Alerting evaluates operational conditions.

Therefore:

```text
Feed Worker
    ↓
Worker Health
    ↓
Operational Evidence
    ↓
Alert Source / Policy
    ↓
Alert
```

Alerting does not own:

* worker execution;
* worker scheduling;
* worker recovery;
* worker restart;
* worker processing logic.

Worker self-recovery remains the responsibility of the appropriate worker/application infrastructure.

---

# 77. Retry Ownership

Social Media must not introduce one universal retry service.

Retry ownership remains separated by failure domain:

```text
Business Workflow Retry
Event / Outbox Retry
Notification Delivery Retry
External Sink Retry
Worker Recovery
Alert Recovery
```

For example:

```text
Feed Handler Failure
        ↓
Event Processing Retry
```

is not the same mechanism as:

```text
Email Provider Failure
        ↓
Notification Delivery Retry
```

and neither is the same as:

```text
Feed Worker Crash
        ↓
Worker Recovery
```

This separation is mandatory.

---

# 78. External Dependency Boundary

All external dependencies used by Social Media or its downstream platforms must be isolated behind internal contracts.

Canonical structure:

```text
Business / Application
        ↓
Internal Contract
        ↓
Infrastructure Adapter
        ↓
External Provider
```

External vendor SDKs and provider-specific types must not leak into domain/application logic.

This applies to:

* object/file storage;
* email;
* SMS;
* push;
* maps/geolocation;
* authentication providers;
* payment providers;
* analytics;
* search;
* AI;
* external operational sinks;
* other external APIs.

Changing a provider should ideally require changing or configuring an adapter rather than rewriting business logic.

---

# 79. Social Media and External Providers

Social domain events must remain vendor-neutral.

For example:

```text
Reaction.Added
```

must describe the business fact:

```text
User X added Reaction Y to Target Z
```

not:

```text
FirebaseReactionPayload...
```

Likewise, Feed business logic must not depend directly on an external vendor SDK.

External provider integration belongs behind the appropriate internal contract and infrastructure adapter.

---

# 80. Feed Consistency Model

SocialConnect distinguishes authoritative transactional consistency from downstream eventual consistency.

## Strong consistency is required for

* Post state;
* Reaction state;
* Comment state;
* Share transaction;
* Media assignment;
* required domain event persistence;
* required Outbox persistence.

## Eventual consistency is acceptable for

* Feed propagation;
* Feed projection updates;
* Feed ranking updates;
* Notification delivery;
* Alert evaluation;
* derived projections;
* other explicitly asynchronous downstream effects.

This distinction is fundamental to scalable social processing.

---

# 81. User Experience Requirement

A Post is considered successfully created when the authoritative Post transaction succeeds.

The user must not receive:

```text
Post Creation Failed
```

merely because Feed propagation is still processing.

The system may therefore legitimately have:

```text
Post Created
+
Feed Processing Pending
```

for a short period.

Feed workers must not create authoritative Posts merely because a Feed representation is missing.

---

# 82. Content Lifecycle

The canonical social lifecycle is:

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
Complete Business State
   ↓
Domain Event
   ↓
Transactional Outbox
   ↓
Background Processing
   ↓
Feed Candidate
   ↓
Eligibility
   ↓
Ranking
   ↓
Feed Distribution
   ↓
Feed Presentation
   ↓
User Interaction
   ├── Reaction
   ├── Comment
   └── Share
         ↓
     New Business Event
         ↓
     Further Impact
```

This forms the SocialConnect event-driven social loop.

---

# 83. End-to-End Post Scenario

The canonical Post vertical slice is:

```text
User Submits Post
        ↓
Post Composer
        ↓
CreatePostDto
        ↓
PostCreationContext
        ↓
PostCreationService
        ↓
Validation
        ↓
Create Post
        ↓
Finalize Media
        ↓
Assign Media
        ↓
Complete Authoritative Post State
        ↓
Social.Post.Created
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Dispatcher
        ↓
Post Created Consumer / Feed Impact
        ↓
Resolve Audience / Candidates
        ↓
Evaluate Eligibility
        ↓
Feed Distribution
        ↓
Feed Projection
        ↓
Feed Available
```

Separately:

```text
Social.Post.Created
        ├── Notification Policy
        ├── Alert Source / Policy
        ├── Audit
        └── Other Approved Consumers
```

---

# 84. End-to-End Reaction Scenario

```text
User Clicks Love
        ↓
Feed / Post / Comment UI
        ↓
POST /api/reaction/toggle
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
        ├── Feed Signal Consumer
        ├── Notification Consumer
        ├── Alert Consumer
        ├── Audit Consumer
        └── Other Consumers
```

---

# 85. End-to-End Comment Scenario

```text
User Submits Comment
        ↓
CommentManager
        ↓
Comment API
        ↓
Comment Application Service
        ↓
Validate Target
        ↓
Validate Parent / Depth
        ↓
Validate Content
        ↓
Validate Media Count ≤ 1
        ↓
Create Comment
        ↓
Assign Finalized Media
        ↓
Persist
        ↓
Social.Comment.Created
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Dispatcher
        ├── Feed Signal Consumer
        ├── Notification Consumer
        ├── Alert Consumer
        ├── Audit Consumer
        └── Other Consumers
```

A reply follows the same business event contract, with parent-comment information identifying the reply relationship.

---

# 86. End-to-End Share Scenario

```text
User Selects Share
        ↓
Share / Post Workflow
        ↓
Validate Original Post
        ↓
Create Internal Share Post
        ↓
Set SharedPostId
        ↓
Do Not Duplicate Original Content / Media
        ↓
Complete Share/Post State
        ↓
Social.Post.Shared
        ↓
Transactional Outbox
        ↓
Commit
        ↓
Dispatcher
        ↓
Feed Impact
        ↓
Audience / Eligibility
        ↓
Feed Distribution
```

---

# 87. End-to-End Comment Media Scenario

```text
User Selects One Media Item
        ↓
CommentManager
        ↓
MediaUploader
        ↓
/api/media/upload-async
        ↓
MediaUploadService
        ↓
Temporary Storage
        ↓
Media Identifier
        ↓
Comment Creation Workflow
        ↓
Comment Persisted
        ↓
Media Finalization
        ↓
Media Assignment
        ↓
Social.Comment.Created
```

CommentManager never implements:

```text
File Storage
Image Processing
Media Finalization
Media Assignment
```

---

# 88. Social Media Component Boundaries

## Post Composer

Owns:

* composition UI;
* form state;
* interaction;
* MediaUploader integration;
* submit/cancel interaction;
* rendering.

Does not own:

* Post business rules;
* Feed generation;
* Notification;
* Alerting;
* media storage;
* Post persistence.

---

## MediaUploader

Owns:

* upload interaction;
* file queue;
* upload state;
* temporary-upload communication;
* lifecycle;
* public upload events;
* public API.

Does not own:

* Post;
* Comment;
* Feed;
* final business transaction;
* media assignment business rules.

---

## Reaction Component

Owns:

* reaction presentation;
* interaction;
* selected-state presentation;
* calling the canonical Reaction endpoint.

Does not own:

* reaction persistence;
* target authorization;
* Add/Change/Remove business rules;
* Reaction aggregate persistence.

---

## CommentManager

Owns:

* comment UI;
* tree interaction;
* create/reply/edit interaction;
* lazy expansion;
* comment API interaction;
* component state.

Does not own:

* comment business rules;
* media storage;
* notification;
* Feed generation;
* authoritative comment-depth policy.

---

## Feed Component

Owns:

* Feed presentation;
* Feed card rendering;
* interaction wiring;
* reusable component integration.

Does not own:

* Feed generation;
* Feed ranking;
* Feed distribution;
* Post creation;
* Reaction persistence;
* Comment persistence;
* Share persistence.

---

# 89. Reusable UI Architecture

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
Razor View
      ↓
Component JavaScript
      ↓
Component CSS
```

The Builder configures the ViewModel.

The ViewComponent renders.

Defaults centralize defaults.

Builders remain transient.

Consumers interact through public component contracts.

---

# 90. JavaScript Requirements

Social Media JavaScript must:

* use `App.Modules.register`;
* use WeakMap where instance storage is required;
* use immutable/frozen configuration and constants where appropriate;
* expose public APIs only;
* implement lifecycle;
* support `init`;
* support `bind`;
* support `destroy`;
* use `App.Events`;
* avoid duplicating server business rules.

`site.js` remains foundation-only.

No Social Media API or business logic belongs in `site.js`.

---

# 91. Server Ownership

The server is authoritative for:

* authentication;
* authorization;
* visibility;
* privacy;
* target validation;
* ownership;
* relationship rules;
* blocking;
* moderation;
* media ownership;
* comment depth policy;
* reaction transitions;
* sharing rules;
* Feed eligibility;
* Feed distribution policy;
* domain events;
* audit requirements.

Client JavaScript must never be trusted for these decisions.

---

# 92. Security Requirements

Every social mutation must validate:

* authenticated user;
* target existence;
* target type;
* target visibility;
* target ownership/permission;
* blocked relationships;
* deleted state;
* moderation state;
* feature/policy availability;
* anti-forgery requirements where applicable;
* request validation.

A user must never gain unauthorized access merely by changing:

```text
TargetId
```

or:

```text
TargetType
```

in a request.

Polymorphic targeting is a routing mechanism, not an authorization mechanism.

---

# 93. Soft Delete

Social entities participating in the established soft-delete architecture must use the canonical soft-delete lifecycle.

When deletion represents an event-relevant business fact, the appropriate event must be emitted.

For example:

```text
Post Deleted
    ↓
Social.Post.Deleted
    ↓
Feed Impact
```

Feed must not independently decide that an authoritative deleted Post remains active.

---

# 94. Audit

Important Social Media mutations participate in the established audit architecture.

Examples include:

* Post deletion;
* Post visibility changes;
* Comment deletion;
* moderation-related changes;
* administrative intervention;
* account-level social restrictions;
* other privileged or security-sensitive social mutations.

Audit remains a cross-cutting capability.

Individual Social services must not create competing audit infrastructures.

---

# 95. Administration Integration

Administration governs policy.

Administration does not directly manipulate Social domain tables.

Correct:

```text
Admin
   ↓
Comment Policy
   ↓
Configured Maximum Reply Depth
   ↓
Comment Application Service
```

Incorrect:

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

# 96. Moderation Integration

Social Media integrates with the established Administration moderation architecture.

Potential moderation actions include:

* hide;
* remove;
* restrict;
* suspend;
* escalate.

Moderation is not implemented as an ad-hoc Feed rule.

The authoritative Post/Comment/moderation state determines whether content remains eligible for presentation.

Feed responds to that authoritative state and its corresponding events.

---

# 97. Social Event Specification Requirement

Before implementation of any Feed-relevant Social event, the event must have an explicit specification containing:

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
Worker / Operational Considerations
Tests
```

No event should be implemented merely because a UI button exists.

---

# 98. Complete Feed Event Lifecycle Requirement

Every Feed-relevant event must be documented and tested through:

```text
1. Business Operation
2. Authoritative State Change
3. Domain Event
4. Transactional Outbox Persistence
5. Transaction Commit
6. Dispatcher
7. Event Handler
8. Audience / Candidate Resolution
9. Feed Eligibility
10. Feed Distribution / Projection
11. Ranking / Signal Update
12. Idempotency
13. Failure / Retry
14. Notification Impact
15. Alert Impact
16. Audit Impact where applicable
17. Worker Health Impact where applicable
18. Tests
```

Not every event executes every stage with a business effect.

The specification must explicitly state which stages apply.

---

# 99. Read / Write Separation

Write operations include:

```text
Post Creation
Post Update
Post Visibility Change
Comment Creation
Comment Update
Reaction Mutation
Internal Share
Media Assignment
```

These are handled by appropriate application/domain services.

Read operations include:

```text
Feed Timeline
Post Detail
Comment Tree
Reaction Summary
Profile Timeline
Shared Post Context
```

These use appropriate query/read services.

Controllers remain thin.

---

# 100. Repository and Loader Boundary

Business/application services must follow the established SocialConnect loader and service architecture.

Where a loader contract applies:

```text
Application Service
        ↓
Loader / Lookup
        ↓
Repository / Persistence
```

must be preferred over direct repository orchestration inside business services.

The established abstractions include:

```text
IEntityLoader<TEntity,TKey>
ITrackedEntityLoader<TEntity,TKey>
```

Repositories remain persistence-oriented.

Business services do not expose or depend on arbitrary `IQueryable` infrastructure merely to move business logic into application code.

---

# 101. No God Service

The Social Media subsystem must not create a service such as:

```text
SocialMediaService
```

responsible for:

* Posts;
* Comments;
* Reactions;
* Media;
* Sharing;
* Feed;
* Notifications;
* Alerts;
* Social Graph.

Responsibilities remain separated.

---

# 102. Canonical Responsibility Map

| Capability                    | Owner                              |
| ----------------------------- | ---------------------------------- |
| Post creation                 | Post Creation application workflow |
| Post business state           | Post domain                        |
| Post lifecycle events         | Post domain/application boundary   |
| Media upload                  | MediaUploader / MediaUploadService |
| Media finalization            | MediaFinalizationService           |
| Media assignment              | MediaAssignmentService             |
| Reaction mutation             | Reaction service                   |
| Reaction target resolution    | Reaction target resolver           |
| Comment mutation              | Comment service                    |
| Comment tree policy           | Comment/Admin policy               |
| Comment rendering             | CommentManager                     |
| Sharing                       | Share/Post workflow                |
| Feed orchestration            | FeedTimelineService                |
| Feed context                  | FeedTimelineContextBuilder         |
| Feed reaction mapping         | FeedReactionResolver               |
| Feed card mapping             | FeedCardFactory                    |
| Feed distribution             | Feed processing / event handler    |
| Feed eligibility              | Feed policy/application boundary   |
| Feed ranking                  | Feed ranking boundary              |
| Notification                  | Notification subsystem             |
| Alerting                      | Alerting subsystem                 |
| Worker health                 | Application-wide Worker Health     |
| Audit                         | Canonical audit infrastructure     |
| External provider integration | Infrastructure adapters            |

---

# 103. Performance Requirements

The Social Media architecture must avoid:

* synchronous fan-out to thousands of users during Post creation;
* N+1 queries in Feed rendering;
* repeated author/profile queries;
* duplicate media loading;
* duplicate Reaction target resolution;
* unnecessary recursive shared-Post loading;
* repeated Social Graph queries for the same Feed context.

Feed context preparation must be efficient.

FeedCardFactory must remain pure mapping.

Feed distribution must support controlled throughput and asynchronous processing.

---

# 104. Feed Distribution Failure Model

A Feed distribution failure must not roll back an already committed Post merely because Feed processing failed afterward.

Correct:

```text
Post Transaction
       ↓
Commit
       ↓
Post Exists
       ↓
Feed Processing Fails
       ↓
Event / Handler Retry
       ↓
Feed Eventually Updated
```

The authoritative business transaction and asynchronous Feed processing therefore have different failure domains.

---

# 105. Recovery and Reprocessing

Because Social event processing is durable and idempotent, Feed processing must support recovery after:

* worker crash;
* application restart;
* database interruption;
* dispatcher interruption;
* event-handler failure;
* lease expiration;
* transient infrastructure failure.

Recovery must rely on the established Event/Outbox/Worker Health architecture.

Social Media must not create a separate recovery framework.

---

# 106. Notification and Feed Independence

Notification and Feed are independent consumers.

For example:

```text
Social.Post.Created
        ├── Feed Consumer
        └── Notification Consumer
```

A Notification failure must not corrupt the Feed business model.

A Feed failure must not corrupt Notification persistence.

The same business event may therefore produce different downstream outcomes without changing the authoritative Post state.

---

# 107. Alerting and Feed Independence

Alerting observes operational evidence.

A Feed degradation condition may become an Alert, but Alerting does not become the Feed execution engine.

Correct:

```text
Feed Worker
    ↓
Worker Health
    ↓
Operational Evidence
    ↓
Alert Policy
    ↓
Alert
```

Incorrect:

```text
Alert Service
    ↓
Run Feed Worker
```

---

# 108. External Operational Sinks

If Alerting eventually delivers to external operational systems, Social Media remains isolated from the provider.

The boundary remains:

```text
Alerting
   ↓
Internal External-Sink Contract
   ↓
Infrastructure Adapter
   ↓
External Operational Provider
```

Social Media must not contain vendor-specific operational sink logic.

---

# 109. Facebook-Class Product Benchmark

SocialConnect uses the established social-network product model as a functional benchmark.

The intended Feed capability includes:

* personalized content;
* relationship-aware content;
* recency;
* engagement signals;
* candidate selection;
* ranking;
* content diversity;
* user controls;
* continuous refinement.

The requirement is not to reproduce another platform's proprietary implementation.

The architectural objective is:

```text
Candidate Generation
→ Eligibility
→ Ranking
→ Distribution
→ Presentation
```

rather than a Feed implemented merely as:

```text
SELECT TOP 50 Posts
ORDER BY CreatedAt DESC
```

---

# 110. Social Media Domain Boundaries

The following ownership boundaries are mandatory:

```text
Post
    owns Post business truth

Comment
    owns Comment business truth

Reaction
    owns Reaction business truth

Media
    owns Media lifecycle

Sharing
    owns Share business operation

Social Graph
    owns relationship truth

Feed
    owns derived distribution/presentation state

Notification
    owns notification truth

Alerting
    owns operational alert truth

Event Platform
    owns durable event transport

Worker Health
    owns worker operational evidence
```

No subsystem may silently take ownership of another subsystem's authoritative state.

---

# 111. Acceptance Criteria

The Social Media subsystem is architecturally complete only when the following requirements are demonstrable.

## Posts

* Posts can be created through the canonical Post Creation workflow.
* Posts can contain supported content and media.
* Visibility is enforced server-side.
* Location is handled according to policy.
* `Social.Post.Created` is emitted according to the canonical Event Catalog.
* Post updates produce the appropriate event where required.
* Post visibility changes affect downstream eligibility where applicable.
* Post deletion affects downstream Feed state.

## Media

* MediaUploader is the single reusable upload gateway.
* Automatic upload is supported.
* Manual upload is supported.
* Temporary storage is distinct from final storage.
* Finalization is separate from assignment.
* Consumers do not implement independent upload pipelines.
* Comments support at most one media attachment.
* Media lifecycle remains independent from Feed business logic.

## Reactions

* One canonical mutation endpoint exists.
* Add/change/remove transitions are resolved server-side.
* `TargetType + TargetId` is used.
* Feed does not own Reaction persistence.
* Reaction aggregate belongs to the target.
* Reaction business events are emitted according to the Event Catalog.
* Reaction processing is idempotent.

## Comments

* Comment trees are supported.
* Parent-child relationships are authoritative.
* Maximum domain depth is policy-controlled.
* Rendering depth remains distinguishable from domain depth.
* Tree rendering supports expansion/collapse.
* CommentManager remains reusable.
* Comment media uses MediaUploader.
* One Comment has at most one media attachment.
* Comment business events follow the canonical Event Catalog.

## Sharing

* Internal shares create a new Post/reference relationship.
* Original content and media are not duplicated as authoritative business state.
* `SharedPostId` is used according to the locked Share contract.
* Shared graphs have maximum-depth and cycle protection.
* `Social.Post.Shared` is used according to the canonical Event Catalog.
* External sharing is distinct from internal Feed distribution.

## Feed

* Feed generation is separate from Post creation.
* Feed processing is event-driven.
* Feed processing may execute asynchronously.
* Feed is not the authoritative social content store.
* Feed does not own Reactions.
* Feed does not own Comments.
* Feed consumes Social events.
* Feed distinguishes candidate generation from ranking.
* Feed distinguishes ranking from rendering.
* Feed distribution is idempotent.
* Feed can be rebuilt/reprocessed from authoritative state/events where required.

## Events

* Social business operations produce the appropriate domain events.
* Events are persisted transactionally through the established Outbox.
* Dispatcher processing is durable.
* Consumers are idempotent.
* Feed, Notification, and Alerting remain independent consumers.
* Event contracts are explicitly documented.
* Each Feed-relevant event has an end-to-end specification.
* No consumer-specific command is disguised as a domain event.

## Worker Health

* Feed workers participate in the application-wide Worker Health platform.
* Worker heartbeat/progress/success/failure evidence is available through the common contract.
* Feed does not create a private health subsystem.
* Alerting can evaluate Feed operational evidence.

## External Dependencies

* External providers are isolated behind internal contracts.
* Vendor SDKs do not leak into Social domain/application logic.
* Provider-specific retry behavior remains in the appropriate failure domain.
* Vendor replacement does not require redesigning Social business logic.

---

# 112. Canonical Social Media Architecture

The complete architecture is:

```text
                         SOCIALCONNECT
                         SOCIAL MEDIA
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
       ▼                      ▼                      ▼
     POST                  COMMENT                REACTION
       │                      │                      │
       │                      │                      │
       └──────────────┬───────┴──────────────┬───────┘
                      │                      │
                      ▼                      ▼
                   MEDIA                 SHARING
                      │                      │
                      └──────────┬───────────┘
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
                 ┌───────────────┼────────────────┐
                 │               │                │
                 ▼               ▼                ▼
               FEED        NOTIFICATION       ALERTING
                 │
                 ▼
        CANDIDATE GENERATION
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
        ┌────────┼──────────┐
        ▼        ▼          ▼
     Reaction  Comment     Share
        │        │          │
        └────────┴──────────┘
                 │
                 ▼
          BUSINESS EVENTS
                 │
                 ▼
          FURTHER IMPACT
```

Operationally:

```text
Feed / Notification / Event Workers
                ↓
          Worker Health
                ↓
       Operational Evidence
                ↓
       Alert Source / Policy
                ↓
             Alert
```

And for external providers:

```text
Business / Application
        ↓
Internal Contract
        ↓
Infrastructure Adapter
        ↓
External Provider
```

---

# 113. Final Architectural Contract

The SocialConnect Social Media subsystem is governed by these principles:

1. **Posts are authoritative social content.**
2. **Media is a generic cross-cutting platform capability.**
3. **MediaUploader is the unique reusable upload gateway.**
4. **Temporary media is not final business media.**
5. **Finalization and assignment are separate responsibilities.**
6. **Comments use the same generic MediaUploader.**
7. **A Comment may contain at most one media attachment.**
8. **Reactions have one canonical mutation gateway.**
9. **Reaction state is determined server-side as Add, Change, or Remove.**
10. **`TargetType + TargetId` remains the canonical polymorphic target mechanism.**
11. **Feed does not own Reactions.**
12. **Feed does not own Comments.**
13. **Feed is a derived distribution/presentation system.**
14. **Post creation and Feed generation are separate responsibilities.**
15. **Feed generation is event-driven.**
16. **Feed propagation may execute asynchronously.**
17. **Transactional Outbox is the durable bridge between authoritative business transactions and downstream processing.**
18. **Important Social business operations produce explicit domain events according to the canonical Event Catalog.**
19. **Domain events describe business facts, not consumer commands.**
20. **The existing Event Catalog is authoritative for event names and semantics.**
21. **A new event must not be invented merely to simplify downstream implementation.**
22. **`Social.Post.Created` does not automatically mean unconditional Feed distribution.**
23. **Feed eligibility determines whether a Post becomes a Feed candidate.**
24. **`Social.Post.Shared` represents the established Post sharing event.**
25. **Comment replies use the canonical Comment event contract unless the Event Catalog is deliberately revised.**
26. **Engagement events generally update Feed signals rather than creating duplicate Feed stories.**
27. **Social Graph changes can alter Feed eligibility and ranking.**
28. **Feed candidate generation is separate from Feed eligibility.**
29. **Feed eligibility is separate from Feed ranking.**
30. **Feed ranking is separate from Feed rendering.**
31. **Feed rendering is separate from Feed business processing.**
32. **Feed is not an authoritative source of social content.**
33. **Feed distributions must be idempotent.**
34. **Feed must remain rebuildable/reprocessable from authoritative state/events where required.**
35. **Comment tree policy is administratively configurable.**
36. **Domain comment depth and component rendering depth remain separate concepts.**
37. **Reusable components own their interaction domain but never own business authority.**
38. **Server-side application/domain logic remains authoritative over client-side behavior.**
39. **Notification consumes Social events independently of Feed.**
40. **Alerting consumes operational/business evidence independently of Feed and Notification.**
41. **Worker Health is application-wide and must support current and future workers.**
42. **Alerting does not own worker execution or recovery.**
43. **Retry and recovery remain separated by failure domain.**
44. **No Social subsystem may create a universal retry or self-healing service.**
45. **External dependencies are isolated behind internal contracts and infrastructure adapters.**
46. **Vendor-specific SDKs and payloads do not leak into business/application logic.**
47. **Audit remains a cross-cutting platform capability.**
48. **Administration governs Social policy without bypassing domain/application ownership.**
49. **Moderation changes authoritative social state/policy; Feed responds to that state.**
50. **No Social subsystem may become a God service.**
51. **Every Feed-relevant event must be documented through the complete business-event lifecycle.**
52. **Every asynchronous consumer must be idempotent.**
53. **Strong consistency applies to authoritative Social state and required event persistence.**
54. **Eventual consistency is acceptable for downstream Feed, Notification, Alert, and derived processing.**
55. **Facebook-class functionality is the product capability benchmark; SocialConnect implementation remains governed by its own canonical architecture and contracts.**

---

# 114. Implementation Sequence

Implementation must follow the finalized architecture rather than mixing all Social capabilities into one development phase.

## Phase 1 — Contract Finalization

Finalize:

* Social Media requirements;
* Post requirements;
* Media requirements;
* Reaction requirements;
* Comment requirements;
* Sharing requirements;
* Feed requirements;
* Social Graph requirements;
* Social Event Catalog alignment;
* Feed Event Impact Matrix.

---

## Phase 2 — Media

Finalize and verify:

```text
MediaUploader
MediaUploadService
Temporary Storage
MediaFinalizationService
MediaAssignmentService
```

Verify both:

```text
Automatic Upload
Manual Upload
```

---

## Phase 3 — Post

Finalize:

```text
Post Creation
Post State
Post Visibility
Post Location
Post Media
Post Events
```

Verify:

```text
Post Operation
→ Social.Post.Created
→ Outbox
```

---

## Phase 4 — Reaction

Finalize:

```text
Reaction Target Resolution
Reaction Mutation
Reaction Aggregate
Reaction Events
```

Verify:

```text
Add
Change
Remove
```

through the single canonical mutation gateway.

---

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

Verify:

```text
Domain Depth
+
Rendering Depth
```

remain properly separated.

---

## Phase 6 — Sharing

Finalize:

```text
Internal Share
SharedPostId
Shared Post Graph
Maximum Share Depth
Cycle Protection
Social.Post.Shared
External Share
```

---

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

---

## Phase 8 — Feed Event Vertical Slices

For each Feed-relevant event:

```text
Business Operation
→ Authoritative State
→ Domain Event
→ Outbox
→ Dispatcher
→ Handler
→ Candidate / Audience Resolution
→ Eligibility
→ Feed Impact
→ Idempotency
→ Retry / Recovery
→ Tests
```

---

## Phase 9 — Notification and Alerting Integration

Prove independently:

```text
Business Operation
→ Domain Event
→ Outbox
→ Dispatcher
→ Consumer
→ Recipient Resolution / Alert Policy
→ Notification / Alert
→ Tests
```

Feed remains an independent consumer.

---

## Phase 10 — Worker Health Integration

Verify Feed and future Social workers participate in:

```text
Worker
  ↓
Common Worker Health Contract
  ↓
Heartbeat
Progress
Success
Failure
Current Operation
  ↓
Operational Evidence
  ↓
Alert Policy
```

No Feed-specific Worker Health implementation is permitted.

---

## Phase 11 — End-to-End Social Verification

Verify complete vertical slices such as:

```text
Post Creation
→ Social.Post.Created
→ Outbox
→ Dispatcher
→ Feed Impact
→ Feed Distribution
→ Feed Rendering
→ Reaction
→ Reaction Event
→ Feed Signal
→ Comment
→ Comment Event
→ Notification
→ Operational Evidence
→ Alerting
```

Each stage must be independently observable and testable.

---

# 115. Definition of Done

The Social Media subsystem is not considered complete merely because:

```text
Post CRUD works
```

or:

```text
Feed displays Posts
```

It is complete when the **business-event-driven Social lifecycle** is demonstrably correct.

The final proof must establish:

```text
Business Operation
        ↓
Authoritative Domain State
        ↓
Domain Event
        ↓
Transactional Outbox
        ↓
Transaction Commit
        ↓
Dispatcher
        ↓
Event Handler
        ↓
Business Impact
        ├── Feed
        ├── Notification
        ├── Alerting
        ├── Audit
        └── Other Consumer
        ↓
Idempotency
        ↓
Failure / Retry
        ↓
Worker Health / Operational Evidence
        ↓
Correct UI Projection
```

The authoritative business state must remain in its owning domain throughout the entire lifecycle.

The Feed must remain a derived distribution system.

Notification must remain a notification platform.

Alerting must remain an operational alert platform.

Worker Health must remain the application-wide source of worker operational evidence.

External providers must remain behind internal contracts and infrastructure adapters.

The Social Media subsystem is therefore complete only when the entire chain is demonstrably correct:

```text
BUSINESS TRUTH
      ↓
EVENT
      ↓
DURABLE ASYNCHRONOUS PROCESSING
      ↓
DOMAIN-SPECIFIC IMPACT
      ↓
DERIVED PROJECTIONS / COMMUNICATIONS
      ↓
OBSERVABILITY
      ↓
RECOVERY
```

This is the canonical SocialConnect Social Media architecture and requirements contract from which the individual Social Media domain/model requirements must be derived.
