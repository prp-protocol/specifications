# PRP Sharded Bloom Discovery Data

- **Version:** 4
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies the shard-native Bloom index used by PRP relationship
discovery. It defines coordinates, geometry, density, sparse Merkle
commitments, origin manifests, relationship-bound remote views, shard records,
and bounded microshard queries. Bloom results are non-authoritative hints;
exact lookup and proof remain separate requirements.

## 1. Status and requirements language

This document is a Working Draft. Canonical payloads and computations are
covered by deterministic vectors. The protected lane binding and payload
envelope are assigned independently. No service ID or fragmentation contract
is to be inferred.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Scope

SHARDED_BLOOM_V4 is a probabilistic membership index. It answers only “not
present in this admitted view” or “possibly present.” A positive grants no
identity, publication, reachability, forwarding, service, allocation, or
amplification authority.

This document does not define identity publication, VOPRF issuance, exact
lookup, destination proof, a carrier, storage organization, or an execution
boundary. It defines byte strings and state transitions, not components that
process them.

## 3. Generations and time

The following values are distinct:

- `discovery_control_generation` is the nonzero u32 generation accepted under
  *PRP Directional Discovery Policy*, version 1;
- `filter_generation` is a nonzero u64 generation of one origin's committed
  filter; and
- `previous_filter_generation` identifies its immediate predecessor or is
  zero for the first admitted generation.

A successor has a numerically greater filter generation and names the current
generation exactly. Generations do not wrap. Exhaustion requires a future
version.

`valid_from` and `valid_until` are unsigned seconds since 1970-01-01 00:00:00
UTC with POSIX time semantics. The interval is half open:
`valid_from <= now < valid_until`. A record has `valid_from < valid_until`.
Clock policy MAY narrow admission but MUST NOT extend the signed interval.

## 4. Lookup and target profile binding

The lookup-token profile registry is:

| ID | Name | Token |
| ---: | --- | ---: |
| `0x0001` | BLAKE3_128_DIRECT | 16 bytes |
| `0x0002` | RFC9497_VOPRF_RISTRETTO255_SHA512_128 | 16 bytes |

This document uses the identifier in Bloom coordinates but does not define
token derivation. Direct discovery profiles `0x0001..0x0030` require lookup
profile `0x0001`; VOPRF discovery profiles `0x0101..0x0130` require lookup
profile `0x0002`.

For either discovery-profile family, let `n` be its low byte. The filter bit
order is exactly `15+n`, producing P16 through P63. The selected index
algorithm is `SHARDED_BLOOM_V4=0x0001`, the query algorithm is
`MICROSHARD_V2=0x0001`, and the microshard record profile is `0x01`.

Every lookup target has exactly one canonical target key:

```text
target_kind:u8 || identifier_profile:u16be ||
identifier_length:u16be || identifier
```

The registered target kinds are maintained in
[`discovery-target-kinds-v1.tsv`](../registries/discovery-target-kinds-v1.tsv):

| ID | Name | Identifier profile |
| ---: | --- | --- |
| `0x01` | `STRONG_ID` | registered identity suite |
| `0x02` | `STRONG_ALIAS` | registered alias profile |
| `0x03` | `WEAK_ALIAS` | registered alias profile |
| `0x04` | `CONTENT_ID` | registered content suite |

Identifier length and bytes are canonical for that kind and profile. For a
content ID, its integral content-suite prefix MUST equal the identifier
profile. Unknown kind, profile, noncanonical length or inconsistent content
suite is rejected before hashing. Target keys are non-substitutable even when
their identifier bytes happen to be equal.

## 5. Geometry and coordinates

The filter contains `2^filter_bit_order` logical bits, where
`16 <= filter_bit_order <= 63`. It is divided into logical shards of
`2^shard_bit_order` bits, where `16 <= shard_bit_order <= 18` and
`shard_bit_order <= filter_bit_order`. Bit numbering is least-significant-bit
first within each byte.

For coordinate `i`, where `0 <= i < 16`:

```text
digest_i = BLAKE3(
    "PRP-DISCOVERY-SHARDED-BLOOM-INDEX-v4" ||
    i:u8 || lookup_token_profile:u16be ||
    target_key_len:u16be || target_key)

bloom_index = u64be(digest_i[0..7]) & ((1 << filter_bit_order) - 1)
shard_index = bloom_index >> shard_bit_order
bit_offset  = bloom_index & ((1 << shard_bit_order) - 1)
```

