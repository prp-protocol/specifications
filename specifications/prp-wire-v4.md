# PRP Wire Protocol Version 4

- **Version:** 4
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies version 4 of the Participant Relationship Protocol
(PRP) wire protocol. It defines canonical integer encoding, logical units,
route-record lists, session establishment (HS2), identity and session suite
selection, cryptographic transcripts, traffic-key derivation, protected units,
fragmentation, and protocol evolution rules.

This document is independent of carrier technology, programming interfaces,
operating systems, and deployment architecture.

## 1. Status and requirements language

This document is a Working Draft. Numeric assignments, byte layouts, and the
current complete conformance-vector set are consolidated from independently
consumed version-4 contracts. Promotion to Draft Standard requires closure of
every structure referenced by the base registry and independent review of the
resulting specification and vectors.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in this document are to be interpreted as described by BCP 14 when,
and only when, they appear in all capitals.

## 2. Conventions

All multibyte integers are unsigned and encoded in network byte order (most
significant byte first). Integer notation such as `u16` includes that rule.

Byte offsets begin at zero. Concatenation is denoted by `||`. Literal domain
strings are their ASCII bytes without a terminating NUL. Reserved fields MUST
be encoded as zero and MUST be rejected when nonzero.

The maximum logical PRP unit is 65,535 bytes. This limit is not a carrier MTU or
a preferred transmission size. A carrier or local policy MAY impose a lower
unit limit.

## 3. Version selection

The version-4 number is `0x04`. A version is immutable within a session. An
endpoint MUST NOT parse version-4 bytes as another version, fall back after a
version-4 validation failure, or infer a version from malformed input.

The version value is not a universal packet marker. Established session state
selects the wire version and session suite before an AOP is parsed. Public HS2
carries the version in its canonical header.

A migration between wire versions requires a new session. When relationship
continuity is to be preserved, migration SHOULD establish and authenticate the
new session before draining the old session.

## 4. Protocol scopes and selectors

Session scopes are:

| Value | Scope |
| ---: | --- |
| `0x01` | adjacent |
| `0x02` | direct |
| `0x03` | end-to-end (E2E) |

A route alias is a 16-bit selector scoped to the adjacent session and its
generation. Alias `0x0000` is invalid. Alias `0xffff` is reserved for adjacent
control. A route alias is not an identity and MUST NOT be interpreted outside
its assigned scope.

The egress capability value `0xffffffff` denotes local direct delivery. It has
no child layer.

## 5. Adjacent operation packet

An Adjacent Operation Packet (AOP) carries exactly one route-record list (RRL).
For a protected session suite, its wire representation is:

```text
nonce[12] || ciphertext || tag[16]
```

The ciphertext plaintext is the complete RRL. For a suite with no AEAD, the AOP
is the RRL itself and carries no nonce or tag.

The effective AOP ceiling is the minimum of the carrier limit, receiver limit,
and sender policy. A sender MAY vary the canonical AOP size across transmission
opportunities.

## 6. Route-record list

An RRL is:

```text
record_count:u16 || records || opaque_padding
```

Each record is:

```text
route_alias:u16 || kind:u8 || flags:u8 || body_length:u16 || body
```

Exactly `record_count` records are decoded. Remaining authenticated plaintext
is opaque padding and MUST NOT be reinterpreted as another record.

Record kinds are:

| Value | Kind |
| ---: | --- |
| `0x01` | adjacent control |
| `0x02` | onion |
| `0x03` | fragment |
| `0x04` | adjacent ACK |
| `0x05` | adjacent NACK |
| `0x06` | E2E item |
| `0x07` | scoped HS2 fragment |

Only `fragment` accepts the `RECOVERY` flag `0x01`. All other flag bits are
zero. An unknown kind, unknown flag, invalid alias, truncated body, or body
length extending beyond the RRL MUST be rejected.

