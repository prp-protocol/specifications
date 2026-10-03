# PRP Relationship Identity Publication

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies publication and withdrawal of self-certifying PRP
identities at a rendezvous over an authenticated durable relationship. It
defines canonical records, authorization, generation and tombstone semantics,
accepted session suites, and the relationship to discovery indexes.

## 1. Status and requirements language

This document is a Working Draft. The PRPU payload and state transitions are
covered by deterministic vectors. Protected carriage is assigned as
`IDENTITY_PUBLICATION=0x01` by *PRP Protected Discovery Payload Envelope*,
version 1. Delegated publication is unavailable until `DISC-004` defines an
interoperable authorization declaration.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Scope

A publication declares that one identity is eligible for exact discovery
through one rendezvous relationship and states the session suites it accepts.
It does not grant identity, establish a session, prove current reachability,
or release a forwarding capability. Exact lookup and destination proof remain
mandatory.

The record repeats neither a public key nor the HS2 identity proof already
admitted for the relationship. A relationship may publish its own exact
identity. A different identity requires a separately authenticated delegation
covering that exact identity-suite and Strong-ID pair. No such delegation is
available in this revision while `DISC-004` is open.

## 3. Canonical record

PRPU uses unsigned big-endian integers:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | magic `PRPU` |
| 4 | 1 | version `0x01` |
| 5 | 1 | operation: `PUBLISH=0x01`, `WITHDRAW=0x02` |
| 6 | 2 | accepted session-suite count |
| 8 | 8 | publication-policy generation |
| 16 | 2 | published identity suite |
| 18 | 2 | published Strong-ID length |
| 20 | 2 | opaque rendezvous-policy length |
| 22 | 2 | reserved, zero |
| 24 | variable | Strong ID, session suites, then opaque policy |

Exact length is:

```text
24 + strong_id_length + 2*session_suite_count + policy_length
```

and, when carried by the protected lane, MUST NOT exceed 65,485 bytes. No
trailing bytes are permitted.

The identity suite is registered for the active wire version and fixes the
Strong-ID length. Every accepted session suite is registered for that wire
version. The suites are strictly increasing, unique, and therefore canonically
ordered.

A PUBLISH carries from one through eight session suites and from zero through
1,024 policy bytes. A WITHDRAW carries zero suites and zero policy bytes. The
Strong ID remains present in both operations.

Opaque policy bytes are generation-bound application data. This specification
assigns them no syntax or authorization meaning. A receiver MUST NOT infer a
policy language, executable instruction, selector, or capability from their
contents. Admission MAY reject unsupported nonempty policy but MUST preserve
the exact bytes when it accepts them.

## 4. Authorization

The authenticated durable relationship is the publication controller. Without
a delegated-publication specification, the published identity suite and
Strong ID MUST equal the controller identity established for that
relationship.

Possession of a Strong ID, a Bloom positive, a matching lookup token, or a
public key supplied inside an exact-lookup proof does not authorize
publication. Authorization is checked before mutation and again before a
publication is used for discovery.

Local relationship identifiers, session handles, route aliases, publication
references, service lanes, and forwarding capabilities never enter PRPU.

## 5. State key and generation

The conceptual state key is:

```text
durable_relationship || published_identity_suite || published_strong_id
```

`publication_policy_generation` is a nonzero u64 scoped to that key. It is
strictly increasing and never wraps.

For a current record:

- a byte-identical replay at the same generation is idempotent;
- different bytes at the same generation are a conflict;
- a lower generation is stale; and
- a greater authorized generation atomically replaces current state.

A WITHDRAW installs a tombstone at its generation. It is not deletion of the
generation history. An older PUBLISH cannot replace the tombstone. Deleting
the controlling durable relationship removes its publications and tombstones;
session replacement and rekey do not.

## 6. Index interaction

An active publication is eligible input to SHARDED_BLOOM_V4 and to the exact
lookup indexes selected for querying relationships. Withdrawal removes it
immediately from authoritative exact lookup even if an older Bloom view can
still return a false positive.

Bloom filter generation and discovery-control generation are independent of
publication-policy generation. Rebuilding a filter does not rewrite PRPU.
Changing a querying relationship's VOPRF key does not rewrite PRPU. Derived
index entries MUST always be traceable to an active authorized publication and
MUST become ineligible atomically with withdrawal or loss of authorization.

## 7. Accepted session suites

The suite list is a publication constraint, not a new session negotiation.
After successful exact lookup and destination proof, session establishment
still performs its own suite selection and proof. A requester MUST NOT infer
support for an omitted suite or treat list membership as admission.

A suite removed by a newer PUBLISH is no longer eligible for new discovery
results under that publication. Existing sessions follow their independent
lifecycle.

## 8. Error behavior

Unknown version or operation, zero generation, unknown suite, wrong Strong-ID
length, duplicate or unordered session suite, nonzero reserved bytes,
forbidden withdrawal content, excessive policy, inconsistent total length,
unauthorized identity, stale generation, and same-generation mutation fail
without changing publication, tombstone, Bloom, or exact-index state.

Failure MUST NOT cause fallback to unauthenticated or public publication.

## 9. Security and privacy considerations

PRPU exposes the published identity, accepted session suites, update timing,
and opaque-policy length to the authenticated rendezvous peer. Protected
carriage does not hide those values from that peer.

A compromised publication controller can publish only identities it is
authorized to control. The destination proof prevents a stale or malicious
rendezvous index from releasing authority based only on a publication record.

Tombstones prevent rollback after withdrawal. Receivers MUST retain sufficient
generation state for the lifetime of the durable relationship.

## 10. Conformance

`vectors/identity-publication-v1` fixes canonical PUBLISH and WITHDRAW bytes,
idempotence, replacement, stale update, conflict, tombstone, authorization,
and structural rejection cases.

Vector conformance does not close `DISC-004`.

## 11. IANA considerations

This document currently requests no IANA action.

## 12. References

### 12.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Wire Protocol Version 4*.
- *PRP Directional Discovery Policy*, version 1.
- *PRP Protected Discovery Payload Envelope*, version 1.
- *PRP Sharded Bloom Discovery Data*, version 4.

### 12.2 Informative references

- *PRP Architecture*, version 1.