Order 63 leaves the top unsigned u64 bit outside the index space. Geometry is
not a cryptographic-strength or cardinality claim. This target-key derivation
replaces the former strong-ID-only coordinate preimage. One active discovery
generation uses only this derivation and MUST NOT try the earlier preimage as
a fallback.

## 6. Density and composition

Density is computed from bits, never from a claimed publication count:

```text
density_ppm = floor(set_bit_count * 1000000 / 2^filter_bit_order)
```

`set_bit_count` is in `0..2^filter_bit_order`. Complete-source admission
requires all committed shards and a recomputed popcount. Metadata may be
screened against the signed count, but that does not prove density. The
density formula is evaluated over mathematical integers; an endpoint MUST
avoid intermediate overflow.

Composition is bitwise OR. Before mutation, the receiver MUST compute the
exact projected popcount and reject the whole composition if its directional
density limit would be exceeded. Each egress relationship has an independent
aggregate; this document defines no global aggregate or protocol hierarchy.

Folding from order `s` to lower order `d` maps every set coordinate to
`index & ((1 << d) - 1)` and recomputes density. It preserves positives but
may add collisions. Conservative expansion from `s` to higher `d` replicates
each source bit at all coordinates congruent modulo `2^s`. It preserves
positives and density but cannot restore precision. A ranking decision uses
the source effective order, not an expanded storage order.

## 7. Shard commitments and sparse root

The commitment of a complete nonempty shard is:

```text
BLAKE3(
    "PRP-DISCOVERY-SHARDED-BLOOM-SHARD-v4" ||
    filter_generation:u64be || lookup_token_profile:u16be ||
    filter_bit_order:u8 || shard_bit_order:u8 ||
    shard_index:u64be || shard_len:u32be || shard)
```

The empty leaf is:

```text
BLAKE3("PRP-DISCOVERY-SHARDED-BLOOM-EMPTY-v4")
```

Leaves are ordered by shard index. At leaf level zero, and then at each level
toward the root:

```text
BLAKE3(
    "PRP-DISCOVERY-SHARDED-BLOOM-NODE-v4" ||
    tree_level:u8 || left[32] || right[32])
```

The leaf count is exactly
`2^(filter_bit_order-shard_bit_order)`, so no odd-node rule exists. Empty
subtrees are derived recursively from the empty leaf. A sparse construction
MUST produce the same root as the complete logical tree.

## 8. Origin manifest

The manifest authenticated prefix begins with this exact 128-byte header:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRB4` |
| 4 | 1 | version `0x04` |
| 5 | 1 | kind `MANIFEST=0x01` |
| 6 | 2 | flags, zero |
| 8 | 2 | origin identity suite |
| 10 | 2 | origin discovery-proof profile |
| 12 | 2 | lookup-token profile |
| 14 | 1 | filter bit order |
| 15 | 1 | shard bit order |
| 16 | 8 | filter generation |
| 24 | 8 | previous filter generation |
| 32 | 8 | valid from |
| 40 | 8 | valid until |
| 48 | 8 | complete set-bit count |
| 56 | 8 | logical shard count |
| 64 | 32 | shard root |
| 96 | 2 | origin strong-ID length |
| 98 | 2 | reserved, zero |
| 100 | 28 | reserved, zero |
| 128 | variable | origin strong ID, then public key and signature |

The authenticated prefix is the header followed by the exact strong ID. The
full record appends the public key and signature lengths fixed by the origin
identity suite. There is no length field for that suffix and no trailing data.
An unsigned prefix exists only as signing input and MUST NOT be admitted.

The discovery-proof profile MUST numerically equal the origin identity suite.
The public key MUST derive the encoded strong ID using the wire-v4 Strong-ID
operation. The signature operation over the complete authenticated prefix is:

| Suite | Discovery-record proof |
| ---: | --- |
| `0x0001` | Ed25519 |
| `0x0002` | RFC 8032 Ed25519ctx, context `PRP-DISCOVERY-v1` |
| `0x0003` | ML-DSA-65, external context `PRP-DISCOVERY-v1` |
| `0x0004` | ML-DSA-87, external context `PRP-DISCOVERY-v1` |
| `0x0005` | suite `0x0001` proof followed by suite `0x0004` proof over the same prefix |

These are discovery-record operations, not HS2 proof transcripts. The record
magic and version are inside the signed message. Both component proofs of a
hybrid suite are required.

`shard_count` is the logical leaf count and equals
`2^(filter_bit_order-shard_bit_order)`. A valid manifest has a nonzero filter
generation, a zero predecessor or a lower predecessor, valid geometry,
registered suites and profiles, canonical reserved bytes, and a valid
signature and time interval.

## 9. Shard record

A shard record has this exact 64-byte header followed by content and proof:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRBS` |
| 4 | 1 | version `0x04` |
| 5 | 1 | kind `SHARD=0x02` |
| 6 | 2 | flags; bit 0 is `IMPLICIT_EMPTY` |
| 8 | 2 | lookup-token profile |
| 10 | 1 | filter bit order |
| 11 | 1 | shard bit order |
| 12 | 8 | filter generation |
| 20 | 8 | shard index |
| 28 | 4 | shard length |
| 32 | 1 | proof depth |
| 33 | 3 | reserved, zero |
| 36 | 8 | shard set-bit count |
| 44 | 20 | reserved, zero |
| 64 | variable | shard bytes, then proof sibling hashes |

