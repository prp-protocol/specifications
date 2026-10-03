# PRP Suite-Bound Content Identity Version 1

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## 1. Scope

Successor notice: revision 4 of `prp-reference-representation-v2.md` retains
only IDENTITY and ALIAS and places content naming/verification outside PRP.
This document remains the legacy native-content contract; its bytes, registries
and vectors are not reinterpreted. Do not carry its CONTENT class or PRCD proof
requirements into the successor. Migration remains gated by REF-001.

This document defines `content_id`, an immutable self-certifying reference to
finite content bytes. It defines content suites, the `MERKLE_64K_BINARY`
profile, PRCD root descriptors, binary leaf proofs, discovery target binding
and common multi-result policy. It defines neither a filesystem, content
store, transfer protocol, ownership model nor protected discovery payload kind.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Reference and metadata semantics

A content ID commits only to the exact finite byte string. It is not a
participant, service or entity identity and has no signing key. Names, paths,
media types, owners, authors, permissions, mutable descriptions and local
storage references are external authenticated metadata. They MUST NOT enter
the content ID or alter it.

A provider declaration asserts only current availability. It does not prove
ownership, authorship, semantic equivalence, authorization to redistribute or
continued availability. A false availability claim cannot alter the committed
bytes and MAY reduce that provider's local admission or scheduling weight.

Live and unbounded streams have no content ID until closed as one finite byte
string. A mutable strong alias MAY refer to the latest completed immutable
version without changing any prior content ID.

## 3. Content suites and profile

A content-suite value is exactly a registered wire-v4 identity-suite value.
Only that suite's Strong-ID hash and digest length are reused. Public-key,
signature and proof-of-possession algorithms remain identity concerns and do
not become content operations. The suite ID is integral to the content ID and
every hash preimage. Equal bytes under different suites have different IDs,
including suites using the same hash. Unknown or retired suites fail closed.

The selected session suite is independent. Rekey, session replacement,
carrier replacement and transfer over another protected session do not change
the content ID. The assignments are maintained in
[`content-suites-v1.tsv`](../registries/content-suites-v1.tsv).

Content profile `0x0001` is `MERKLE_64K_BINARY`. Leaves contain at most 65,536
bytes and internal nodes have one or two ordered children. Leaf order is 16 and
fanout order is 1. Different leaf geometry, tree construction or hash domain
requires another content-profile value and MUST NOT be negotiated as an
interpretation of profile 1.

## 4. Merkle construction

All integers are unsigned big-endian. `HASH` denotes the unkeyed hash and
digest length selected by the content suite. Files are split from offset zero
into consecutive 65,536-byte leaves. Only the final leaf may be shorter. The
empty file has exactly one zero-length leaf.

Each leaf digest is:

```text
HASH("PRP-CONTENT-LEAF-v1" ||
     content_suite:u16 || content_profile:u16 ||
     leaf_length:u32 || leaf_bytes)
```

Parents are built left to right. A final unpaired child forms a one-child
parent and is never duplicated. Level 1 consumes leaves and each later level
consumes the preceding level:

```text
HASH("PRP-CONTENT-NODE-v1" ||
     content_suite:u16 || content_profile:u16 ||
     level:u16 || child_count:u16 || ordered_child_digests)
```

Construction stops at one digest. A one-leaf object has tree height zero. For
an exact total length `L`, leaf count is `max(1, ceil(L / 65536))` and tree
height is `ceil(log2(leaf_count))`. Consequently an object at the maximum u64
length has at most 48 proof levels. Leaf position is committed by ordered
parents, allowing identical chunks at different positions without ambiguity.

## 5. PRCD descriptor and content ID

