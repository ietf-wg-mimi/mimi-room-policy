---
title: "Room Policy for the More Instant Messaging Interoperability (MIMI) Protocol"
abbrev: "MIMI Room Policy"
category: info

docname: draft-ietf-mimi-room-policy-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - room policy
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "ietf-wg-mimi/mimi-room-policy"
  latest: "https://ietf-wg-mimi.github.io/mimi-room-policy/draft-ietf-mimi-room-policy.html"
author:
 -
  fullname: Rohan Mahy
  organization: Rohan Mahy Consulting Services
  email: rohan.ietf@gmail.com

normative:

informative:


--- abstract

This document describes a set of concrete room policies for the
More Instant Messaging Interoperability (MIMI) Working Group. It describes
several independent properties and policy attributes which can be combined
to model a wide range of chat and multimedia conference types.

--- middle

# Introduction

The MIMI architecture {{!I-D.ietf-mimi-arch}} describes how each room
has an associated policy. Providers offer a "policy envelope"
of supported and allowed policy settings, from which the creator of a room
selects a specific room policy. The room policy might further allow
individual participants to make specific choices (for example, allowing
but not requiring read-message notifications), while constraining other
choices (for example, prohibiting self-deleting messages). Individual
users can examine the room policy to determine if it is consistent with
policies they accept either before or immediately on joining a room.
{{Section 4.4 of !I-D.ietf-mimi-arch}}

Making rooms interoperable across existing clients is challenging, as rooms
and clients can support different policies and capabilities across vendors
and providers. Our goal is to balance the policy and authorization goals of
the room with the policy and authorization goals of the end user, so we can support a broad range of vendors and providers.

Each room is owned by one provider at a time. The owning provider controls the range of acceptable policies. The user responsible for the room can further choose among the acceptable policies. Users (regardless if on other providers) can either accept the policies of the room or not. However we want to make it as easy as possible for clients from other providers to comply with the room policy primitives without enumerating specific features or requiring all clients implementations to present an identical user experience.

Configurable Role-based access control with permissions. An example
scheme is described in the Appendix.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Role:
A long-lived position reflecting the privilege level of a participant in a room. Possible values are owner, admin, regular-user, visitor, none, or outcast. A role closely maps to the MUC concept of affiliation, not the MUC concept of role.

Occupant:
A user that has at least one client in the corresponding MLS group, and a role of owner, admin, regular-user, or visitor.

Room ID:
An identifier which uniquely identifies a room.

User ID:
An internal identifier which uniquely identifies a user.

Nickname:
The identifier by which a user is referred inside a room. Depending on the context it may be a display name, handle, pseudonym, or temporary identifier. The nickname in one room need not correlate with the nickname for the same user in a different room.

Client ID:
An internal identifier which uniquely identifies one client/device instance of one user account.

**Persistent vs. Temporary rooms**:
A temporary room is destroyed when the last occupant exits whereas a persistent room is not destroyed when the last occupant exist. As MLS has no notion of a group with no members, a persistent room could consist of a sequence of distinct MLS groups, zero or one of which would exist at a time.

## Moderation Terms

Knock:
To request entry into a room.

Ban:
To remove a user from a room such that the user is not allowed to re-enter the room (until and unless the ban has been removed). A banned user has a role of "outcast". Typically this action is only used in an open or semi-open room. It is distinct than merely removing a user from an administered group.

Kick:
To temporarily remove a participant or visitor from a room. The user is allowed to re-enter the room at any time.

Voice (noun):
The privilege to send messages in a room.

Revoke Voice:
To remove the permission to send messages in a room.

Grant Voice:
To grant the permission to send messages in a room.

Moderator:
A client whose user has a role of admin or owner in the room. A moderator can ban users, and (when allowed in the room) can grant and revoke voice, kick individual clients, and grant access (ex: in response to a knock).

# Room Capabilities

Membership-Style:
The overall approach of membership authorization in a room, which could be open, members-only (administrated), fixed-membership, or parent-dependent.

Open room:
An open room can be joined by any non-banned user.

Members-Only room:
A members-only room can only be joined by a user in the occupant list,
or who is pre-authorized. Authorized users can add or remove users to the
room. In an enterprise context, it is also common (but not required) for
users from a particular domain, group, or workgroup to be pre-authorized to
add themselves to a Members-Only room.