An adjacent-control record uses alias `0xffff`. Every other record uses an
alias other than `0x0000` and `0xffff`.

## 7. Canonical structure inventory

| Structure | Canonical layout |
| --- | --- |
| HS2 | `version:u8 || session_suite_id:u16 || kind:u8 || ordered_tlvs` |
| HS2 TLV | `type:u8 || length:u16 || value` |
| Scoped HS2 | `scope:u8 || route_alias:u16 || HS2` |
| Onion | `direct_layer_length:u16 || direct_layer || E2E_annex` |
| Direct plaintext | `commitment_length:u16 || commitment || branch_count:u16 || branches` |
| Branch | `egress_capability:u32 || child_layer_length:u16 || child_layer` |
| Fragment | `object_kind:u8 || reserved:u8 || transfer_id:u32 || offset:u16 || total_length:u16 || segment` |
| ACK/NACK | `range_count:u16 || ranges` |
| ACK/NACK range | `transfer_id:u32 || offset:u16 || length:u16` |
| E2E item | `connection_alias:u16 || flags:u16 || sequence:u64 || body_length:u16 || body` |
| Adjacent control | `control_kind:u8 || kind_specific_body` |
| Alias lifecycle | `operation:u8 || flags:u8 || route_alias:u16 || generation:u32 || lease_seconds:u32` |
| Scoped error | `scope:u8 || class:u8 || reason:u16 || retry_after_ms:u32 || token_length:u16 || token` |
| Close or close ACK | `scope:u8 || mode_or_status:u8 || reason_or_reserved:u16 || generation:u32 || selector:u32` |
| Relation control | `kind:u8 || version:u8 || operation:u8 || reserved:u8 || scope:u8 || reserved:u8 || route_alias:u16 || transaction_id:u64 || section_count:u16 || reserved:u16 || sections` |
| Relation section | `type:u16 || version:u8 || flags:u8 || status:u8 || reserved:u8 || body_length:u16 || generation:u32 || base_generation:u32 || body` |

This inventory fixes base framing. A kind-specific specification MUST close the
semantics and validation of its body before that kind can be emitted.

An onion has a nonzero direct-layer length. Bytes after that exact direct layer
form the E2E annex and may be empty.

The direct-plaintext commitment length is exactly the commitment length of the
selected session suite. Each branch has a nonzero egress capability. A local
branch uses capability `0xffffffff` and an empty child layer; a nonlocal branch
uses a different capability and a nonempty child layer.

### 7.1 Onion annex commitment

The creator of each direct layer MUST set its commitment to the session-suite
hash of the exact E2E annex byte string that follows that direct layer:

```text
annex_commitment = session_suite_hash(E2E_annex)
```

No domain string, length prefix, or other bytes are added to this preimage. The
onion framing determines the exact annex boundary. An empty annex is committed
as the hash of the zero-length byte string; its commitment is not an all-zero
placeholder.

Every receiver MUST reject a missing or incorrectly sized commitment. This
structural requirement is independent of whether the receiver recomputes the
commitment.

A receiver that terminates the annex, delivers it locally, or derives protocol
state or authority from it MUST recompute the commitment using that direct
session's suite and MUST compare the complete hash output with the
commitment in the direct plaintext. It MUST reject an unequal commitment before
the annex or any branch has an externally visible local effect. A unit containing
both local and transit branches is subject to this rule for every branch; the
receiver MUST validate the commitment before producing any branch effect.

A transit-only forwarding participant that treats the annex as opaque MAY omit
recomputation when its selected profile and local policy permit opaque transit.
It MUST still validate the direct-layer and annex boundaries, canonical branch
inventory, commitment presence and length, installed session and generation,
route alias, egress capabilities, and every other applicable framing,
authentication, replay, and authorization requirement. It MUST preserve the
annex and selected child layer byte-for-byte and MUST NOT report the annex or
commitment as validated. If it attempts commitment verification, a hash failure
or unequal result MUST reject the unit before forwarding.

