# PRP Relationship Lifecycle and Continuous Control

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies the lifecycle semantics of a Participant Relationship
Protocol (PRP) relationship. It distinguishes a durable relationship from its
replaceable session generations, trajectories, paths, aliases, service lanes,
and application items. It defines continuity across rekey and path replacement,
directional availability, drain and closure, and version-1 continuous relation
control over PRP wire version 4.

This document does not prescribe storage, process, operating-system, or
administrative-interface architecture.

## 1. Status and requirements language

This document is a Working Draft. The session-generation, key-lifecycle, rekey,
relation-control, and drain semantics are consolidated from independently
consumed contracts. Route-alias pairing and scope-specific close selectors are
identified where their semantics still require independent interoperability
review. Semantic vectors accompany this document for key-policy intersection,
rekey scheduling, simultaneous proposals, generation overlap, and drain
transitions.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in this document are to be interpreted as described by BCP 14 when,
and only when, they appear in all capitals.

## 2. Scope

This document defines:

- relationship and relationship-epoch lifetimes;
- directional operational availability;
- session generations and rekey continuity;
- trajectory and path replacement;
- continuous relation-control transactions;
- key-lifecycle and rekey-schedule sections;
- planned drain behavior;
- route-alias lifecycle invariants;
- session-scope closure; and
- failure and restart requirements.

This document does not define:

- relationship discovery or publication;
- authorization policy content;
- carrier-specific paths or locators;
- application APIs;
- retained-storage mechanisms;
- resource-report bodies; or
- deletion procedures for local administrative records.

## 3. Lifecycle object model

### 3.1 Relationship

A **relationship** is the durable communication and authorization context
defined by the PRP Architecture. Its identity constraints, trust material,
authorization boundaries, and negotiated durable policy may survive the loss
or erasure of all active sessions and paths.

A relationship is not created by a route, carrier attachment, route alias,
session handle, or application open operation. Closing any one of those objects
does not by itself delete the relationship.

### 3.2 Relationship epoch

A **relationship epoch** is one uninterrupted incarnation of a relationship's
continuity and authorization context. A change that cannot prove continuity of
the authenticated participants or that replaces the authorization context
creates a new relationship epoch.

A local durable relationship identifier MAY identify an epoch, but such an
identifier is not a wire value and has no meaning at another participant unless
an explicit specification gives it one.

### 3.3 Session and session generation

A **session** is replaceable cryptographic and operational state within one
scope of a relationship. A **session generation** identifies one nonreusable
set of traffic keys, nonce spaces, replay windows, and associated transcript
state.

The session generation is selected by HS2 M2. It is distinct from a
relationship epoch, relation-section generation, route-alias generation,
connection generation, and trajectory generation.

### 3.4 Trajectory and path

A **trajectory** is a validated realization of reachability for a relationship
direction. A trajectory contains one or more **paths**. A path is an
authenticated directional opportunity to carry traffic; it is not a
relationship or identity.

Multiple paths and trajectories MAY be simultaneously eligible. Their local
selectors and carrier locators are volatile and do not enter the durable
relationship identity.

### 3.5 Route and route alias

A **route** is a scoped directional forwarding decision used in the
realization of a trajectory. It selects an eligible next path, adjacency, or
local-delivery outcome and is not itself a trajectory.

A **route alias** is a receiver-interpreted 16-bit selector scoped to one
adjacent session and its active alias generation. It selects installed local
route state and is never a participant, service, relationship identity, path,
or trajectory.

### 3.6 Service lane and item

A **service lane** is a delivery context for one authorized service within a
relationship. An **item** is one independently identifiable application unit.
Service-lane, ordering, replay, and item identity can outlive a trajectory
change or session rekey when their specification requires continuity.

Transport acknowledgment, retained-copy acknowledgment, and application
acceptance are distinct events and MUST NOT be conflated.

## 4. Independent lifetimes

The following invariants apply:

