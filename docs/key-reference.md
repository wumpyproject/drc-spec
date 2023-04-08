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

#### Normalization

Application commands have no normalization steps.

### Interactions

* `drc:v1:interactions:{interaction.id}` (string)
* TTL: 15 minutes (900 seconds)

String for storing normalized data of an interaction. This is a string
due to each interaction needing to have its own expiry.

### Applications

* `drc:v1:applications:{application.id}` (string)
* TTL: ...

String storing the normalized application data.

#### Normalization

```lua
local function normalize_application(application) then
    if application.owner then
        application.owner = normalize_user(application.owner)
    end

    if application.team then
        for i, member in ipairs(message.team) then
            if member.user then
                member.user = normalize_user(member.user)
            end
        end
    end

    redis.call(
        'SET', ARGV[1] .. ':applications:' .. application.id,
        cjson.encode(application)
    )
    return application.id
end
```

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

#### Normalization

```lua
local function normalize_message(message) then
    -- If this message was sent by a webhook, this is not a valid user
    if not message.webhook_id then
        normalize_user(message.author)
    end
    message.author = message.author.id

    if message.member then
        normalize_member(message.member, message.author, message.guild_id)
        message.member = nil
    end

    for i, mention in ipairs(message.mentions) then
        if mention.member then
            normalize_member(mention.member, mention.id, message.guild_id)
            mention.member = nil
        end

        message.mentions[i] = normalize_user(mention)
    end

    -- Reactions can possibly be normalized here

    if message.referenced_message then
        message.referenced_message = normalize_message(message.referenced_message)
    end

    if message.interaction then
        message.interaction.user = normalize_user(message.interaction.user)

        if message.interaction.member then
            normalize_member(
                message.interaction.member, message.interaction.user, message.guild_id
            )
            message.interaction.member = nil
        end
    end

    if message.thread then
        message.thread = normalize_thread(message.thread)
    end

    redis.call('SET', ARGV[1] .. ':messages:' .. message.id, cjson.encode(message))
    return message.id
end
```

### Threads

* `drc:v1:threads:{thread.id}` (string)
* TTL: ...

String for each thread, storing its normalized data. This has to be a
string due to each thread's data needing its own expiry.

#### Normalization

```lua
local function normalize_thread(thread) then
    -- Threads and channels have a lot of data which quickly goes stale,
    -- however this is already understood but kept in-case anyone wishes
    -- to estimate based off of it. Could be removed for additional
    -- performance gains

    thread.member = normalize_thread_member(thread.member)

    redis.call('SET', ARGV[1] .. ':threads:' .. thread.id, cjson.encode(thread))
    return thread.id
end
```

### Thread Members

* `drc:v1:thread-members:{thread.id}` (hash)
* TTL: ...

Hash of a thread member ID to its normalized data.

#### Normalization

```lua
local function normalize_thread_member(thread_member, thread_id) then
    if thread_member.id then
        thread_id = thread_member.id
    end

    if thread_member.member then
        -- If user_id is not set
        thread_member.user_id = normalize_member(thread_member.member)
        thread_member.member = nil
    end

    redis.call(
        'HSET', ARGV[1] .. ':thread-members:' .. thread_id,
        thread_member.cjson.encode(thread)
    )
    return nil
end
```

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

#### Normalization

```lua
local function normalize_member(member, user_id, guild_id) then
    -- guild_id is an extra field present in some gateway events;
    -- this makes the guild_id parameter optional
    if member.guild_id then
        guild_id = member.guild_id
    end

    if member.user then
        -- In the event that user_id was not passed
        user_id = member.user.id
        normalize_user(user)
        member.user = nil
    end

    redis.call('SADD', ARGV[1] .. ':members:' .. guild_id, user_id)
    redis.call(
        'SET', ARGV[1] .. ':members:' .. guild_id .. ':' .. user_id,
        cjson.encode(member)
    )
    return user_id
end
```

### Guild Integrations

* `drc:v1:integrations:{guild.id}` (hash)
* TTL: ...

Hash of all integrations in a guild, mapping the integration ID to
its normalized data.

### Guild Welcome Screen

* `drc:v1:welcome-screen:{guild.id}` (string)
* TTL: ...

String storing the normalized welcome screen object for its guild.

### Guild Scheduled Events

* `drc:v1:scheduled-events:{guild.id}` (hash)
* TTL: ...

Hash of scheduled event IDs to its normalized data.

### Guild Roles

* `drc:v1:roles:{guild.id}` (hash)
* TTL: ...

Hash mapping role IDs to its role data.

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

### Users

* `drc:v1:users:{user.id}` (string)
* TTL: ...

String storing individual user data.

#### Normalization

```lua
local function normalize_user(user) then
    redis.call('SET', ARGV[1] .. ':users:' .. user.id, cjson.encode(user))
    return user.id
end
```

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