The canonical descriptor has a 32-octet prefix followed by one root digest:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRCD` |
| 4 | 1 | version `0x01` |
| 5 | 1 | flags, zero |
| 6 | 2 | content suite |
| 8 | 2 | content profile `0x0001` |
| 10 | 1 | leaf order `16` |
| 11 | 1 | fanout order `1` |
| 12 | 2 | tree height |
| 14 | 2 | root digest length |
| 16 | 8 | exact total length |
| 24 | 8 | reserved, zero |
| 32 | variable | root digest |

The descriptor length is exactly `32 + root_digest_length`. Suite, profile,
geometry, height, digest length and u64 total length MUST agree with the tree.
No trailing bytes are permitted.

The content ID is:

```text
content_suite:u16 || content_profile:u16 ||
HASH("PRP-CONTENT-ID-v1" || descriptor_length:u16 || descriptor)
```

It is 36 octets for current 256-bit suites and 52 octets for the current
SHA-384 suite. The suite and profile prefix inside the content ID MUST equal
the descriptor. A requester can apply local size policy to the authenticated
u64 length before accepting bulk content.

## 6. Binary leaf proof

A proof begins with this exact 24-octet prefix:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 1 | version `0x01` |
| 1 | 2 | content suite |
| 3 | 2 | content profile |
| 5 | 8 | leaf index |
| 13 | 8 | leaf count |
| 21 | 2 | level count |
| 23 | 1 | reserved, zero |

Each ascending level then contains:

```text
position:u8 || child_count:u8 || reserved:u16 || other_child_digests
```

For profile 1, `child_count` is one or two and `position < child_count`. A
one-child level has no sibling digest. A two-child level carries exactly the
other ordered digest. Level count equals descriptor tree height. Leaf index,
leaf count and every level MUST be possible for the descriptor's total length.
The verifier hashes the supplied leaf, walks levels in ascending order and
requires the final digest to equal the descriptor root. Unknown values,
nonzero reserved bytes, oversized leaves, impossible geometry, truncated or
trailing proof bytes and root mismatch fail closed.

Each leaf can therefore be authenticated before other leaves arrive. Range and
multi-provider retrieval use the same descriptor and proof construction; this
document assigns no transport or scheduling architecture for that retrieval.

## 7. Discovery target binding

`CONTENT_ID=0x04` is the fourth registered discovery target kind. Its
identifier profile is the content suite and its identifier is the complete
content ID, whose integral suite MUST equal that outer identifier profile.
The canonical target key and Bloom or exact-lookup derivations are defined by
the discovery specifications. A content ID MUST NOT be substituted for a
strong ID or used to answer an identity challenge.

One participant MAY expose a bounded local catalog over explicitly selected
roots. Catalog indexing is local administrative state: it does not emit one
PRPU or other publication record per file. Paths, names and storage handles
remain local. A content change creates a new immutable content ID; removal
withdraws only that provider's availability.

A protected exact lookup deliberately directed to that participant MAY consult
its local catalog without a prior Bloom positive. It is a bounded local lookup
and MUST NOT recursively query adjacent peers. Local policy MAY also contribute
catalog content IDs to the ordinary directional Bloom aggregate. Such entries
use the common target-key and density rules; they MUST NOT create a separate
content filter or incomparable density signal.

## 8. Common multi-result policy

Weak aliases and content IDs use exactly one multi-result admission and
selection policy. Neither receives more results, propagation, rate allowance
or a separate fairness class because of its target kind. Initial common limits
are 4,096 admitted candidates per target and 32 returned candidates. Policy MAY
lower both values but MUST apply the same values to both kinds in the same
relationship policy.

Candidates are grouped by a receiver-local `allocation_domain`. The default is
the durable administrative relationship that admitted them. Local policy MAY
merge relationships controlled by one principal and MUST place unqualified
anonymous relationships into bounded shared pools. More identities, sessions,
files or declarations inside one domain do not create more share.

Eligible domains use weighted fair queuing with equal default weight. Each
round selects at most one candidate from a domain before another candidate
from any already served domain. When fewer results than eligible domains fit,
persistent scheduler credit and a rendezvous-secret epoch score rotate service
across requests. Query nonces MUST NOT affect ordering.

Within a domain, ordering uses a rendezvous-secret score over target kind,
canonical target key, selection epoch and a local candidate reference. Neither
the secret nor local reference enters a protected record. An identical query
within one epoch is idempotent and rapid retries do not purchase another share.
Admission, Bloom-density, lookup-rate and failure limits apply before selection.

## 9. Protocol boundaries

This document changes no wire-v4 base field, kernel interface, discovery
envelope kind, terminal result or introduction. The existing 32-octet exact
query can carry a target-derived lookup tag without exposing the target.
Successful delivery of PRCD, leaf proofs or content bytes belongs to an
explicitly selected content service and has no protected discovery payload kind
in this version. A local value MUST NOT be invented for it.

## 10. Conformance

`vectors/content-identity-v1` fixes all registered suite hashes, empty and
multi-leaf trees, u64 descriptor fields, binary proofs, malformed inputs,
session-suite independence and metadata separation. Discovery target and
multi-result vectors are fixed independently by `vectors/discovery-target-v1`.

## 11. IANA considerations

This document currently requests no IANA action.

## 12. References

### 12.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Architecture*, version 1.
- *PRP Wire Protocol Version 4*.
- *PRP Discovery Architecture*, version 1.
- *PRP Sharded Bloom Discovery Data*, version 4.
- *PRP Exact and Oblivious Discovery Lookup*, version 1.