Fixed-Membership room:
Fixed-membership rooms have the list of occupants specified when they are created. Other users cannot be added. Occupants cannot leave or be removed, however a user can remove all its clients from the associated MLS group. The most common case of a fixed-membership room is a 1:1 conversation. This room membership style is used to implement Direct Message (DM) and Group DM features. Only a single fixed-membership room can exist for any unique set of occupants.

Parent-dependent room:
In a parent-dependent room, the list occupants of the room must be a strict subset of the occupants of the parent room. If a user leaves or is removed from the parent room, that user is automatically removed from any parent-dependent rooms of that parent.

Multi-device vs. Single-device:
A multi-device room can have multiple simultaneous clients of the same user as participants in the room. A single-device room can have a maximum of one client per user in the room at any moment.

Knock-Enabled vs. Knock-Disabled:
In a knock-enabled room, non-banned users are allowed to programmatically request entry into the room. In a knock-disabled room this functionality is disabled.

Moderated vs. Unmoderated:
An an unmoderated room, any occupant can send messages to the room. In a moderated room, only occupants who have "voice" can send messages to the room.

# Room policy format syntax

## Membership-related policy

The `membership_style` of a room can be one of the following values:

- open
- members-only (default)
- fixed-membership
- parent-dependent

~~~
enum {
  reserved(0)
  open(1),
  members-only(2),
  fixed-membership(3),
  parent-dependent(4),
  (255)
} MembershipStyle;
~~~

An open room allows any unbanned user. A members-only room allows only
invited or preauthorized users to join. A fixed-membership room (which can
be used for DMs or Group DMs) has a list of authorized users set at creation
time that cannot be added to. A parent-dependent room always has a strict subset of the participants of its parent room.

If the membership_style is `parent-dependent` the `parent_room_uri` MUST be set with the room ID of the parent. Otherwise the field is zero-length.

If `multi_device` is true (the default), the MLS group may contain multiple clients per user. If false only a single client can be an MLS member at one time.

If `knock_allowed` is true, a non-participant can send a knock requesting access to the target room. If false, a user cannot. This option can only be enabled if the membership_style is members-only. The default is false.

If `moderated` is true, the room supports granting and revoking voice. The default is false.

~~~
enum {
  false(0),
  true(1)
} bool;

struct {
  MembershipStyle membership_style;
  Uri parent_room_uri<V>;
  bool multi_device;
  bool knock_allowed;
  bool moderated;
  bool persistent_room;
  bool password_protected;
  ...
} RoomPolicy;
~~~

If persistent_room is false, the room will be automatically deleted when the corresponding MLS group is destroyed (when there are no clients in the group). If persistent_room is true, the room policy will remain and a client whose user has appropriate authorization can create a new MLS group for the same room. (There is not a 1:1 correlation of MLS group to room ID in a persistent room.)

If password_protected is true, the room requires a passcode or passphrase when a client of a new user requests access to the GroupInfo used to join a group. The default is false.

## Pre-authorized users

~~~
enum {
  reserved(0),
  system(1),
  owner(2),
  admin(3),
  regular_user(4),
  visitor(5),
  banned(6),
  (255)
} Role;

struct {
  Role target_role;
  /* preauth_domain consists of ASCII letters, digits, and hyphens */
  opaque preauth_domain<V>;
  /* the remaining fields are in the form of a URI */
  opaque preauth_workgroup<V>;
  opaque preauth_group<V>;
  opaque preauth_user<V>;
} PreAuthPerRoleList;

struct {
  ...
  PreAuthPerRoleList pre_auth_list<V>;
  ...
} RoomPolicy;
~~~

In members-only rooms, it is convenient to pre-authorize specific users--or users from specific domains, workgroups/teams, and groups--to specific roles. The workgroup, group, and user are expressed as a Uri. The domain is expressed in US-ASCII letters, digits, and hyphens only. If the domain is internationalized, the Internationalized Domain Names {{!RFC5890}} conversion MUST be done before filling in this value.

Note that the system role is used to authorize external proposals for operations for other users. For example, the system role can be used to authorize a provider to remove clients from groups when the corresponding user account is deleted.

## Delivery and Read notifications, Pseudonyms

~~~
enum {
  optional(0),
  required(1),
  forbidden(2)
} Optionality;

