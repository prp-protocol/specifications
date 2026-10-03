# PRP Exact and Oblivious Discovery Lookup

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies exact discovery tags, a canonical-target BLAKE3 profile, an
RFC 9497 VOPRF profile, VOPRF key and evaluation payloads, the protected exact
lookup request, and the destination challenge and proof. Exact lookup confirms
an active candidate; it does not itself grant forwarding or session
authority.

## 1. Status and requirements language

This document is a Working Draft. Deterministic vectors cover canonical-target
tag derivation, the RFC 9497 primitive, PRP VOPRF input and compact-token
framing, canonical payloads, destination proof, and state transitions.
Protected kinds
for key issuance, exact query, challenge and proof are assigned by *PRP
Protected Discovery Payload Envelope*, version 1. Terminal result,
introduction and cancellation signaling remain open under `DISC-005`.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Model and scope

A **publication relationship** controls an identity publication. A **query
relationship** connects a requester to the rendezvous that evaluates an exact
lookup. These relationships need not be the same.

An exact index contains bounded active target candidates. Strong-ID candidates
derive from authorized PRPU publications; alias candidates derive from their
authenticated declarations; content candidates derive from bounded local
availability catalogs. A lookup tag is meaningful only in its registered
target, profile, rendezvous and generation context. A match grants no route,
session, service, publication, storage, ownership or forwarding authority.

SHARDED_BLOOM_V4 and MICROSHARD_V2 remain hint and bitmap-access mechanisms.
VOPRF is not PIR and does not hide microshard coordinates or timing.

## 3. Lookup-token profiles

The registry is:

| ID | Name | Token | Status |
| ---: | --- | ---: | --- |
| `0x0001` | BLAKE3_128_DIRECT | 16 bytes | Registered |
| `0x0002` | RFC9497_VOPRF_RISTRETTO255_SHA512_128 | 16 bytes | Registered |

Direct discovery profiles `0x0001..0x0030` select token profile `0x0001`.
VOPRF discovery profiles `0x0101..0x0130` select token profile `0x0002`. Both
families cover P16 through P63. A participant advertises only acceptable
members of those ranges.

Token-profile choice is not a cryptographic-strength claim. Profile `0x0002`
is not post-quantum; policy MAY reject it. A new construction requires a new
identifier and vectors.

## 4. Canonical-target direct tag version 2

For lookup profile `0x0001`, the canonical target key is:

```text
target_kind:u8 || identifier_profile:u16be ||
identifier_length:u16be || identifier
```

Target kinds are `STRONG_ID=0x01`, `STRONG_ALIAS=0x02`,
`WEAK_ALIAS=0x03` and `CONTENT_ID=0x04`. Their profile and identifier rules are
defined by the discovery-target registry. The direct lookup preimage is:

```text
domain_len:u16be || "PRP-DISCOVERY-TARGET-LOOKUP-v2" ||
rendezvous_strong_id_len:u16be || rendezvous_strong_id ||
filter_generation:u64be || lookup_index_profile:u16be ||
target_key_len:u16be || target_key
```

The domain length is 30 and lookup-index profile is `0x0001`. `lookup_tag` is
the first 16 octets of BLAKE3 over the complete preimage. It changes across
target kind, identifier profile or bytes, rendezvous, filter generation or
lookup profile. A truncated collision remains a bounded false positive and
grants no authority.

This derivation replaces `PRP-DISCOVERY-LOOKUP-v1`. An active relationship and
filter generation select only version 2; senders and receivers MUST NOT derive,
index, retry or accept the former version as a fallback. The target identifier
is absent from the protected request, which limits disclosure but does not
prevent offline candidate enumeration.

## 5. VOPRF profile

Profile `0x0002` is RFC 9497 VOPRF mode `0x01` with ciphersuite
`ristretto255-SHA512`. Context string, hash-to-group, hash-to-scalar,
canonical element and scalar encodings, DLEQ proof, and Finalize are exactly
those of RFC 9497. There is no PRP-specific primitive fallback.

PRP fixes:

| Value | Bytes |
| --- | ---: |
| server public key | 32 |
| blinded element | 32 |
| evaluated element | 32 |
| DLEQ proof | 64 |
| VOPRF output | 64 |
| compact lookup token | 16 |
| maximum batch size | one input |

Canonical elements exclude the identity. Scalars use the RFC ciphersuite's
canonical encoding. Invalid encodings, identity elements, invalid proofs, and
Finalize failure are terminal for that evaluation.

The version-one PRP private input below is assigned only for a strong-ID
target. A target-key VOPRF input covering strong aliases, weak aliases and
content IDs is not assigned. Those targets MUST NOT be offered under VOPRF
discovery profiles until `DISC-006` is resolved; a receiver MUST NOT insert a
target key into the version-one identity fields or infer a private-input
format from the direct version-two preimage.

### 5.1 Private input

The client private input is:

```text
domain_len:u16be || "PRP-DISCOVERY-VOPRF-INPUT-v1" ||
discovery_control_generation:u32be ||
rendezvous_identity_suite:u16be ||
rendezvous_strong_id_len:u16be || rendezvous_strong_id ||
published_identity_suite:u16be ||
published_strong_id_len:u16be || published_strong_id
```

