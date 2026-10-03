# PRP Bootstrap Candidates and Reachability Domains

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document defines implementation-independent semantics for physical and
protected reachability domains, opaque bootstrap candidates, candidate
authority, referral projection, requester-owned search bounds and promotion
through HS2. It also defines which candidate sources remain unavailable until
their authorization and privacy profiles are specified.

## 1. Status and requirements language

This document is a Working Draft. Deterministic vectors cover domain queries,
candidate admission, generation and lease checks, split horizon, cancellation,
HS2 promotion and unavailable referral and PIR sources.

Protected referrals cannot be emitted until `BOOT-001` defines referral
authorization and a later registry revision assigns their protected payload.
Rendezvous PIR cannot be offered until `DISC-002` registers a complete
private-query and payload-retrieval profile.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Scope and terminology

A **reachability domain** is a bounded set of possible next-step candidates
visible to one authorized principal. A **bootstrap candidate** is an opaque,
generation-bound reference that can be resolved to enough operational context
to attempt authentication. A candidate is not an identity or route.

This specification defines no API, database, storage layout, local handle
size, carrier interface, PIR provider interface or new PRP wire field. Local
candidate references and operational context never cross the protocol
boundary.

## 3. Domain kinds

### 3.1 Physical-carrier domain

A physical-carrier domain is reconstructed from current local carrier-member
evidence as defined by *PRP Directional Carrier Semantics*. Its authority is
`physical`. It can identify operational contexts in which public adjacent HS2
may be attempted.

Carrier enumeration supplies no PRP identity. A target identity associated
with a physical candidate is a local expectation or previously authenticated
policy, never a claim made by the carrier member. Public `ADVERTISE` is not a
source of physical candidates.

Physical candidates, member selectors, locators and pre-authentication context
are volatile. They MUST be reacquired after restart and MUST be checked
against their exact current member generation and local `TX` direction before
use.

### 3.2 Protected-relationship domain

A protected-relationship domain is derived from an authenticated durable
relationship, its active discovery selection and an admitted relationship-
bound discovery view. Its authority is `authenticated`. Its scope is `direct`
or `aggregate`.

The domain's generation, index profile, query profile, Bloom geometry,
commitment, validity and owner are the admitted relationship state. A Bloom or
microshard positive requires the selected protected exact lookup. It grants no
identity, forwarding or introduction authority.

Loss of the relationship, selected discovery generation or admitted view
invalidates the domain and every pending query derived from it.

## 4. Domain visibility and state

Each conceptual domain has at least:

```text
domain reference
authorized owner
kind
authority
scope
generation
validity
selected discovery context, when protected
```

Domain references are local and opaque. The owner scopes visibility and use.
A domain MUST NOT be shared with another principal merely because its carrier,
relationship, identity or discovery profile appears equal.

Generation is nonzero and immutable for one domain snapshot. Membership,
eligibility, ordered projection or authorization change creates a new
generation. Current and previous protected generations may coexist only when
their authenticated validity rules permit it.

## 5. Query and resolution outcomes

A target is an exact registered identity-suite and Strong-ID pair. A domain
query has one of these outcomes:

| Outcome | Meaning |
| --- | --- |
| `ABSENT` | the current authoritative test is definitely negative |
| `CANDIDATES` | one or more bounded opaque candidates are available |
| `EXACT_LOOKUP_REQUIRED` | a protected approximate test is positive |
| `UNAVAILABLE` | the selected required mechanism has no active binding |

For a protected domain, a definite Bloom or verified microshard negative is
`ABSENT`. A positive is `EXACT_LOOKUP_REQUIRED`; it MUST NOT be returned as a
candidate. Successful exact lookup and destination proof may produce
candidates only through a future assigned introduction payload. A false
positive becomes `ABSENT` without disclosing the candidate identity.

For a physical domain, resolution returns current operational candidates, not
an identity assertion. A candidate can be promoted only through successful
HS2 for the exact expected identity.

Queries and results are bounded by local policy. Truncation MUST be explicit;
it MUST NOT be represented as an exhaustive negative.

## 6. Candidate model

A candidate contains these conceptual properties:

```text
opaque local reference
nonzero candidate generation
expiration
source kind
capabilities
authorization scope
opaque local bootstrap context
```

The reference is unique only within its local owner and lifetime. It has no
portable size or encoding. Reference and generation MUST be matched together.
A stale, withdrawn, expired or owner-mismatched candidate fails closed.

The bootstrap context remains opaque to the requester. It may select a carrier
member, protected referral exchange or future private payload, but none of those
selectors becomes a PRP identity or is returned in a discovery record.

One identity may have several candidates. One physical member may be
associated by local policy with zero or more expected identities. Candidate
multiplicity MUST be preserved through exact results; Bloom membership alone
cannot represent it.

## 7. Candidate sources

### 7.1 Local carrier

`LOCAL_CARRIER` candidates arise from current physical carrier state. Their
maximum authorization scope is local use. They MUST NOT be projected to a
protected egress or rendezvous merely because HS2 later authenticates them.

Resolution requires current direction, readiness, evidence and member
generation. Promotion requires successful public adjacent HS2. A failed proof
does not rewrite the expected identity or try another identity implicitly.