A profile MAY require commitment verification for transit-only forwarding.
Whether a transit participant recomputes the commitment is processing policy;
it does not change the wire representation and is not signaled by replacing,
zeroing, or omitting the commitment. Structural parsing needed to establish the
direct layer, annex boundary, and canonical branch inventory MAY precede a
required comparison, but no affected branch may have an externally visible
effect before that comparison succeeds.

Every nested child direct layer carries its own commitment to the same annex,
calculated using the suite of the direct session in which that child layer will
be processed. Consequently, commitments in successive layers can use different
algorithms or lengths. Ordinary transit forwarding promotes the selected child
layer and preserves the annex byte-for-byte; it does not create or replace the
child layer's commitment.

A generic fragment has a nonzero transfer ID, nonzero total length, and a
nonempty segment whose range is contained in that total. Its object kind is a
defined record kind other than `fragment` or `scoped HS2 fragment`; recursive
fragment wrapping is invalid. Each ACK or NACK range has a nonzero transfer ID,
nonzero length, and a range contained within the logical-unit limit.

## 8. Adjacent control registry

Adjacent control kinds are:

| Value | Kind |
| ---: | --- |
| `0x01` | permanently reserved |
| `0x02` | HS2 |
| `0x03` | alias lifecycle |
| `0x04` | scoped error |
| `0x05` | close |
| `0x06` | close acknowledgment |
| `0x07` | public HS2 fragment |
| `0x08` | relation control |
| `0x09` | resource report |
| `0x0a` | HS2 admission offer |
| `0x0b` | HS2 admission challenge |
| `0x0c` | HS2 admission proof |

The reserved `0x01` value MUST NOT be decoded, emitted, or reassigned. Unknown
control kinds fail closed unless a future version defines an authenticated
negotiated extension rule.

## 9. PRP identity addresses

A version-4 identity address is:

```text
identity_suite_id:u16 || strong_id[identity-suite-defined]
```

The identity suite selects the strong-ID hash and length, public-key format,
proof format, and proof algorithm. An unknown identity suite MUST be rejected;
none of those properties may be inferred from an observed length.

The strong ID is:

```text
HASH("PRP-STRONG-ID-v4" || identity_suite_id:u16 || public_key)
```

`HASH` is selected by the identity suite. The public key is its exact canonical
encoding. A different suite or public key therefore produces a different PRP
identity address.

The initial assignments are maintained in
[`identity-suites-v4.tsv`](../registries/identity-suites-v4.tsv) and summarized
below.

| ID | Name | Strong ID | Public key | Proof |
| ---: | --- | --- | ---: | ---: |
| `0x0001` | ED25519-BLAKE3-256 | BLAKE3, 32 bytes | 32 | 64 |
| `0x0002` | ED25519CTX-SHA256-256 | SHA-256, 32 bytes | 32 | 64 |
| `0x0003` | MLDSA65-SHA256-256 | SHA-256, 32 bytes | 1952 | 3309 |
| `0x0004` | MLDSA87-SHA384-384 | SHA-384, 48 bytes | 2592 | 4627 |
| `0x0005` | ED25519-BLAKE3-MLDSA87-BLAKE3-256 | BLAKE3, 32 bytes | 2624 | 4691 |

Suite `0x0005` encodes its public key as `Ed25519[32] || ML-DSA-87[2592]`
and its proof as `Ed25519[64] || ML-DSA-87[4627]`. Both proof components are
required. Algorithm standardization does not assert validation or certification
of a particular cryptographic module.

Suite `0x0005` is the RECOMMENDED general PRP identity suite. A policy requiring
only NIST-standardized algorithms may select suite `0x0003` or `0x0004`; that
selection still does not assert module certification.

Deterministic all-zero-public-key strong-ID results are provided in
[`identity-strong-id-v4.tsv`](../vectors/wire-v4/identity-strong-id-v4.tsv).