struct {
  ...
  Optionality delivery_notifications;
  Optionality read_receipts;
  bool pseudonymous_ids;
  ...
} RoomPolicy;
~~~

The delivery_notifications value can be set to "forbidden", "optional", or "required". If the value is set to "optional", the client uses its local configuration to determine if it should send delivery notifications in the group.

The read_receipts value can be set to "forbidden", "optional", or "required". If the value is set to "optional", the client uses its local configuration to determine if it should send read receipts in the group.

The format for delivery notifications and read receipts is described in Section 5.12 of {{?I-D.ietf-mimi-content}}.

If pseudonymous_ids is true, clients in the MLS group are free to use pseudonymous identifiers in their MLS credentials. Otherwise the policy of the room is that "real" long-term identifiers are required in MLS credentials in the room's corresponding MLS group.

## Link, Logging, History, and Bot policies

~~~ tls
struct {
  bool on_request;
  Uri join_link;
  bool multiuser;
  uint32 expiration;
  Uri link_requests;
} LinkPolicy;

struct {
  Optionality logging;
  Uri logging_clients<V>;
  Uri machine_readable_policy;
  Uri human_readable_policy;
} LoggingPolicy;

struct {
  Optionality history_sharing;
  Role who_can_share<V>;
  bool automatically_share;
  uint32 max_time_period;
} HistoryPolicy;

struct {
  opaque name<V>;
  opaque description<V>;
  Uri homepage;
  Role bot_role;
  bool can_read;
  bool can_write;
  bool can_target_message_in_group;
  bool per_user_content;
} Bot;

struct {
  ...
  bool discoverable;
  LinkPolicy link_policy;
  LoggingPolicy logging_policy;
  HistoryPolicy history_sharing;
  Bot allowed_bots<V>;
  ...
} RoomPolicy;
~~~

### Link policies

If discoverable is true, the room is searchable. Presumably this means the the only way to join the room in a client user interface is to be added by an administrator or to use a joining link.
Inside the LinkPolicy are several fields that describe the behavior of links.If the on_request field is true, no joining link will be provided in the room policy; the client will need to fetch a joining link out-of-band or generate a valid one for itself. If present, the URI in link_requests can be used by the client to request an invite code. The value of join_link is empty and the other fields are ignored.If the on_request field is false, the join_link field will contain a joining link. If the link will work for multiple users, multiuser is true. The expiration field represents the time, in seconds after the start of the UNIX epoch (1-January-1970) when the link will expire. The link_requests field can be empty.

### Logging policies

Inside the LoggingPolicy, the logging field can be forbidden, optional, or required. If logging is forbidden then the other fields are empty. If logging is required, the list of logging_clients needs to contain at least one logging URI. Each provider should have no more than one logging client at a time in a room. The machine_readable_policy and human_readable_policy fields optionally contain pointers to the owning provider's machine readable and human readable logging policies, respectively. If logging is optional and there is at least one logging_client then logging is active for the room.

### Chat history policies

Inside the HistoryPolicy, if history_sharing is forbidden, this means that clients (including bots) are expected to not to share chat history with new joiners, in which case who_can_share is empty, automatically_share is false, and max_time_period is zero.
Otherwise who_can_share is a list of roles that are authorized to share history (for example, only admins and owners can share). The values of none and outcast cannot be used in who_can_share. If automatically_share is true, clients can share history with new joiners without user initiation. The history that is shared is limited to max_time_period seconds worth of history.

### Chat bot policies

Inside the RoomPolicy there is a list of allowed_bots. Each of which has several fields. The name, description, and homepage are merely descriptive. The bot_role indicates if the chat bot would be treated as a system-user, owner, admin, regular_user, or visitor.
The can_read and can_write fields indicate if the chat bot is allowed to read messages or send messages in the MLS group, respectively. If can_target_message_in_group is true it indicates that the chat bot can send an MLS targeted message (see Section 2.2 of [I-D.ietf-mls-extensions]) or use a different conversation or out-of-band channel to send a message to specific individual users in the room. If per_user_content is true, the chat bot is allowed to send messages with distinct content to each member. (For example a poker bot could deal a different hand to each user in a chat).Users could set policies to reject or leave groups with bots rights that are inconsistent with the user's privacy goals.

