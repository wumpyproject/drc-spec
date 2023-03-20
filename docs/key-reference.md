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
That said, individual keys are used for for objects which require
their own expiry,

### Applications

* `drc:v1:applications:{application.id}` (string)
* TTL: ...

String storing the normalized application data.

### Auto Moderation Rules

* `drc:v1:guilds:{guild.id}:automod-rules` (hash)
* TTL: ....

Hash with all automoderation rules for the guild. The hash is a mapping
of the auto moderation rule's ID to its normalized data.

### Guild Channels

* `drc:v1:guilds:{guild.id}:channels` (hash)
* TTL: ...

Hash of all types of channels in a guild. The hash maps the channel's ID
to its normalized data.

### Messages

* `drc:v1:channels:{channel.id}:{message.id}` (string)
* TTL: ...

String for the normalized message data. As a result, each key stores
a single message. The reason this isn't a hash is due to requiring
individual expiry times. If this were to be a hash, it would be
continuously updated and the hashes for very active channels would
grow infinitely.