- relationship policy MAY survive without active key material;
- a session generation MAY be replaced without replacing the relationship;
- a trajectory MAY be replaced without replacing the session generation;
- a path MAY disappear while paths in the other direction remain valid;
- a route alias expires or is revoked independently of participant identity;
- an application endpoint may detach without closing the relationship; and
- deletion or revocation of the relationship invalidates every object whose
  authority derives exclusively from it.

An implementation MUST NOT claim continuity by silently resetting a generation,
nonce, replay window, ordered item identifier, or authenticated peer binding.

## 5. Conceptual relationship states

The states in this section describe interoperable semantics and are not wire
enumerations.

| State | Meaning |
| --- | --- |
| retained | Durable relationship policy exists; no active session is required |
| establishing | HS2 or an authenticated recursive establishment is in progress |
| active | At least one policy-required direction has an authenticated usable path |
| partially available | Some admitted directions remain active and others do not |
| rekeying | A newer session generation is being established while continuity is retained |
| draining | New work is restricted while accepted work migrates, completes, or fails |
| dormant | No active session or useful path exists, but the relationship remains retained |
| revoked | Relationship authorization no longer permits communication |

State names are local observations; peers need not transition at the same
instant. Wire acknowledgments confirm only the event explicitly defined by the
corresponding control.

Successful HS2 can move a retained or dormant relationship to active. Idle key
erasure or loss of every useful path can move it back to dormant without
deleting it. A later demand performs full HS2 with a newer session generation.

Revocation is terminal for the affected relationship epoch. Re-establishing
communication after revocation requires explicit authorization and MUST NOT
reuse the revoked epoch's traffic state.

## 6. Directional availability

Direction is always expressed from the observing participant:

- **TX** means the participant can transmit toward the remote participant;
- **RX** means the participant can receive from the remote participant; and
- **TX_RX** means both properties exist, not necessarily simultaneously.

Carrier capability, configured endpoint intent, operational readiness, and
authenticated relationship direction are separate facts. None implies the
others.

Receiving HS2 M1 on an RX path does not authorize use of that path in the
opposite direction. The responder uses an independently authorized TX path for
M2. A path MAY use the same carrier in both directions, but reverse
reachability MUST NOT be inferred from locator equality or ingress evidence.

A bilateral relationship is operational only while it has at least one
authenticated TX path and one authenticated RX path. A unidirectional
relationship is valid only when relationship and service policy explicitly
admit that direction.

Loss of the last TX path makes TX unavailable while valid RX state is
preserved, and conversely. The relationship closes only when policy requires
bilateral availability or no useful admitted direction remains.

## 7. Trajectory replacement

A replacement path MUST be installed, authenticated, and validated for its own
direction before it becomes eligible. Make-before-break replacement proceeds:

1. acquire and authenticate the candidate path;
2. bind it to the existing relationship, direction, and permitted session
   generation;
3. validate its ability to carry the required units;
4. make it eligible for new transmissions;
5. migrate or complete eligible in-flight work; and
6. drain and remove the old path.

Connection aliases, service lanes, ordering, replay, and item identities MUST
remain unchanged across the handover when continuity is claimed. If continuity
cannot be proven, the affected operation fails or establishes a new
relationship epoch; it does not reset those identifiers in place.

Feedback and retransmission may use any authenticated reverse path authorized
for their direction. They do not require the same carrier or trajectory used by
the corresponding data.

## 8. Session generation and rekey

Every installed session generation is nonzero. A newer generation is strictly
greater than every generation previously installed for that session scope. A
generation MUST NOT be reused, resumed after nonce or replay-state loss, or
installed with old key-establishment material.

Rekey uses a complete canonical HS2 M1/M2 exchange with fresh key-establishment
material. Relation control may schedule rekey but cannot install keys. M2 is
the sole wire authority for the new generation.

The responder installs new receive state before emitting M2. The initiator
installs the new state only after authenticating M2. New-generation TX begins
after the local M2 transition succeeds, at sequence one.

Old-generation TX stops at cutover. Old-generation RX MAY remain for a bounded
negotiated overlap to admit in-flight units. It MUST be erased when that overlap
ends. Rejection of the new generation MUST NOT roll state back or extend the
old generation beyond policy.

