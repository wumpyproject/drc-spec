# 1. Specification

This section defines the overlying structure and important concepts
of the specification. The name does not imply that the other sections
are not part of the Discord Redis Specification.

## 1.1 Versioning

Every backwards incompatible update made to this specification MUST
increment a major version number although multiple backwards
incompatible changes MAY be done in the same update.

The following table can be used to view different versions of this
specification:

| Version | Status | Note            |
| ------- | -----  | --------------- |
| `v1`    | WIP    | Current version |

## 1.2 Key Format

The most important detail of interoperability is that the same key is
used to store the same data between implementations. All keys defined
in this specification will therefore use the following structure, where
a colon (`:`) is used as a delimiter.

```text
prefix:version:resource
```

The prefix avoids collisions when used on the same Redis database which
means that interoperability is explicitly opt-in between
implementations. Each key MUST have a prefix which SHALL NOT be an
empty string. The prefix MAY contain the delimiter character. The
prefix MUST be configurable by the user.

The prefix MAY be optional for the user, in which case it MUST default
to a unique representative name of the implementation. As a result,
connecting the same implementation to the same Redis database without
specifying a prefix makes it implicitly interoperable - as would
presumably be expected.

The version info contains the major version - prefixed by a `v` -
followed by the encoding used (see [2. Encodings](./encodings.md)).
The version info MUST match the following regex:

```javascript
/v(?<version>\d+)(?<encoding>[a-zA-Z_\+-]*)/
```

Considering the version info is part of the key, the result is that
incompatible usages of this specification can be used independently.
For example, this allows rolling updates as there are no migrations
necessary. That said it is RECOMMENDED that implementations provide a
way to delete all keys with a specific version info after the update
has fully rolled out. This could be done through a CLI command or
provided function.

## 1.3 Expiry and TTL

The point of key expiry is ensuring that unused keys get cleaned up.
This allows rolling updates without migrations as keys from previous
versions automatically expire. This specification defines expiries for
each key which implementations SHOULD follow. That said, the user MAY
be provided a way to override these which SHOULD allow them to
configure that no expiry is set for a key.

Keys' TTL MAY be updated through the `EXPIRE` command at the discretion
of each implementation however the value passed SHOULD be the one
defined in this specification, or - if the user has overriden the
value - the value provided by the user.

## 1.4 Data Normalization

To more efficiently use the memory available and ensure that objects do
not become outdated, this specification normalizes the Discord data.

Normalized data MUST have the nested object's key set to its ID
(Discord snowflake). For example, the below codeblocks depict a JSON
payload before- and after it has been normalized.

```json
{
    "user": {
        "id": "344404945359077377",
        "username": "Bluenix",
        "discriminator": "7543",
        "avatar": null
    },
    "note": "This field is retained."
}
```

```json
{
    "user": "344404945359077377",
    "note": "This field is retained."
}
```

The data MUST be normalized according to the rules written in each key,
refer to [3. Key Reference](./key-reference.md) for more.