Proof depth is exactly `filter_bit_order-shard_bit_order`. Siblings are 32
bytes each, ordered from leaf level toward the root. At level `l`, bit `l` of
the shard index determines whether the current node is left or right.

A nonempty record has no flags, has exactly
`2^(shard_bit_order-3)` shard bytes, and states their exact popcount. An empty
record has only `IMPLICIT_EMPTY`, zero shard length, zero popcount, and no
shard bytes. Shard index is less than the manifest shard count. Exact length
is `64 + shard_len + 32*proof_depth` and MUST NOT exceed the wire-v4 E2E body
inner-payload maximum of 65,485 octets. Consequently P18 is the largest permitted shard: at
filter order P63 its record is 34,272 octets. P19 would require at least 65,600
octets (and 67,008 at P63) and is forbidden. A shard MUST NOT be split across
E2E items and no continuation record exists.

Admission requires exact equality with an admitted PRB4 manifest or PRVM view
for lookup profile, geometry and generation, followed by proof reconstruction
to its root. Unknown flags, nonzero reserved bytes, wrong length, wrong
popcount, wrong proof count or order, and a wrong root fail without state
change.

## 10. Relationship-bound remote view

PRVM binds a root and geometry to a previously selected discovery tuple. Its
authenticated prefix begins with this exact 120-byte header:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRVM` |
| 4 | 1 | version `0x02` |
| 5 | 1 | kind `VIEW=0x01` |
| 6 | 2 | flags, zero |
| 8 | 4 | discovery-control generation |
| 12 | 2 | selected session suite |
| 14 | 2 | selected identity suite |
| 16 | 2 | discovery index algorithm |
| 18 | 2 | discovery query algorithm |
| 20 | 2 | discovery profile |
| 22 | 2 | lookup-token profile |
| 24 | 2 | origin identity suite |
| 26 | 2 | origin discovery-proof profile |
| 28 | 1 | filter bit order |
| 29 | 1 | shard bit order |
| 30 | 1 | query record profile |
| 31 | 1 | reserved, zero |
| 32 | 8 | filter generation |
| 40 | 8 | previous filter generation |
| 48 | 8 | valid from |
| 56 | 8 | valid until |
| 64 | 8 | complete set-bit count |
| 72 | 8 | logical shard count |
| 80 | 32 | shard root |
| 112 | 2 | origin strong-ID length |
| 114 | 6 | reserved, zero |
| 120 | variable | origin strong ID, then public key and signature |

Strong-ID and proof construction are identical to Section 8. The discovery
control generation, session suite, identity suite, algorithms, and discovery
profile MUST equal the active selection. NONE is invalid. Origin identity
suite and proof profile MUST equal the selected identity suite, and the origin
strong ID MUST equal the authenticated remote peer.

Profile binding follows Section 4. Geometry, generation, validity, count and
root follow Sections 3 and 5 through 8. PRVM independently authenticates the
advertised root; a separately received PRB4 is not required. If both records
describe the same origin and filter generation, every common metadata field
MUST match exactly.

A first view has predecessor zero. A byte-identical replay is idempotent and
does not renew validity or rate limits. A same-generation byte difference,
lower generation, or successor that does not name the current generation
fails closed. Current and previous filters may coexist only within their
signed validity intervals and a bounded local rotation policy.

## 11. MICROSHARD_V2

MICROSHARD_V2 is bounded partial disclosure, not private information
retrieval. The serving peer sees all requested regions.

The requester derives the 16 Bloom coordinates, divides each by 128 to obtain
aligned 16-byte region indexes, removes duplicates, fills to exactly 16
distinct indexes with independently unpredictable cover positions, and sorts
the final list in strictly ascending order. Real and cover positions are not
distinguished on the wire. For filter order `p`, every index is less than
`2^(p-7)`.

The request is exactly 196 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRMQ` |
| 4 | 1 | version `0x02` |
| 5 | 1 | query record profile `0x01` |
| 6 | 1 | filter bit order |
| 7 | 1 | region bytes `16` |
| 8 | 2 | lookup-token profile |
| 10 | 2 | region count `16` |
| 12 | 8 | filter generation |
| 20 | 32 | admitted shard root |
| 52 | 16 | query nonce |
| 68 | 128 | 16 region indexes as u64be |