The domain length is 28. The generation is nonzero and equals the active
discovery-control generation of the query relationship. Both identity suites
are registered and both lengths match their suites.

Rendezvous identity and discovery generation prevent reuse across authorities
and replacement discovery state. Session rekey and path replacement within
that state do not change the input.

### 5.2 Compact token

After successful RFC 9497 Finalize:

```text
digest = SHA-512(
    domain_len:u16be || "PRP-DISCOVERY-VOPRF-LOOKUP-TOKEN-v1" ||
    output_len:u16be || voprf_output[64])
lookup_token = digest[0..15]
```

The domain length is 35 and output length is 64. The full VOPRF output MUST
NOT be sent as an exact request.

## 6. Query-relationship VOPRF key

Each query-relationship direction owns at most one VOPRF server key for one
discovery-control generation. The peer acting as rendezvous declares the
public key after profile selection and before admitting a VOPRF Bloom view.

The declaration is exactly 44 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRVK` |
| 4 | 1 | version `0x01` |
| 5 | 1 | kind `KEY=0x01` |
| 6 | 2 | lookup-token profile `0x0002` |
| 8 | 4 | discovery-control generation |
| 12 | 32 | server public key |

The key is a canonical nonidentity ristretto255 element. It is immutable for
that relationship direction and generation. Byte-identical replay is
idempotent; absence, same-generation mutation, wrong profile or generation,
and invalid encoding fail closed. Replacement requires a newer authentically
selected discovery-control generation.

The exact VOPRF index is scoped by:

```text
query_relationship || direction || discovery_control_generation || token
```

Consequently, it is not a single rendezvous-global token index. When an active
publication changes, every affected active query-relationship view changes.
When a query relationship or VOPRF key changes, its view is derived anew from
active publications. This reconciles relationship-specific VOPRF inputs with
publication relationships that may be different.

## 7. VOPRF evaluation payloads

The request is exactly 60 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRVQ` |
| 4 | 1 | version `0x01` |
| 5 | 1 | kind `REQUEST=0x02` |
| 6 | 2 | lookup-token profile `0x0002` |
| 8 | 4 | discovery-control generation |
| 12 | 16 | request ID |
| 28 | 32 | blinded element |

The response is exactly 124 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRVR` |
| 4 | 1 | version `0x01` |
| 5 | 1 | kind `RESPONSE=0x03` |
| 6 | 2 | lookup-token profile `0x0002` |
| 8 | 4 | discovery-control generation |
| 12 | 16 | request ID |
| 28 | 32 | evaluated element |
| 60 | 64 | DLEQ proof |

The request ID is uniformly unpredictable, nonzero, unique among pending
requests in that relationship direction, and not VOPRF input. A response
echoes it exactly. Success or definitive rejection consumes it.

The server applies per-relationship rate, pending-request, byte and evaluation
limits before group operations. Unknown, duplicate or consumed request ID,
context mismatch, malformed element or proof, and wrong declared key fail
without creating exact-query state.

## 8. Exact lookup request

After a Bloom positive, a deliberately directed local-catalog lookup, or, for
an assigned profile `0x0002` strong-ID target, successful VOPRF Finalize, the
requester sends exactly:

```text
lookup_tag[16] || query_nonce[16]
```

The nonce is uniformly unpredictable, nonzero and unique among pending exact
queries in that query-relationship direction. Target kind, identifier profile
and identifier are absent.

The 32 bytes do not carry target, profile, relationship, direction, discovery
generation, filter generation, session suite, or service identity. The
protected lane binding supplies the relationship, direction, policy and
session context. The `EXACT_LOOKUP_QUERY` envelope MUST bind the exact query
context. Its outer kind remains `0x0a`; target lookup version 2 allocates no
new inner or outer request format. A receiver MUST NOT guess context from tag
length or try multiple derivations.

For the direct profile, the exact index key is
`(filter_generation, lookup_index_profile, lookup_tag)` and maps to at most
4,096 bounded local candidate references. For the VOPRF profile, it includes
the scope in Section 6. Pending state binds the incoming protected session,
query nonce, rendezvous, generation, profile, complete canonical target key
and selected candidate.

No match is an ordinary Bloom false positive. This document assigns no
unauthenticated negative-response payload; protected terminal-result signaling
remains unavailable under `DISC-005`.

### 8.1 Target-specific validation

The four target kinds share tag, admission, allocation-domain and bounded
selection machinery, but not proof semantics:

- a strong-ID candidate proves that exact self-certifying identity;
- a strong alias resolves through its authenticated controller declaration and
  each selected concrete member proves its own strong ID;
- a weak alias resolves to authenticated local declarations and each selected
  concrete result proves its strong ID; and
- a content ID never signs an identity challenge. A selected provider presents
  the bounded PRCD descriptor through an explicitly selected content service;
  the requester first validates the requested content ID and then every leaf's
  canonical Merkle proof.

Weak aliases and content IDs use the identical multi-result limits and
allocation-domain fair-share policy in *PRP Suite-Bound Content Identity*.
Before identity proof or content-descriptor validation, a lookup creates only
bounded ephemeral state. It cannot create a trajectory, recursive fanout,
amplification or release a local forwarding or storage handle.

There is no provisional `ACCEPTED` response. This document assigns no terminal
result, introduction or content payload kind. PRCD, proofs and content bytes
MUST NOT be placed in a locally numbered discovery payload.

## 9. Identity-target destination challenge

An exact identity-target match identifies an active candidate but does not
release its introduction material. The rendezvous creates an unpredictable 32-byte challenge,
unique among pending challenges, and constructs:

```text
domain_len:u16be || "PRP-DISCOVERY-CHALLENGE-v1" ||
rendezvous_strong_id_len:u16be || rendezvous_strong_id ||
filter_generation:u64be || selected_session_suite:u16be ||
lookup_token_profile:u16be || lookup_token[16] || query_nonce[16] ||
challenge[32] || published_identity_suite:u16be ||
published_strong_id_len:u16be || published_strong_id
```

The domain length is 26. The complete preimage is also the canonical challenge
payload delivered to the published destination. All suites and generations
equal the pending query and active publication. The selected session suite is
present in that publication's accepted-suite list.

The destination checks the complete context and that the published identity is
its own authorized identity before signing. It MUST NOT sign a syntactically
valid challenge received outside the protected discovery context.

## 10. Destination proof

The destination signs the complete challenge payload using the
discovery-record proof operation defined by *PRP Sharded Bloom Discovery Data*.
The proof profile numerically equals the published identity suite.

The response is:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRPP` |
| 4 | 1 | version `0x01` |
| 5 | 1 | reserved, zero |
| 6 | 2 | discovery-proof profile |
| 8 | 32 | challenge |
| 40 | 2 | proof length |
| 42 | variable | public key followed by signature |