## 10. Session suites

Identity and session suites are independent. The session suite selects key
establishment, transcript and fragment hash, traffic-key derivation, AEAD,
nonce length, and tag length. The signed HS2 transcript binds the selected
session suite.

The initial assignments are maintained in
[`session-suites-v4.tsv`](../registries/session-suites-v4.tsv).

| ID | Name | Transcript/commitment | Nonce/tag | Ephemeral | KEM public/ciphertext |
| ---: | --- | --- | --- | ---: | ---: |
| `0x0001` | NONE-BLAKE3 | BLAKE3/32 | 0/0 | 0 | 0/0 |
| `0x0002` | NONE-SHA256 | SHA-256/32 | 0/0 | 0 | 0/0 |
| `0x0101` | X25519-CHACHA20POLY1305-BLAKE3 | BLAKE3/32 | 12/16 | 32 | 0/0 |
| `0x0102` | X25519-AES256GCM-BLAKE3 | BLAKE3/32 | 12/16 | 32 | 0/0 |
| `0x0201` | X25519-MLKEM768-CHACHA20POLY1305-BLAKE3 | BLAKE3/32 | 12/16 | 32 | 1184/1088 |
| `0x0202` | X25519-MLKEM1024-CHACHA20POLY1305-BLAKE3 | BLAKE3/32 | 12/16 | 32 | 1568/1568 |
| `0x0301` | MLKEM768-AES256GCM-SHA256 | SHA-256/32 | 12/16 | 0 | 1184/1088 |
| `0x0302` | MLKEM1024-AES256GCM-SHA384 | SHA-384/48 | 12/16 | 0 | 1568/1568 |

Where this document specifies the session-suite hash, the hash is the unkeyed
algorithm named by the suite. SHA-256 and SHA-384 use their complete fixed-size
digest. BLAKE3 uses unkeyed `hash` mode beginning at output offset zero and takes
the first registered commitment-length bytes. A formula includes only the bytes
shown in its preimage; no implicit domain string or length prefix is added.

Suite `0x0202` is the RECOMMENDED general protected session suite. Suite
`0x0302` is the RECOMMENDED strict-NIST session suite. A strict-NIST policy
requires a compatible NIST identity suite independently of the session-suite
choice.

The `NONE` suites remain structurally complete and still require an identity
proof in HS2. They provide no confidentiality, cryptographic integrity, traffic
authentication, replay protection, traffic key, nonce, or tag. Policy MAY
reject either `NONE` suite or restrict the records accepted through it.
Continuous relation control changes persistent relationship state and therefore
MUST NOT be carried through a `NONE` suite. Its authentication requirements are
defined by the PRP Relationship Lifecycle specification.

A `RESOURCE_REPORT` remains structurally decodable through `NONE`, but such a
record is not authenticated operational evidence and MUST NOT update accepted
relationship-report state. Its semantic admission requirements are defined by
PRP Protected Resource Reporting.

## 11. HS2 session establishment

### 11.1 Message form

HS2 is a two-flight establishment exchange. M1 has kind `0x01`; M2 has kind
`0x02`. Every HS2 begins:

```text
version:u8=4 || session_suite_id:u16 || flight_kind:u8
```

The initiator selects one exact session suite in M1. M2 MUST repeat that suite.
There is no counteroffer or fallback inside the exchange.

An HS2 message MUST NOT exceed 16,384 bytes after reconstruction.

### 11.2 TLV registry and canonical order

Each TLV is `type:u8 || length:u16 || value`. TLVs occur at most once and are
encoded in strictly increasing type order.