For conformance, local cutover time is the successful role-specific M2
transition defined above. The overlap interval is half-open: it begins at
cutover and ends at `cutover + overlap`. At the ending instant, old-generation
RX is no longer admissible and its traffic state is erased. A zero overlap
therefore admits only the new generation at cutover.

Ordering, replay, retained items, and service-lane continuity are preserved
across rekey only when their state is explicitly scoped above the replaced
traffic-key generation. Otherwise the affected operation fails rather than
claiming transparent continuity.

Abbreviated resumption and early data are not defined. After erasure, a full
HS2 exchange is required.

## 9. Continuous relation control

### 9.1 Protection and scope

Continuous relation control carries persistent relationship policy and
explicitly registered authenticated relationship operations. It MUST be
carried inside an already authenticated and integrity-protected relationship
scope. Wire-v4 control kind `0x08` MUST NOT be sent or accepted through a
`NONE` session suite.

Relation control is not a public-bootstrap message and carries no identity,
carrier locator, topology, application credential, or local implementation
handle. Scope and route alias select an already authenticated adjacent, direct,
or E2E relationship namespace.

### 9.2 Header

The relation-control header is 20 bytes:

| Offset | Bytes | Field | Rule |
| ---: | ---: | --- | --- |
| 0 | 1 | control kind | `0x08` |
| 1 | 1 | version | `0x01` |
| 2 | 1 | operation | Section 9.3 |
| 3 | 1 | flags | zero |
| 4 | 1 | session scope | adjacent, direct, or E2E |
| 5 | 1 | reserved | zero |
| 6 | 2 | route alias | `0xffff` for adjacent; nonreserved for direct or E2E |
| 8 | 8 | transaction ID | opaque and nonzero |
| 16 | 2 | section count | `1..16` |
| 18 | 2 | reserved | zero |

The complete control is at most 1024 bytes. Sections are strictly ordered by
ascending nonzero type and occur at most once.

### 9.3 Operations

| Value | Operation | Semantics |
| ---: | --- | --- |
| `0x01` | DECLARE | Unilateral authenticated declaration; RESULT confirms receipt, not agreement |
| `0x02` | PROPOSE | Negotiated update with no effect before explicit acceptance |
| `0x03` | CONDITION | Negotiated continuity requirement whose refusal may cause degradation or closure |
| `0x04` | RESULT | Explicit per-section response to the referenced transaction |

Silence, loss, timeout, or restart never implies acceptance.

The transaction identity is the tuple of relationship epoch, session scope,
route alias, and transaction ID. An exact retransmission is idempotent and
receives the cached result. Reuse of that identity with different canonical
bytes is a conflict and MUST NOT mutate state.

Peers MAY initiate transactions simultaneously. Each negotiated section must
define a deterministic commutative intersection or another explicit conflict
rule before simultaneous proposals can be installed.

### 9.4 Section header and generations

Each section begins with:

| Offset | Bytes | Field | Rule |
| ---: | ---: | --- | --- |
| 0 | 2 | type | nonzero |
| 2 | 1 | version | nonzero |
| 3 | 1 | flags | `CRITICAL=0x01`; other bits zero |
| 4 | 1 | status | zero in requests; nonzero in results |
| 5 | 1 | reserved | zero |
| 6 | 2 | body length | exact following bytes |
| 8 | 4 | generation | nonzero |
| 12 | 4 | base generation | less than generation |

An initial section uses base generation zero. A later update names the current
effective generation as its base and uses a strictly greater generation. An
exact repeat is idempotent. Rollback, a different body at the same generation,
or a stale base MUST NOT mutate state.

A structurally invalid generation lineage produces MALFORMED. A proposal whose
base does not equal the current effective generation, or whose generation is
older than the current generation, produces STALE with the current effective
state. Reuse of the current generation with different canonical bytes produces
REJECTED. These rules do not replace the cached result for an exact
retransmission.