Proof length is exactly the registered public-key length plus signature length
for the profile. There are no trailing bytes. Both components of a hybrid
proof are required.

The rendezvous retrieves pending state by the opaque challenge, requires every
challenge-preimage field to equal that state, derives the published Strong ID
from the supplied public key, verifies the signature, and consumes the
challenge after success or definitive rejection. Unknown, expired, duplicated,
cross-relationship, cross-generation, cross-suite, altered-token, altered-
nonce, and altered-identity proofs fail closed.

Successful proof authorizes only the introduction explicitly defined by a
future registered protected payload exchange. No local forwarding selector or capability
appears in the challenge or PRPP.

## 11. Lifecycle and failure

Publication withdrawal, alias revocation and removal of local content
availability immediately make the affected exact candidates ineligible and
invalidate pending unmatched results. Filter rotation preserves assigned VOPRF
tokens within the same discovery-control generation but changes direct tags.
Discovery-control generation replacement changes VOPRF inputs, keys, indexes
and pending evaluation context.

Session rekey does not change either token profile's durable context. Loss of
the authenticated query relationship or selected discovery state cancels
pending VOPRF evaluations, exact queries and challenges.

Malformed payloads, unavailable profile, absent key, exhausted generation,
invalid proof, resource limit, and state mismatch MUST NOT cause fallback to a
different token profile or disclosure of the candidate Strong ID.

## 12. Security and privacy considerations

BLAKE3_128_DIRECT permits offline candidate testing. VOPRF hides the candidate
and resulting token from the server during issuance under RFC 9497 assumptions,
but the rendezvous already controls its publication database and observes
request timing, relationship metadata, Bloom access and subsequent exact
lookup.

Profile `0x0002` is not post-quantum. Server-key compromise enables token
derivation for inputs in that key's scope. Relationship and generation scoping
limits cross-context reuse but does not repair compromise.

Sixteen-byte tags have finite collision probability. A tag match is never
sufficient identity or content evidence; target-specific validation is
mandatory. Rate limiting is required for VOPRF issuance, exact lookup,
challenge creation and multi-result selection to prevent enumeration and
computation amplification.

## 13. Conformance

`vectors/exact-discovery-v1` fixes the canonical-target version-two preimage,
tag and unchanged request; the
RFC 9497 Appendix A.1.2.1 VOPRF vector; PRP private input and compact token;
PRVK, PRVQ and PRVR; destination challenge and PRPP; and deterministic state
transitions, target-specific proofs, single-derivation cutover and rejection
cases.

Vector conformance does not assign terminal results, introduction or
cancellation tracked by `DISC-005`.

## 14. IANA considerations

This document currently requests no IANA action.

## 15. References

### 15.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- RFC 9497, *Oblivious Pseudorandom Functions (OPRFs) Using Prime-Order Groups*.
- *PRP Wire Protocol Version 4*.
- *PRP Directional Discovery Policy*, version 1.
- *PRP Protected Discovery Payload Envelope*, version 1.
- *PRP Sharded Bloom Discovery Data*, version 4.
- *PRP Relationship Identity Publication*, version 1.
- *PRP Suite-Bound Content Identity*, version 1.

### 15.2 Informative references

- *PRP Architecture*, version 1.