| Type | Name | Rule |
| ---: | --- | --- |
| `0x00` | discovery material | optional, opaque to base HS2 |
| `0x01` | nonce | required, 16 bytes |
| `0x02` | ephemeral key | present exactly when selected by the session suite |
| `0x03` | KEM public key | M1 only, present exactly for a KEM suite |
| `0x04` | KEM ciphertext | M2 only, present exactly for a KEM suite |
| `0x05` | identity suite | required, exactly 2 bytes |
| `0x06` | identity public key | required, identity-suite-defined length |
| `0x07` | M1 binding | M2 only, session-suite hash length |
| `0x08` | session generation | M2 only, nonzero `u32` |
| `0x09` | identity proof | required, identity-suite-defined length; final TLV |

The ephemeral and KEM lengths MUST match the selected session suite. Public-key
and proof lengths MUST match the selected identity suite. A parser MUST reject
duplicate, unordered, unknown, missing, forbidden, or length-invalid TLVs.

The optional discovery-material value is opaque to base HS2. Wire version 4
does not use it to activate relationship discovery. Discovery capability is
directional Relation Control policy established after HS2. An endpoint MUST
NOT infer policy from the existence of TLV `0x00`, append a new HS2 TLV, or
treat an untransmitted local parameter as peer agreement.

### 11.3 Identity proof

The proof prefix is the complete canonical HS2 byte sequence from its version
byte up to, but excluding, the identity-proof TLV header. Thus the M2 proof also
covers the M1 binding and session generation. The proof TLV header and proof
value are not part of the prefix.

Identity proofs are:

| Suite | Operation |
| ---: | --- |
| `0x0001` | Ed25519 over `BLAKE3("PRP-HS2-ID-PROOF-v4" || prefix)` |
| `0x0002` | RFC 8032 Ed25519ctx over `prefix`, context `PRP-HS2-v4` |
| `0x0003` | ML-DSA-65 over `prefix`, external context `PRP-HS2-v4-PQ` |
| `0x0004` | ML-DSA-87 over `prefix`, external context `PRP-HS2-v4-PQ` |
| `0x0005` | suite `0x0001` proof followed by suite `0x0004` proof over the same prefix |

Context and domain bytes are fixed by this specification and are never
peer-supplied strings. Both M1 and M2 carry an independent proof by their
sender. Proof verification MUST use the suite explicitly encoded in that
message.

### 11.4 M1 binding and transcript

The M1 binding is the session-suite hash of:

```text
"PRP-HS2-M1-BINDING-v4" || m1_length:u32 || complete_canonical_m1
```

M2 MUST carry the exact binding. A receiver MUST validate it before accepting
M2 as a response to M1 and before verifying the M2 identity proof.

The final transcript hash is the session-suite hash of:

```text
"PRP-HS2-TRANSCRIPT-v4" ||
m1_length:u32 || complete_canonical_m1 ||
m2_length:u32 || complete_canonical_m2
```

The length fields are part of the preimage. The transcript binds both identity
proofs, both nonces, all selected suites, all key-establishment material, the M1
binding, and the session generation.

## 12. Traffic keys and protected units

Key-establishment secret components are concatenated in this order:

```text
X25519_shared_secret[32] || ML-KEM_shared_secret[32]
```

A suite omits a component it does not select.

For BLAKE3 suites, the canonical traffic material is:

```text
scope:u8 || session_suite_id:u16 || generation:u32 ||
transcript_hash_length:u16 || transcript_hash ||
shared_secret_length:u16 || ordered_shared_secret
```

Each directional 32-byte traffic key is BLAKE3 derive-key output over that
material. The derive-key context is `PRP-HS2-I2R-v4` for initiator-to-responder
traffic and `PRP-HS2-R2I-v4` for responder-to-initiator traffic.

Suites `0x0301` and `0x0302` use HKDF-HMAC-SHA-256 and HKDF-HMAC-SHA-384,
respectively. The transcript hash is the salt, the ordered shared secret is the
input keying material, and expand info is:

```text
"PRP-HS2-TRAFFIC-v4" || scope:u8 || direction:u8 ||
session_suite_id:u16 || generation:u32
```