## Extensibility of the policy format

Finally, The extensibility mechanism allows for future addition of new room policies.

~~~
enum {
  null(0),
  boolean(1),
  number(2),
  string(3),
  jsonObject(4)
} ExtType;

struct {
  opaque name<V>;
  ExtType type;
  opaque value<V>;
} PolicyExtension;

struct {
  ...
  PolicyExtension policy_extensions<V>;
} RoomPolicy;
~~~

foo


# Role-Based Access Control

With Role-Based Access Control, the room policy defines a list of roles and the capabilities associated with each role.

~~~
enum {
  reserved(0),
  canAddParticipant(1),
  ...
  (65535)
} Capability;

struct {
  int role_index;
  opaque role_name<V>;
  opaque role_description<V>;
  Capability role_capabilities<V>;
  int minimum_participants_constraint;
  optional int maximum_participants_constraint;
  int minimum_active_participants_constraint;
  optional int maximum_active_participants_constraint;
} Role

struct {
  Role target_role;
  /* preauth_domain consists of ASCII letters, digits, and hyphens */
  opaque preauth_domain<V>;
  /* the remaining fields are in the form of a URI */
  opaque preauth_workgroup<V>;
  opaque preauth_group<V>;
  opaque preauth_user<V>;
} PreAuthPerRoleList;

struct {
  Role roles<V>;
  ...
  PreAuthPerRoleList pre_auth_list<V>;
  ...
} RoomPolicy;
~~~

## Capability Categories

### Membership Capabilities

- canAddParticipant
- canRemoveParticipant
- canAddOwnClient
- canRemoveSelf
- canAddSelf
- canCreateJoinLink
- canUseJoinLink

These actions are subject to role constraints described below.

### Moderation Capabilities

- canBan
- canUnBan
- canKick
- canRevokeVoice
- canGrantVoice
- canKnock
- canAcceptKnock
- canChangeUserRole
- canCreateSubgroup

These actions are subject to role constraints described below.

### Breakouts

- canSendDirectMessage
- canTargetMessage

### Message Capabilities

- canSendMessage
- canReceiveMessage
- canReportAbuse
- canReactToMessage
- canEditReaction
- canDeleteReaction
- canEditOwnMessage
- canDeleteOwnMessage
- canDeleteAnyMessage
- canStartTopic
- canReplyInTopic
- canSendDirectMessage
- canTargetMessage

The Hub can enforce whether a member can send a message and not fanout application messages. Other capabilities can only be enforced by other clients.


### Asset Capabilities

- canUploadImage
- canUploadVideo
- canUploadAttachment
- canDownloadImage
- canDownloadVideo
- canDownloadAttachment
- canSendLink
- canSendLinkPreview
- canFollowLink

### Adjust metadata

- canChangeRoomName
- canChangeRoomSubject
- canChangeRoomAvatar
- canChangeOwnName
- canChangeOwnPresence
- canChangeOwnMood
- canChangeOwnAvatar

### Real-time media

- canStartCall
- canJoinCall
- canSendAudio
- canReceiveAudio
- canSendVideo
- canReceiveVideo
- canShareScreen
- canViewSharedScreen

### Disruptive Policy Changes

- canChangeRoomMembershipStyle
- canChangeOtherPolicyAttribute
- canDestroyRoom
- MLS specific
  - update
  - reinit
  - PSK
  - external proposal
  - external commit

## Role constraints

Minimum and maximum number of each role.

# Operational policy

Section 7 of the {{?I-D.ietf-mls-architecture}} defines a set of operational
policy considerations that influence interoperability of MLS clients. MIMI
explicitly address a handful of the issues in the document by taking a position on ordering (Proposals referenced in a Commit need to be received before the Commit; the Commit entering a new epoch needs to be received before any other messages in that epoch), privacy of handshake messages (handshakes can be a PublicMessage or SemiPrivateMessage), and GroupInfo storage (committers need to provide a valid GroupInfo to the Hub). The rest of these issues are described here. Just because a topic is listed does not mean that a room needs to take a position; nor different rooms on a Hub need to have different policies for these items.

## Some MLS-related policy that could be tied to a room

- any mandatory or forbidden MLS extensions.