### 7.2 Protected referral

`PROTECTED_REFERRAL` candidates would arise from an authenticated relationship
and an identity publication explicitly authorized for referral. Authorization
has three conceptual maxima:

| Scope | Maximum use |
| --- | --- |
| `LOCAL_ONLY` | only within the receiving relationship context |
| `REFERRABLE` | eligible protected relationship egresses |
| `RENDEZVOUS` | an authorized rendezvous projection |

Local policy MAY narrow but MUST NOT widen the authenticated maximum.

PRPU v1 assigns no syntax to its opaque policy bytes. Therefore those bytes
MUST NOT be interpreted as one of these authorization scopes. Until
`BOOT-001` defines an authenticated encoding, only `LOCAL_ONLY` is safe and no
protected referral may be emitted to another relationship. The protected
payload registry version 1 deliberately withholds referral and introduction
kinds.

### 7.3 Rendezvous PIR

`RENDEZVOUS_PIR` requires both private candidate selection and private
retrieval of fixed-cardinality, fixed-size bootstrap payload slots. Returning
an intersected column bitmap and then accepting selected column numbers or a
linkable token is forbidden.

No Spiral or YPIR profile is registered. MICROSHARD_V2 is partial disclosure,
not PIR. Consequently this source is `UNAVAILABLE` and MUST NOT be advertised,
queried or used for promotion under this version.

The candidate 120-byte `PRPM` matrix descriptor and provider framing from the
consolidation sources are not incorporated here. Without a registered
algorithm, threat model, padded result cardinality and private payload phase,
they do not form an interoperable protocol.

## 8. Projection and split horizon

An egress projection contains only active candidates authorized for that
egress. A candidate whose first protected hop is the same egress MUST NOT
contribute to that egress's exact result or approximate membership view.

This split-horizon rule is applied per exact eligible path. If at least one
other eligible path remains, the identity remains in the projected Bloom view
and exact index. Removing one of several paths does not remove membership.
Removing the last eligible path creates a new immutable projection generation
without that identity.

`LOCAL_ONLY` material may remain visible inside its authenticated ingress
context but cannot be exported through a different relationship.

## 9. Requester-owned search

The requester owns and enforces:

- maximum domain depth and total queried domains;
- parallel-query and pending-candidate limits;
- deadline, byte and computation budgets;
- a visited set containing domain and generation; and
- result cardinality and minimum local acceptance policy.

Topological distance is the number of already constructed onion stacks.
Selecting another domain increments that distance by exactly one. Bloom data,
referral metadata, candidate source and forwarding state MUST NOT be converted
into a claimed remote distance.

Intermediaries do not recursively execute the requester's search or reduce its
already accumulated depth.

## 10. Promotion and lifecycle

Resolving a candidate only exposes operational context for the selected next
authentication step. Successful HS2 proof for the expected identity is
required before creating authenticated relationship, session, path or
trajectory state.

Promotion atomically consumes or marks the exact reference and generation as
policy requires. Failure, cancellation, expiration, withdrawal, domain loss,
generation change, owner mismatch and commitment mismatch create no authority
and invalidate dependent pending work.

Restart discards physical-carrier candidates. Protected candidate retention,
if a future binding permits it, MUST remain subordinate to the authenticated
relationship, authorization and lease and MUST NOT retain operational
forwarding selectors as protocol state.

## 11. Error behavior

Unknown domain kind, zero generation, invalid identity suite or Strong-ID
length, stale snapshot, wrong owner, expired candidate, reference-generation
mismatch, unauthorized projection, split-horizon loop, unavailable source,
unregistered PIR profile, cancelled query and failed HS2 all fail without
fallback to a weaker source or disclosure of hidden selectors.

## 12. Security and privacy considerations

Opaque references prevent callers from depending on carrier locators or
forwarding handles, but opacity alone does not grant authority. References
must be unguessable where possession affects local work and always remain
owner-, generation- and lease-scoped.

Approximate membership leaks query timing and can produce false positives.
Exact lookup and destination proof remain mandatory. Split horizon limits
immediate referral loops but does not replace the requester's visited set.

PIR privacy requires payload retrieval as well as private index access. A
matrix-only mechanism can let a rendezvous correlate the subsequently chosen
column and construct targeted false publications; it is forbidden here.

## 13. Conformance

`vectors/bootstrap-reachability-v1` fixes domain-query, candidate, projection,
generation, cancellation, split-horizon and promotion outcomes.

Conformance does not close `BOOT-001` or `DISC-002` and does not
activate protected-referral or rendezvous-PIR candidate sources.

## 14. IANA considerations

This document currently requests no IANA action.

## 15. References

### 15.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Architecture*, version 1.
- *PRP Wire Protocol Version 4*.
- *PRP Directional Carrier Semantics*, version 1.
- *PRP Discovery Architecture*, version 1.
- *PRP Sharded Bloom Discovery Data*, version 4.
- *PRP Relationship Identity Publication*, version 1.
- *PRP Exact and Oblivious Discovery Lookup*, version 1.

### 15.2 Informative references

None.