Direction is `0x01` for initiator-to-responder and `0x02` for
responder-to-initiator. Every protected suite produces a 32-byte traffic key.
Keys are distinct by scope, direction, session suite, generation, transcript,
and key-establishment secret.

The protected-unit nonce is:

```text
session_generation:u32 || sequence:u64
```

Both fields are nonzero. Sequences are independent by scope, direction, and
generation. The AEAD associated data is:

```text
"PRP-AEAD-v4" || scope:u8 || session_suite_id:u16 ||
generation:u32 || transcript_hash
```

The associated data is reconstructed from authenticated session state and is
not transmitted with each unit.

A receiver MUST reject a nonce with an unexpected generation before AEAD
processing. It MUST perform replay prevalidation without consuming the sequence
and commit the sequence to its replay window only after successful AEAD
authentication. The replay window contains 64 sequence positions.

## 13. HS2 fragmentation

### 13.1 Commitment

The fragment commitment is the selected session-suite hash of:

```text
"PRP-HS2-FRAGMENT-v4" || hs2_length:u32 || complete_canonical_hs2
```

Only the zero-offset fragment carries start metadata:

```text
session_suite_id:u16 || commitment[session-suite-defined]
```

Continuation fragments carry neither field. The reconstructed HS2 MUST repeat
the suite in the start metadata.

### 13.2 Public adjacent HS2

Public first contact accepts exactly two base shapes. An unfragmented unit is a
bare scoped HS2 with adjacent scope and route alias `0xffff`:

```text
scope=0x01 || route_alias=0xffff || HS2
```

It is not an RRL or an AOP.

A fragmented public unit is:

```text
control_kind=0x07 || scope=0x01 || route_alias=0xffff ||
hs2_kind:u8 || reserved:u8 || transfer_id:u32 ||
offset:u16 || total_length:u16 || start_metadata_if_offset_zero || segment
```

### 13.3 Protected recursive HS2

Direct and E2E establishment occurs through RRL record kind `0x07` inside an
established adjacent AOP. Its record alias selects a nonreserved recursive
route, and its body is:

```text
scope:u8 || hs2_kind:u8 || reserved:u16 || transfer_id:u32 ||
offset:u16 || total_length:u16 || start_metadata_if_offset_zero || segment
```

The scope MUST be direct or E2E. Generic fragment records MUST NOT wrap scoped
HS2 fragments, and the public fragment wrapper MUST NOT be nested in an AOP.

### 13.4 Reassembly

Reassembly state is isolated at least by containing adjacent generation,
direction, route alias, scope, and transfer ID. A receiver MUST reject bytes
outside the declared total, inconsistent totals or kinds, duplicate or
overlapping coverage, inconsistent start metadata, an invalid commitment, or a
reconstructed noncanonical HS2.

Continuation fragments MAY precede the zero-offset fragment. No state
transition occurs until every byte is covered exactly once, the commitment is
verified, and the reconstructed HS2 passes full validation. Receivers MUST
bound incomplete state, lifetime, and completed-transfer replay state.

## 14. Rekey and generation continuity

Rekey is a new M1/M2 exchange with fresh key-establishment material and a
strictly greater, durably nonreused generation. The responder installs new
receive state before sending M2. The initiator installs it only after
authenticating M2. New-generation transmission begins only after the local M2
transition succeeds, with sequence one.

An old receive generation MAY remain for a bounded in-flight overlap. Old
transmission stops at cutover. A rejected new generation MUST NOT roll state
back. Loss of sequence state requires a still newer generation; a generation
or nonce sequence MUST NOT be resumed.

## 15. E2E item boundary

An E2E session terminates at exactly one self-certifying service identity. The
base item is:

```text
connection_alias:u16 || flags:u16 || sequence:u64 ||
body_length:u16 || body
```

An E2E item is the body of one RRL record. Its 14-octet header and body together
MUST fit the enclosing `body_length:u16`; therefore the E2E body is at most
65,521 octets. An E2E item is not split across RRL records.