- which proposals are valid to have in a commit, including but not limited to:
    - when, and under what circumstances, a reinitialization proposal is allowed.
    - when proposals from external senders are allowed and how to authorize those proposals.
    - when external joiners are allowed and how to authorize those external commits.
    - which other proposal types are allowed.
- when members should commit pending proposals in a group.

- when two credentials represent the same client.
- how long to allow a member to stay in a group without updating its leaf keys before removing them.

- When and how to pad messages.
- When to send a reinitialization proposal.
- How often clients should update their leaf keys.
- Whether to prefer sending full commits or partial/empty commits.
- Whether there should be a required_capabilities extension in groups.

- minimum and maximum lifetime of KeyPackages
- if last resort KeyPackages are allowed
- how long to store resumption PSK (how much time and how many epochs)
- minimum and maximum number past epochs to keep
- how long to keep unused nonce and key pairs for a sender
- maximum number of unused key pairs to keep
- maximum number of steps that clients will move a secret tree ratchet forward in response to a single message before rejecting it
- tolerance to out of order app messages
- tolerance to out of order handshake messages
- handshakes may be which of PublicMessage, PrivateMessage, or SemiPrivateMessage.
- if external joiners are allowed
- if external proposals are allowed
    - if so, who can submit
    - which member(s) are responsible for submitting pending proposals
- how a joiner gets access to the ratchet_tree

## Not relevant to MIMI (between client and its provider)

- how many KPs to keep active
- how group IDs are constructed
- which ciphersuites are acceptable.

## Areas for future works

Which credential types are allowed/required

How to protect and share the GroupInfo objects needed for external joins.

If an application wishes to detect and possibly discipline members that send malformed commits with the intention of corrupting a group's state, there must be a method for reporting and validating malformed commits.
MLS requires the following parameters to be defined, which must be the same for two implementations to interoperate:

Which media types are required to send and required to understand in MIMI.

What Additional authenticated data, can/should be sent unencrypted in an otherwise encrypted message.

Application-level identifiers of public key material (specifically the application_id extension as defined in Section 5.3.3 of [RFC9420]).


# Some types of rooms with RBAC

## Strict administrator policy

Role levels (low to high capabilities):

- banned
- guest
- ordinary_user
- group_admin
- super_admin

Capabilities per role

- banned
    - (no capabilities)
- guest
	- canSendMessage
	- canReceiveMessage
	- canRemoveSelf
- ordinary_user
    - (everything guest can do, plus:)
	- canAddOwnClient
	- canChangeOwnName,Presence,Mood,Avatar
	- canReport
	- canSendLinks
    - canSendImages
    - canSendVideos