Section generations advance independently. Acceptance is atomic per section,
not implicitly across every section in the control. A section specification MAY
define independent instance lineages within one type when its canonical body
contains an explicit instance key. Resource-report policy type `0x0003` uses
request originator and direction as specified by PRP Protected Resource
Reporting; absence of such a rule never permits an inferred local subkey.

A registered section MAY instead define a transaction-scoped, nonpersistent
operation. Such a section MUST define its instance key, meaning of generation,
required base generation, status transitions, replay behavior and authority
lifetime. Its generation is not thereby installed as persistent relationship
policy. Generic persistent-lineage behavior MUST NOT be inferred for it.

### 9.5 Results

Result status values are:

| Value | Status |
| ---: | --- |
| `0x01` | RECEIVED |
| `0x02` | ACCEPTED |
| `0x03` | DEFERRED |
| `0x04` | REJECTED |
| `0x05` | STALE |
| `0x06` | UNSUPPORTED |
| `0x07` | MALFORMED |

For a known persistent negotiated section, ACCEPTED carries the complete
canonical effective body. STALE carries the complete current effective body
and its current generation lineage. Other statuses carry no body.

A registered transaction-scoped section MAY assign a status-specific result
body. That body is authority only for the operation and lifetime defined by
that section; it is not an installed effective policy body. A generic relation
receiver MUST NOT reinterpret it as persistent state.

A participant installs an accepted section only after confirming exact type,
version, generation, base generation, length, and effective bytes. An empty or
different ACCEPTED body is not agreement.

An unknown optional section produces UNSUPPORTED and does not affect other
sections. An unknown critical section produces UNSUPPORTED and prevents the
transaction from taking effect. A known malformed section produces MALFORMED.

## 10. Key-lifecycle policy

Section type `0x0001`, version `0x01`, is used with PROPOSE or CONDITION. Its
body is 40 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 8 | maximum active key age, milliseconds |
| 8 | 8 | maximum encrypted bytes per generation |
| 16 | 8 | maximum encrypted items per generation |
| 24 | 8 | idle key retention, milliseconds |
| 32 | 4 | old-generation overlap, milliseconds |
| 36 | 4 | reserved, zero |

Maximum active age and idle retention are nonzero. Zero disables the encrypted
byte or item threshold; it has no other meaning.

The effective policy is the commutative field-by-field intersection:

- active age and idle retention use the smaller nonzero value;
- byte and item thresholds use the smaller nonzero value, treating zero as
  infinity unless both are zero; and
- overlap uses the smaller numeric value.

Local policy MAY trigger earlier action but MUST NOT extend a negotiated
maximum.

Only successfully authenticated cryptographic TX or RX refreshes activity.
Administrative observation, failed authentication, and unauthenticated input
do not. Reaching the first enabled age, byte, or item limit blocks new
application TX for that generation while allowing the protected control needed
to rekey, drain, or close.

Idle expiry erases traffic keys, nonce state, replay state, and overlap
generations, but preserves the durable relationship and negotiated policy.
Later traffic requires full HS2 with a newer generation.

## 11. Rekey schedule

Section type `0x0002`, version `0x01`, negotiates one rekey attempt. It has a
32-byte prefix followed by `suite_count` session-suite identifiers:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 4 | current session generation |
| 4 | 2 | reason |
| 6 | 2 | suite count, `1..16` |
| 8 | 8 | earliest-start delay from authenticated receipt, milliseconds |
| 16 | 8 | deadline delay from authenticated receipt, milliseconds |
| 24 | 4 | requested overlap, milliseconds |
| 28 | 4 | reserved, zero |
| 32 | 2 each | acceptable session suites in strictly ascending order |

Reason values are `AGE=0x0001`, `BYTES=0x0002`, `ITEMS=0x0003`,
`ADMINISTRATIVE=0x0004`, `COMPROMISE=0x0005`, and
`SUITE_MIGRATION=0x0006`.

The current generation is nonzero and must equal the active generation. The
deadline is nonzero and no earlier than the start delay. Requested overlap MUST
NOT exceed the effective key-lifecycle overlap. Every suite is registered,
unique, and ordered.