Connection alias zero carries control for the selected E2E session. Values
`1..65535` are receiver-allocated and scoped by E2E session and generation.
Version 4 has no service alias, service port, or global service selector in an
E2E item.

Defined item flags are `START=0x0001`, `END=0x0002`, and `RESET=0x0004`; all
other bits are zero.

Connection control occurs only in an E2E item whose connection alias is zero.
Such an item also has zero item flags and sequence. Its nonempty control body
begins with a one-byte kind:

| Value | Kind | Length | Canonical body |
| ---: | --- | ---: | --- |
| `0x01` | OPEN | 17 | `kind:u8 || request_id:u32 || flags:u16 || window:u16 || max_queued_bytes:u32 || tombstone_ms:u32` |
| `0x02` | ACCEPT | 23 | `kind:u8 || request_id:u32 || connection_alias:u16 || generation:u32 || flags:u16 || window:u16 || max_queued_bytes:u32 || tombstone_ms:u32` |
| `0x03` | REFUSE | 11 | `kind:u8 || request_id:u32 || reason:u16 || retry_after_ms:u32` |
| `0x04` | CLOSE | 10 | `kind:u8 || connection_alias:u16 || generation:u32 || mode:u8 || reason:u16` |
| `0x05` | CLOSE_ACK | 8 | `kind:u8 || connection_alias:u16 || generation:u32 || status:u8` |
| `0x06` | FEEDBACK | 23 | `kind:u8 || connection_alias:u16 || generation:u32 || next_sequence:u64 || received_bitmap:u64` |

Request IDs, accepted connection aliases, and generations are nonzero. Control
flags are `ORDERED=0x0001`, `RELIABLE=0x0002`, and
`MESSAGE_BOUNDARIES=0x0004`; all other bits are zero. Reliable delivery
requires ordered delivery. Ordered delivery requires a window in `1..64` and
a nonzero queue-byte limit. Unordered delivery encodes both fields as zero.
Every accepted policy has a nonzero tombstone duration.

REFUSE reasons are `NONE=0`, `POLICY=1`, `RESOURCE=2`, `UNSUPPORTED=3`,
`STALE=4`, and `SHUTDOWN=5`. CLOSE modes are `GRACEFUL=1` and `HARD=2`.
CLOSE_ACK statuses are `CLOSED=0` and `ALREADY_CLOSED=1`.

### 15.1 Explicit application agreement extension

PRP Application Agreement v1 assigns connection-control kinds 0x07..0x0a
for experimental application-bound opening, acceptance, confirmation and
refusal. See `prp-application-agreement-v1.md` and
`registries/connection-control-kinds-v1.tsv` for their exact versioned bodies.
Legacy kinds above are unchanged. Legacy OPEN/ACCEPT MUST NOT be interpreted
as application agreement. An implementation without this extension rejects
its kinds under Section 16; an application requiring agreement MUST NOT
downgrade to legacy opening. This extension adds no item flags or data envelope.

## 16. Error behavior and extensibility

Unless an enclosing specification explicitly provides an ignorable extension
rule, an endpoint MUST reject:

- unknown suites, kinds, flags, scopes, or required algorithms;
- nonzero reserved fields;
- noncanonical ordering or duplicate fields;
- inconsistent lengths, generations, aliases, or commitments;
- invalid identity proofs, transcript bindings, or AEAD tags; and
- state transitions not valid for the current scope and generation.

Failure of one candidate session MUST NOT cause fallback to a weaker suite or
wire version. Error signaling MUST NOT require disclosure of unauthenticated
state or act as an amplification oracle.

## 17. Security considerations

Identity possession proves control of the key material selected by the
identity suite. It does not establish a civil identity, authorization, or
external certification. Authorization remains relationship policy.

