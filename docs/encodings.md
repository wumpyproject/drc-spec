# 2. Encodings

Encodings define how the value of the key MUST be interpreted and
decoded. Encodings MAY make changes to the structure of the data stored
or other parts of the specification, but SHOULD keep those changes at a
minimum.

If an implementation encounters an extension which is unsupported,
unknown, or in any other way invalid it MUST raise an error and refuse
to continue. This ensures that interoperability is not silently broken.

* [2.1 JSON](#21-json)
* [2.2 Protocol Buffers](#22-protocol-buffers)

## 2.1 JSON

* Identifier: `json`

JSON encoding is the RECOMMENDED default which implementations should
use if the user does not pass on explicitly.

## 2.2 Protocol Buffers

* Identifier: `proto`
