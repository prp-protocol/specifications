# PRP Protected Discovery Payload Envelope Version 1

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## 1. Scope

This document assigns the outer version and payload-kind registry for the
runtime-internal ordered, reliable, message-bounded lane defined by *PRP
Protected Discovery Lane*, version 1. It composes canonical discovery records
without changing their internal encoding. It assigns no service identifier,
application endpoint, generic fragmentation protocol, carrier format, or
implementation boundary.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Envelope

Every discovery-lane E2E item contains exactly one envelope. Unsigned integers
are big-endian.

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 1 | envelope version, `0x01` |
| 1 | 1 | registered payload kind |
| 2 | 2 | flags, zero |
| 4 | 4 | accepted discovery-control generation, nonzero |
| 8 | 8 | subject generation, nonzero |
| 16 | 16 | correlation identifier |
| 32 | 2 | payload length |
| 34 | 2 | reserved, zero |
| 36 | variable | canonical inner payload |

The payload length is exact and no trailing bytes are permitted. The complete
envelope MUST fit one wire-v4 E2E body, whose maximum is 65,521 octets; the
inner payload therefore MUST NOT exceed 65,485 octets. Empty payloads are not
assigned in this version.

The discovery-control generation MUST equal both the active directional policy
generation and the generation bound by the lane. A record under a different
generation, relationship, direction, E2E session, connection generation, or
receiver alias is rejected by the lane and creates no discovery state.

## 3. Registry

The normative assignments are maintained in
[`protected-discovery-payload-kinds-v1.tsv`](../registries/protected-discovery-payload-kinds-v1.tsv).
The complete initial registry is:

| Kind | Name | Class | Canonical inner payload |
| ---: | --- | --- | --- |
| `0x01` | `IDENTITY_PUBLICATION` | durable | PRPU version 1 |
| `0x02` | `BLOOM_MANIFEST` | durable | PRB4 manifest |
| `0x03` | `REMOTE_BLOOM_VIEW` | durable | PRVM version 2 |
| `0x04` | `BLOOM_SHARD` | durable | PRBS version 4 |
| `0x05` | `MICROSHARD_REQUEST` | ephemeral | PRMQ version 2 |
| `0x06` | `MICROSHARD_RESPONSE` | ephemeral | PRMS version 2 |
| `0x07` | `VOPRF_KEY` | durable | PRVK version 1 |
| `0x08` | `VOPRF_REQUEST` | ephemeral | PRVQ version 1 |
| `0x09` | `VOPRF_RESPONSE` | ephemeral | PRVR version 1 |
| `0x0a` | `EXACT_LOOKUP_QUERY` | ephemeral | 16-byte lookup tag then 16-byte nonce |
| `0x0b` | `DESTINATION_CHALLENGE` | ephemeral | canonical challenge preimage |
| `0x0c` | `DESTINATION_PROOF` | ephemeral | PRPP version 1 |

Here, **durable** means that admission is governed by the inner object's own
authenticated lineage rather than by a pending correlation; it does not
require a particular storage medium or process lifetime. **Ephemeral** means
that admission is governed by bounded pending exchange state and conveys no
authority after that state is consumed, expires, or is cancelled.

Dispatch is exclusively by envelope version and kind. An implementation MUST
NOT dispatch by inner magic, payload length, a former service string, or an
implementation-local message number. Unknown versions, unknown kinds and
unregistered combinations are rejected without attempting another decoder.

## 4. Generation and correlation binding

The subject generation and correlation identifier are fixed per kind:

| Kinds | Subject generation | Correlation |
| --- | --- | --- |
| `IDENTITY_PUBLICATION` | PRPU publication-policy generation | all zero |
| `BLOOM_MANIFEST`, `REMOTE_BLOOM_VIEW`, `BLOOM_SHARD` | inner filter generation | all zero |
| `MICROSHARD_REQUEST`, `MICROSHARD_RESPONSE` | inner filter generation | inner query nonce |
| `VOPRF_KEY`, `VOPRF_REQUEST`, `VOPRF_RESPONSE` | inner discovery-control generation, promoted to u64 | zero for key; inner request ID otherwise |
| `EXACT_LOOKUP_QUERY` | selected filter generation | inner query nonce |
| `DESTINATION_CHALLENGE`, `DESTINATION_PROOF` | pending exact-query filter generation | pending query nonce |