The transcript domains, length prefixes, suite identifiers, scope, direction,
and generation provide cross-protocol and cross-context separation. Their exact
bytes are protocol constants and MUST NOT be configurable from peer input.

The `NONE` suites have no cryptographic traffic protection. Their presence in
the registry is not a recommendation for use on an untrusted carrier.

An onion annex commitment is a per-direct-layer consistency binding, not a
standalone proof of origin. When the direct layer is authenticated, that
protection binds the expected commitment to the direct peer. A receiver that
recomputes the commitment detects annex modification at that layer. A permitted
transit-only omission removes that layer's early consistency check but does not
remove, satisfy, or weaken validation required by a nested direct layer or E2E
protection. An altered annex can therefore cross that transit participant but
is not thereby accepted by any later protected layer or terminating endpoint.
Under a `NONE` direct suite, an active party able to change both the annex and
direct plaintext can recompute the commitment; the check then detects accidental
corruption and mismatched layer/annex association but does not authenticate
either one.

Implementations MUST bound parsing work, fragment state, incomplete handshakes,
replay state, and cryptographic allocations before authentication. Admission
mechanisms may add stricter bounds but cannot replace HS2 identity proof or
grant service authorization.

## 18. Conformance vectors

The version-4 vector set is:

| Vector set | Scope |
| --- | --- |
| [`codec-vectors-v4.tsv`](../vectors/wire-v4/codec-vectors-v4.tsv) | Canonical structures and rejection behavior |
| [`annex-validation-policy-v4.tsv`](../vectors/wire-v4/annex-validation-policy-v4.tsv) | Required termination checks and permitted opaque-transit behavior |
| [`crypto-vectors-v4.tsv`](../vectors/wire-v4/crypto-vectors-v4.tsv) | HS2 proofs, transcript, keys, nonce, AAD, and protected payload |
| [`identity-strong-id-v4.tsv`](../vectors/wire-v4/identity-strong-id-v4.tsv) | Strong-ID derivation for every identity suite |
| [`mlkem-vectors-v4.tsv`](../vectors/wire-v4/mlkem-vectors-v4.tsv) | Deterministic ML-KEM known-answer material |
| [`public-unit-vectors-v4.tsv`](../vectors/wire-v4/public-unit-vectors-v4.tsv) | Complete and fragmented public HS2 units |
| [`scoped-hs2-fragment-vectors-v4.tsv`](../vectors/wire-v4/scoped-hs2-fragment-vectors-v4.tsv) | Protected direct and E2E HS2 fragmentation |
| [`traffic-key-vectors-v4.tsv`](../vectors/wire-v4/traffic-key-vectors-v4.tsv) | Traffic-key derivation for every session suite |

[`manifest.sha256`](../vectors/wire-v4/manifest.sha256) fixes the exact bytes of
this vector revision.

A version-4 codec MUST accept every vector marked `accept` and MUST reject every
vector marked `reject-*` for the stated reason class. An implementation claiming
support for a cryptographic suite MUST reproduce that suite's applicable
known-answer outputs exactly. Rejection vectors are not an error-recovery or
fallback instruction.

The normative text defines protocol semantics. A disagreement between text and
a vector is a specification defect requiring review; an implementation MUST NOT
silently choose one interpretation and claim unqualified conformance.

## 19. IANA considerations

This document currently requests no IANA action. A future Internet-Draft may
request a PRP registry after registration policy and change control are defined.

## 20. References

### 20.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8032, *Edwards-Curve Digital Signature Algorithm (EdDSA)*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- FIPS 203, *Module-Lattice-Based Key-Encapsulation Mechanism Standard*.
- FIPS 204, *Module-Lattice-Based Digital Signature Standard*.
- *PRP Relationship Lifecycle and Continuous Control*, version 1.
- *PRP Protected Resource Reporting*, version 1.
- *PRP Discovery Architecture*, version 1.

### 20.2 Informative references

- *PRP Architecture*, version 1.
