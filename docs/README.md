# Discord Redis Specification

The Discord Redis Specification (DRC Spec.) defines the format of
Discord data when stored in Redis. This aims to make interoperability
between cache implementations possible.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 and as later
updated by RFC 8174.

## Authorship

The first draft of this specification was created [Bluenix][Bluenix2]
and was subsequently improved and finalized together with
[Sammy][SammyWhamy] and [Luke Stoodley][Luke-6723] with advice from
[Simon Beal][muddyfish].

  [Bluenix2]: https://github.com/Bluenix2
  [SammyWhamy]: https://github.com/SammyWhamy
  [Luke-6723]: https://github.com/Luke-6723
  [muddyfish]: https://github.com/muddyfish

## Table of Contents

* [1. Specification](./specification.md)
  * [1.1 Versioning](./specification.md#11-versioning)
  * [1.2 Key Format](./specification.md#12-key-format)
  * [1.3 Expiry and TTL](./specification.md#13-expiry-and-ttl)

* [2. Encodings](./encodings.md)

* [3. Key Reference](./key-reference.md)