For PRVM, PRVK, PRVQ and PRVR, the inner discovery-control generation MUST
also equal the envelope policy generation. Every duplicated inner value MUST
equal its outer value byte-for-value or numerically as specified. The exact
query, challenge and proof obtain their omitted profile and policy fields only
from the authenticated pending state selected by this envelope and lane; a
receiver MUST NOT guess among profiles. The exact-query tag is derived from
the full canonical target key under *PRP Exact and Oblivious Discovery
Lookup*. The target key is pending authenticated state, not a new field in
this envelope or its inner request.

Durable kinds require a zero correlation identifier. Every ephemeral kind
requires a nonzero identifier. A response is admissible only for an expected
pending request of the exact response kind and context. Thus a byte string
valid as one inner format cannot be substituted under another outer kind.

## 5. Per-kind admission

`IDENTITY_PUBLICATION` follows the authorization, lineage and tombstone rules
of *PRP Relationship Identity Publication*. Its complete PRPU bytes are the
inner payload. Only self-publication is currently available; delegated
publication remains unassigned.

`BLOOM_MANIFEST`, `REMOTE_BLOOM_VIEW`, `BLOOM_SHARD`,
`MICROSHARD_REQUEST`, and `MICROSHARD_RESPONSE` follow *PRP Sharded Bloom
Discovery Data*. An unsigned PRB4 signing prefix is not a payload. A shard is
admitted only against its authenticated manifest or remote view and remains
bounded independently; it is never split between envelopes.

`VOPRF_KEY`, `VOPRF_REQUEST`, `VOPRF_RESPONSE`, `EXACT_LOOKUP_QUERY`,
`DESTINATION_CHALLENGE`, and `DESTINATION_PROOF` follow *PRP Exact and
Oblivious Discovery Lookup*. A PRPP proof is accepted only for the pending
challenge selected by the envelope correlation and subject generation.

The role in the registry is a protocol role, not a process, host, service, or
deployment component. An endpoint MAY hold several roles, but a message is
admitted only in the direction allowed by its role and active relationship
policy.

## 6. State, replay and flow control

The lane's E2E item window and byte bounds apply before inner decoding.
Receivers MUST additionally bound durable objects, pending correlations,
cryptographic work, query rate and retained replay state per authenticated
relationship and direction. Rotation MUST NOT reset abuse accounting.

A byte-identical durable replay is idempotent and does not renew validity,
leases, quotas or rate limits. Mutation, succession and tombstones follow the
inner record specification. An ephemeral request creates at most one pending
entry for its correlation. A duplicate request is idempotent only where its
inner specification defines that result; it never renews expiry or creates
another downstream operation. A response consumes its pending entry on
success or definitive rejection. Consumed and expired correlations remain
tombstoned for the lane's bounded replay interval.

Session rekey and path replacement do not change durable discovery state or
permit correlation reuse. Loss or replacement of the directional discovery
policy cancels pending ephemeral state and rejects the old policy generation.
Relationship deletion removes state governed by that relationship, subject to
the durable record specifications' withdrawal and tombstone requirements.

There is no outer continuation. Flow-control acknowledgement concerns delivery
of an E2E item and does not imply semantic admission of its inner payload.

## 7. Unassigned messages

Exact terminal results (`UNAVAILABLE`, `RETRY_AFTER`, and
`STALE_GENERATION`), introduction material, cancellation, protected referrals,
private-query request or response bodies, and content descriptors, proofs or
bytes have no payload-kind value in this version. They MUST NOT be emitted
using a locally selected value. Content delivery belongs to an explicitly
selected content service; it is not a discovery terminal message. A later
assignment requires complete encoding, state, privacy, bounds and rejection
vectors; use in another project does not create an assignment.

## 8. Error behavior and conformance

A receiver rejects unknown version or kind, nonzero flags or reserved bytes,
zero generations, wrong policy or subject generation, forbidden zero or
nonzero correlation, length mismatch, oversized item, wrong sender role,
malformed inner payload, cross-kind substitution, unsolicited or duplicate
response, stale correlation, loss of policy and unbound-lane payload. Failure
MUST NOT select a fallback decoder or change admitted state.

`vectors/protected-discovery-payload-v1` fixes a positive composition for each
assigned kind and rejection behavior for envelope, context, dispatch and replay
boundaries.

## 9. IANA considerations

This document currently requests no IANA action.

## 10. References

### 10.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Wire Protocol Version 4*.
- *PRP Protected Discovery Lane*, version 1.
- *PRP Relationship Identity Publication*, version 1.
- *PRP Sharded Bloom Discovery Data*, version 4.
- *PRP Exact and Oblivious Discovery Lookup*, version 1.
- *PRP Suite-Bound Content Identity*, version 1.

### 10.2 Informative references

- *PRP Discovery Architecture*, version 1.