Delays are measured from local monotonic authenticated receipt time; no civil
clock synchronization is required. Acceptance authorizes an attempt within the
window, not key activation. The exact suite used by subsequent HS2 must be in
the accepted set.

Loss of session state, idle erasure, or suspected compromise MAY require full
HS2 without a prior schedule. That is reestablishment rather than coordinated
rekey.

## 12. Drain policy

Section type `0x0004`, version `0x01`, is valid only in an authenticated
DECLARE. Its 12-byte body is:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 1 | drain scope |
| 1 | 1 | flags, zero |
| 2 | 2 | reason |
| 4 | 4 | grace period, milliseconds |
| 8 | 4 | final timeout, milliseconds |

Drain scopes are `CANCEL=0`, `TRANSIT_ONLY=1`,
`TRANSIT_AND_LOCAL=2`, and `ALL=3`.

CANCEL requires every other field to be zero. An active drain has a nonzero
grace period and a final timeout not shorter than that period. Durations begin
at local monotonic authenticated receipt time.

TRANSIT_ONLY immediately stops new transit, fanout, and route allocations while
permitting authorized local and administrative services. TRANSIT_AND_LOCAL also
stops new nonadministrative local work. ALL additionally drains administrative
service use permitted to be drained by local policy. Administrative status is
derived from authenticated service identity and local policy; it is not a
peer-asserted flag.

During grace, already accepted work may complete, migrate, or produce an
explicit failure. At final timeout, remaining targeted work is closed or fails
according to its delivery contract. It MUST NOT be silently reported as
delivered.

A RESULT confirms receipt of the declaration, not successful migration or
agreement. A newer drain-section generation MAY cancel future drain action but
cannot restore state already closed by either participant. After an accepted
cancellation, the drain no longer prohibits new admission; the relationship's
other policies still apply. Work closed before authenticated receipt of the
cancellation remains closed.

## 13. Route-alias lifecycle

Wire-v4 alias lifecycle has the 12-byte body defined by the wire specification:

```text
operation:u8 || flags:u8=0 || route_alias:u16 ||
generation:u32 || lease_seconds:u32
```

Operations are `OFFER=0x01`, `ACCEPT=0x02`, `RENEW=0x03`, and
`REVOKE=0x04`. Alias and generation are nonzero; alias `0xffff` is forbidden.
OFFER, ACCEPT, and RENEW carry a nonzero lease. REVOKE carries a zero lease.

An alias is not active merely because it was offered. Activation requires an
authenticated acceptance of the exact alias, generation, and lease. RENEW
proposes a strictly newer generation for an active alias and likewise requires
authenticated acceptance. Exact retransmissions are idempotent; different
bytes for the same alias and generation fail closed.

Lease duration begins at local authenticated acceptance time. Expiry or REVOKE
makes that alias generation unusable for new traffic. Receivers retain bounded
tombstone state sufficient to reject delayed traffic and stale lifecycle
messages. A newer alias generation cannot revive data from an expired or
revoked generation.

The OFFER/ACCEPT pairing and simultaneous-renewal rules remain under
interoperability review in this Working Draft. Implementations MUST NOT claim
general lifecycle conformance based only on structural parsing of these
records.

## 14. Scope closure

Wire-v4 close and close acknowledgment have this 12-byte body:

```text
scope:u8 || mode_or_status:u8 || reason_or_reserved:u16 ||
generation:u32 || selector:u32
```

Scope, generation, and selector are nonzero and refer to existing authenticated
state. Close modes are `GRACEFUL=0x01` and `HARD=0x02`. A close acknowledgment
uses status `CLOSED=0x01`, `STALE=0x02`, or `REFUSED=0x03` and encodes the
reason field as zero.

Graceful close stops new admission in the selected scope and permits bounded
accepted work to complete before acknowledgment. Hard close stops admission
immediately and explicitly fails any accepted work that cannot be preserved.
Neither mode implies deletion of the durable relationship.

An acknowledgment reports only the selected scope and generation. It is not an
application-delivery acknowledgment and does not erase an origin's independent
retained item.

