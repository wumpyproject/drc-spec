# 3. Key Reference

This section defines the normalization rules and each key available.
Implementations MUST use the data type and format defined in this
section and MUST NOT allow the user to override how data is serialized,
normalized, or stored - refer to what is written in this specification.

There may be exceptions or amendments to this section depending on the
[encoding](./encodings.md) used - these MUST be followed over the rules
defined in this section.

## 3.1 Key Definitions

In the following example keys, `drc` is used as the prefix and `v1json`
is the version info - refer to
[1.2 Key Format](./specification.md#12-key-format). Curly brackets mark
a variable.

This specification makes heavy use of hashes to store related child
objects. This allows implementations to efficiently fetch those by
their parent through Redis's `HGETALL`, `HKEYS`, or `HVALS` command.
As both an upside and downside, Redis only allows entire keys to have
an expiry, which means that all related child objects expire together.
To combat this, individual keys (using the string type) are used for
objects which require their own expiry.

### Application Commands

* `drc:v1:commands:{application.id}` (hash)
* TTL: ...

Hash of *created* application command objects.

### Interactions

* `drc:v1:interactions:{interaction.id}` (string)
* TTL: 15 minutes (900 seconds)

String for storing normalized data of an interaction. This is a string
due to each interaction needing to have its own expiry.

### Applications

* `drc:v1:applications:{application.id}` (string)
* TTL: ...

String storing the normalized application data.

### Auto Moderation Rules

* `drc:v1:automod-rules:{guild.id}` (hash)
* TTL: ....

Hash with all automoderation rules for the guild. The hash is a mapping
of the auto moderation rule's ID to its normalized data.

### Guild Channels

* `drc:v1:channels:{guild.id}` (hash)
* TTL: ...

Hash of all types of channels in a guild. The hash maps the channel's
ID to its normalized data.

### Messages

* `drc:v1:messages:{message.id}` (string)
* TTL: ...

String for the normalized message data. As a result, each key stores
a single message. The reason this isn't a hash is due to requiring
individual expiry times. If this were to be a hash, it would be
continuously updated and the hashes for very active channels would
grow infinitely.

### Threads

* `drc:v1:threads:{thread.id}` (string)
* TTL: ...

String for each thread, storing its normalized data. This has to be a
string due to each thread's data needing its own expiry.

### Thread Members

* `drc:v1:thread-members:{thread.id}` (hash)
* TTL: ...

Hash of a thread member ID to its normalized data.

### Reactions

This resources requires two keys to fully represent the data in a way
which is normalized. First, a set of emojis which have been reacted to
the message:

* `drc:v1:reactions:{message.id}` (set)
* TTL: ...

> **Note**
> A compromise between performance and information has to be made here;
> checking whether an emoji is already present in a linked-list is very
> inefficient

Secondly, another set is used to represent the users who have reacted
with the specific emoji:

* `drc:v1:reactions:{message.id}:{emoji}` (set)
* TTL: ...

### Guild Emojis

* `drc:v1:emojis:{guild.id}` (hash)
* TTL: ...

Hash of custom emoji IDs to its normalized data.

### Guilds

* `drc:v1:guilds` (set)
* TTL: ...

Index key storing a set of all guild IDs the bot is a member of.

* `drc:v1:guilds:{guild.id}` (string)
* TTL: ...

String for each guild, storing its normalized data. This is to allow
each guild to expire independently.

### Guild Widget

* `drc:v1:widget:{guild.id}` (string)
* TTL: ...

String storing the normalized widget data for a specific guild.

### Guild Members

* `drc:v1:members:{guild.id}` (set)
* TTL: ...

Similar to guilds set, this is a set of user IDs used as an index of
all users in a particular guild.

* `drc:v1:members:{guild.id}:{user.id}` (string)
* TTL: ...

String representing the normalized member data for the specific user.
This has to be a string to allow per-member expiry.

### Guild Integrations

* `drc:v1:integrations:{guild.id}` (hash)
* TTL: ...

Hash of all integrations in a guild, mapping the integration ID to
its normalized data.

### Guild Bans

* `drc:v1:bans:{guild.id}:{user.id}` (string)
* TTL: ...

String representing the ban reason for the particular user. This has
to be a string to allow per-user expiry, otherwise the hash may grow
infinitely large.

### Guild Welcome Screen

* `drc:v1:welcome-screen:{guild.id}` (string)
* TTL: ...

String storing the normalized welcome screen object for its guild.

### Guild Scheduled Events

* `drc:v1:scheduled-events:{guild.id}` (hash)
* TTL: ...

Hash of scheduled event IDs to its normalized data.

### Guild Templates

* `drc:v1:templates:{guild.id}` (hash)
* TTL: ...

Hash of template codes to their respective normalized guild template.

### Invites

* `drc:v1:invites:{invite.code}` (string)
* TTL: ...

String storing normalized invite data from its invite code.

### Stage Instances

* `drc:v1:stage-instances:{channel.id}` (string)
* TTL: ...

String for looking up the normalized stage instance associated with a
stage channel.

### Stickers

* `drc:v1:stickers:{guild.id}` (hash)
* TTL: ...

Hash mapping custom stickers ID to its normalized sticker data.

### Voice States

* `drc:v1:voice-states:{guild.id}` (hash)
* TTL: ...

Hash mapping user IDs to their normalized voice state data.

### Presences

* `drc:v1:presence:{guild.id}:{user.id}` (string)
* TTL: ...

String storing normalized presence information. This has to be a string
to allow per-key expiry.

### Webhooks

* `drc:v1:webhooks{webhook.id}` (string)
* TTL: ...

String storing normalized webhook data.