The nonce is unpredictable and MUST be unique among outstanding requests in
that relationship direction. The response is exactly 452 bytes: magic `PRMS`
replaces `PRMQ`, bytes 4 through 195 repeat the request context exactly, and
bytes 196 through 451 contain the 16 regions in request-index order.

The server answers only from shard material admitted under the stated root,
generation, profile and geometry. The fixed response carries no per-region
Merkle proof. The requester rejects any context difference, unknown request,
duplicate response, invalid index, or response after request consumption.

Request rate, concurrent requests, evaluated bytes and cumulative regions are
bounded per authenticated relationship across filter-generation rotation.
Rotation MUST NOT reset abuse accounting.

## 12. Membership and lifecycle

A complete local filter tests all 16 coordinate bits. A microshard response
tests the corresponding bit within each returned region. All present means
“possibly present”; any absent bit means “not present in this admitted view.”

Only a complete committed shard set can establish exact density. Withdrawal
from the authoritative exact publication index remains immediate even while
an older Bloom generation can still produce a false positive. Additive deltas
are not canonical v4 records; updates use a new manifest or view and complete
changed shards.

Session rekey and path replacement preserve admitted discovery state. Loss of
the authenticated relationship, discovery selection, origin proof, or root
blocks new queries and shard admission. It does not turn cached Bloom bytes
into authority.

## 13. Privacy and security considerations

Coordinates are deterministic for a target key and lookup profile. Anyone able
to enumerate candidate identities, aliases or content IDs can test them
against a complete filter.
Filter geometry, density, roots, changed shards, access patterns and timing can
reveal properties of the indexed set.

Cover regions provide no formal PIR guarantee. Repeated adaptive microshard
queries can progressively disclose the filter. VOPRF lookup-token issuance
does not hide these microshard accesses.

Receivers MUST bound record length before allocation, including shard bytes
and proof depth. Authentication does not make claimed popcount truthful; only
complete recomputation does. Bloom positives never bypass exact lookup,
destination proof, authorization, or amplification policy.

## 14. Conformance

`vectors/sharded-bloom-v4` fixes coordinate, sparse-tree, manifest-prefix,
empty-shard, signed-view, transition, and microshard examples. A conforming
endpoint MUST reproduce those bytes and apply the malformed and
context-divergent rejection behavior in this document.

The protected envelope vectors independently cover outer dispatch, generation,
correlation, replay and item bounds.

## 15. IANA considerations

This document currently requests no IANA action.

## 16. References

### 16.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8032, *Edwards-Curve Digital Signature Algorithm (EdDSA)*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- FIPS 204, *Module-Lattice-Based Digital Signature Standard*.
- *PRP Wire Protocol Version 4*.
- *PRP Directional Discovery Policy*, version 1.
- *PRP Protected Discovery Payload Envelope*, version 1.

### 16.2 Informative references

- *PRP Architecture*, version 1.