The meaning of the 32-bit selector for every nonadjacent scope is not fully
closed by this Working Draft. A profile MUST define that meaning before emitting
scope close for that profile; a local implementation handle MUST NOT be placed
on the wire by inference.

## 15. Retention and in-flight work

Retention is replicated recovery state, not exclusive custody. Adjacent
acknowledgment states only that an adjacent participant has the state promised
by its selected profile. It does not mean destination application acceptance
and does not authorize the origin to release a reliable item.

A retained object is scoped by relationship epoch, service, direction, logical
generation, and stable item identity. It MUST NOT be identified by a volatile
path, route alias, carrier locator, or local session handle.

Rekey or trajectory handover preserves retained work only when the object
remains cryptographically acceptable and can be reprotected without nonce
reuse. Otherwise the loss is an explicit terminal or retryable outcome under
the selected retention profile.

Once bounded admission succeeds, ordinary pressure MUST NOT silently discard
the object before its accepted expiry. Backpressure or refusal occurs before
admission when the obligation cannot be met.

## 16. Failure, restart, and reconciliation

Durable state consists of relationship intent, authorization, negotiated
policy, section generations, and any profile-defined retained checkpoints.
Volatile path selectors, carrier locators, nonce state, replay windows, and
traffic keys MUST NOT be reconstructed by guessing after loss.

Loss of traffic sequence or replay state invalidates that session generation.
Recovery uses full HS2 and a strictly newer generation. Restart MUST NOT restore
an old traffic generation merely because durable relationship policy remains.

Observed carrier members and paths are reacquired and reauthenticated. Equality
of a locator before and after restart does not prove continuity.

Incomplete transactions remain unaccepted unless a peer returns a canonical
cached result. Reconciliation accepts exact idempotent state, rejects rollback
or conflicting bytes, and preserves independently advanced section
generations.

## 17. Security and privacy considerations

Lifecycle controls are authorization-sensitive. Integrity without an
authenticated relationship binding is insufficient. This is why continuous
relation control is forbidden through a `NONE` session suite even though that
suite can structurally encode other PRP records.

Generation monotonicity prevents restored snapshots, stale paths, delayed alias
messages, and replayed transactions from reviving obsolete authority. Durable
reservation of session generations is required before use, but this document
does not prescribe how it is stored.

Make-before-break limits avoidable interruption but does not itself prove
continuity. The replacement path must bind the same authenticated participants,
relationship epoch, direction, and permitted session state.

Drain and close can be used for denial of service if accepted from an
unauthenticated source. Implementations MUST authenticate them before mutation,
bound transaction and tombstone state, rate-limit expensive work, and avoid
amplifying errors to unauthenticated peers.

Lifecycle metadata can reveal activity, maintenance, and path-change timing.
Profiles concerned with correlation SHOULD minimize unnecessary declarations
and avoid exposing civil time or topology in lifecycle controls.

## 18. Conformance

Wire-level conformance for relation-control framing, drain records, alias
lifecycle records, and scope-close records uses the applicable cases in
`vectors/wire-v4/codec-vectors-v4.tsv`. Passing those vectors demonstrates
canonical encoding and rejection behavior only; it does not by itself
demonstrate the lifecycle state transitions defined by this document.

A claim of conformance to Sections 8 through 12 requires passing every
applicable case in `vectors/relationship-lifecycle-v1`. The vector outcomes are
derived from protocol-visible values and are not additional wire status values.
Complete relationship-lifecycle conformance additionally requires the
route-alias and scope-close semantics that remain open as REL-001 and REL-002;
until those issues are resolved, conformance claims MUST identify the specific
sections covered.

## 19. IANA considerations

This document currently requests no IANA action.

## 20. References

### 20.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Architecture*, version 1.
- *PRP Wire Protocol Version 4*.
- *PRP Protected Resource Reporting*, version 1.

### 20.2 Informative references

- *PRP Relationship Retention*, version 1.
- *PRP Directional Carrier Semantics*, version 1.