- group_admin
	- (everything ordinary_user can do, plus:)
	- canSendAttachments
	- canAddParticipant
	- canRemoveParticipant
	- canBanish - promote
	- canUnBanish
	- canKick
	- canChangeRoomName,Subject,Avatar
	- canPromoteRole(from=[ordinary]; to=[admin])
	- canDemoteRole([from=[admin], to=[ordinary])
	- canDestroyRoom
	- canCreateJoinLink
- super_admin
    - (everything group_admin can do, plus:)
    - canPromoteRole(from=[ordinary,admin], to=[superadmin])
    - canDemoteRole(from=[superadmin], to=[ordinary,admin])
    - canChangeRoomMembershipStyle

Role constraints: must have at least 1 "admin"

## Cooperative room (everyone can add and remove)

Role levels (low to high capabilities):

- banned
- ordinary_user
- group_admin
- super_admin

Capabilities per role

- banned
    - (no capabilities)
- ordinary_user
	- canSendMessage
	- canReceiveMessage
	- canRemoveSelf
	- canAddOwnClient
	- canChangeOwnName,Presence,Mood,Avatar
	- canReport
	- canAddParticipant
	- canRemoveParticipant
	- canKick
	- canChangeRoomName,Subject,Avatar
	- canCreateJoinLink
	- canSendLinks
    - canSendImages
    - canSendVideos
    - canSendAttachments
- group_admin
	- (everything ordinary_user can do, plus:)
	- canBanish
	- canUnBanish
	- canPromoteRole(from=[ordinary], to=[admin])
	- canDemoteRole([from=[admin], to=[ordinary])
	- canDestroyRoom
- super_admin
    - (everything group_admin can do, plus:)
    - canPromoteRole(from=[ordinary,admin], to=[superadmin])
    - canDemoteRole(from=[superadmin], to=[ordinary,admin])
    - canChangeRoomMembershipStyle

Role constraints: must have at least 1 "group_admin"


## Moderated room

Role levels (low to high capabilities):

- banned
- non-participant
- guest
- attendee
- speaker
- moderator
- super_admin

Capabilities per role

- banned
    - (no capabilities)
- non-participant
	- canKnock
	- canJoinViaLink
- guest
	- canReceiveMessage
	- canRemoveSelf
- attendee
    - (everything guest can do, plus:)
	- canAddOwnClient
	- canChangeOwnName,Presence,Mood,Avatar
	- canReport
- speaker
    - (everything attendee can do, plus:)
	- canSendMessage
	- canChangeRoomName,Subject,Avatar
	- canSendLinks
    - canSendImages
    - canSendVideos
    - canSendAttachments
- moderator
	- (everything ordinary_user can do, plus:)
	- canAddParticipant
	- canRemoveParticipant
	- canAcceptKnock
	- canBan
	- canUnBan
	- canKick
	- canPromoteRole(from=[attendee]; to=[speaker])
	- canDemoteRole([from=[speaker], to=[attendee])
	- canCreateJoinLink
- super_admin
    - (everything moderator can do, plus:)
	- canDestroyRoom
    - canPromoteRole(from=[attendee,speaker], to=[moderator])
    - canDemoteRole(from=[moderator], to=[attendee,speaker])
    - canChangeRoomMembershipStyle

Role constraints: must have at least 1 "moderator"


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Complete TLS Presentation Language Syntax

~~~
enum {
  false(0),
  true(1)
} bool;

struct {
  /* a valid Uniform Resource Identifier (URI) */
  opaque uri<V>;
} Uri;

enum {
  optional(0),
  required(1),
  forbidden(2)
} Optionality;

enum {
  reserved(0),
  system(1),
  owner(2),
  admin(3),
  regular_user(4),
  visitor(5),
  banned(6),
  (255)
} Role;

struct {
  Role target_role;
  /* preauth_domain consists of ASCII letters, digits, and hyphens */
  opaque preauth_domain<V>;
  /* the remaining fields are in the form of a URI */
  opaque preauth_workgroup<V>;
  opaque preauth_group<V>;
  opaque preauth_user<V>;
} PreAuthPerRoleList;

enum {
  reserved(0)
  open(1),
  members-only(2),
  fixed-membership(3),
  parent-dependent(4),
  (255)
} MembershipStyle;

struct {
  Optionality logging;
  bool enabled;
  Uri logging_clients<V>;
  Uri machine_readable_policy;
  Uri human_readable_policy;
} LoggingPolicy;

struct {
  bool on_request;
  Uri join_link;
  bool multiuser;
  uint32 expiration;
  Uri link_requests;
} LinkPolicy;

struct {
  opaque name<V>;
  opaque description<V>;
  Uri homepage;
  Role bot_role;
  bool can_read;
  bool can_write;
  bool can_target_message_in_group;
  bool per_user_content;
} Bot;

struct {
  Optionality history_sharing;
  Role who_can_share<V>;
  bool automatically_share;
  uint32 max_time_period;
} HistoryPolicy;

enum {
  null(0),
  boolean(1),
  number(2),
  string(3),
  jsonObject(4)
} ExtType;

struct {
  opaque name<V>;
  ExtType type;
  opaque value<V>;
} PolicyExtension;

struct {
  MembershipStyle membership_style;
  bool multi_device;
  bool knock_allowed;
  bool moderated;
  bool password_protected;
  PreAuthPerRoleList pre_auth_list<V>;
  Uri parent_room_uri;
  bool persistent_room;
  Optionality delivery_notifications;
  Optionality read_receipts;
  bool semi_anonymous_ids;
  bool discoverable;
  LinkPolicy link_policy;
  LoggingPolicy logging_policy;
  HistoryPolicy history_sharing;
  Bot allowed_bots<V>;
  PolicyExtension policy_extensions<V>;
} RoomPolicy;

RoomPolicy room_policy;
~~~

# Example Roles and Permissions scheme

- Owner
- Admin
- Moderator
- Ordinary-user
- Guest


# Acknowledgments
{:numbered="false"}

TODO acknowledge.


