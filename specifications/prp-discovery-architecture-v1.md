# PRP Discovery Architecture Version 1

- **Version:** 1
- **Status:** Working Draft
- **Category:** Architecture and protocol boundaries
- **Date:** 2026

## 1. Scope

This document distinguishes discovery performed by a local carrier before
HS2 from authenticated discovery performed through an established
relationship. It defines carrier-domain correlation, resource-hint limits,
directional discovery policy, Bloom-view composition, suite-bound content
commitments and exact target validation.

This document does not modify the wire-v4 base envelope, AOP, RRL or HS2. New
carrier encodings and discovery messages outside the protected payload
registry require coordinated assignments before they can be emitted.

The former standalone discovery offer, selection and service-payload
formats are unassigned and removed. Implementations MUST NOT retain them as a
fallback or dispatch protected messages by magic or payload length.

## 2. Discovery domains

A `carrier-local` domain is operational enumeration before HS2. Its entries
are unauthenticated hints and grant no identity, forwarding, publication or
amplification authority.

A `protected-relationship` domain exists after HS2 and carries authenticated,
directional relationship policy. It is a logical reachability domain above
carriers. It is not a kernel carrier, KAPI provider or separate service
identity.

## 3. Carrier-local resources

Carrier enumeration MAY associate these independent resources with an
observed member generation:

```text
MEMBER
BOOTSTRAP_IDENTITY
WEAK_ALIAS_OFFER
TRANSIT_HINT
```

Presence implies none of the optional resources. Every optional publication
is explicit policy. A weak alias is an opaque 128-bit classification value and
MAY be announced without a strong ID. A bootstrap identity is an explicitly
local enrollment identity and is not a public rendezvous publication. A
transit hint expresses eligibility only.

Public carrier announcements MUST NOT be forwarded. An authorized aggregator
MAY instead issue a protected referral naming itself as introducer and using
an opaque token. It MUST NOT export the remote carrier locator or present the
remote member as directly attached. Relay policy is default-deny, directional,
bounded and explicit at every hop.

## 4. Carrier attachments and domain equivalence

An aggregator MAY expose one stable `carrier_attachment_id` for each local
carrier attachment. Different attachments owned by the same aggregator MUST
use different identifiers. The value is operational evidence and MUST NOT be
used as a PRP identity, public address, Bloom target or global fabric name.

The identifier is visible only through direct carrier-native enumeration and
MUST NOT be relayed. After adjacent HS2, its owner confirms the exact binding
to the authenticated relationship through a protected Relation Control
extension. Before confirmation, the observation is tentative.

The runtime forms a local `carrier_domain` equivalence class from directly
observed attachment sets. Authenticated overlap MAY join classes transitively.
Relayed, stale, unauthenticated or generation-mismatched evidence MUST NOT join
classes. Equal aggregator identity alone MUST NOT join classes because one
aggregator can attach to several carriers.

Unknown equivalence preserves separate paths but MUST NOT be counted as proven
carrier diversity. No permanent global carrier identifier is derived from a
changing member set. Domains without common direct evidence require a future
explicit federation mechanism.

### 4.1 Carrier attachment binding

`CARRIER_ATTACHMENT_BINDING` is optional Relation Control section type
`0x0005`, version `0x01`, operation `DECLARE`, with no section flags. It is
valid only in the authenticated adjacent relationship that owns the directly
observed attachment. Its body is exactly 24 octets:

| Offset | Width | Field |
| ---: | ---: | --- |
| 0 | 16 | opaque `carrier_attachment_id`, not all zero |
| 16 | 8 | attachment generation, nonzero unsigned big-endian |

The section contains no provenance assertion. The receiver accepts it only
when receiver-local direct carrier evidence has the exact identifier and
generation and belongs to that authenticated relationship epoch. Relayed,
stale, indirect, unauthenticated, differently owned or conflicting evidence
is rejected without changing existing state. An exact repeat is idempotent.
Loss of the relationship epoch or the observed attachment generation removes
the binding. Absence is valid and leaves correlation unknown.

The section instance is the authenticated relationship epoch and attachment
identifier. Relation-section `generation` and `base_generation` follow the
generic persistent lineage and are distinct from the attachment generation in
the body. The receiver installs its local correlation before returning
`RECEIVED`; that status confirms authenticated receipt and local evidence
matching, not creation of a relationship or a claim about another endpoint's
carrier. A structurally valid declaration with absent or mismatched local
evidence is `REJECTED`; stale section lineage is `STALE`.

The binding neither creates nor authenticates a relationship and grants no
identity, address, locator, publication, Bloom, forwarding or relay authority.

## 5. Protected discovery policy

Discovery is durable relationship capability negotiated independently in each
direction. Disabled is the default. The logical policy tuple is:

```text
discovery_index_algorithm
discovery_query_algorithm
discovery_profile
```

These selectors remain independent from the selected session and identity
suites. The non-critical `DISCOVERY_POLICY` Relation Control section defined
by *PRP Directional Discovery Policy*, version 1, proposes one exact tuple.
The receiver accepts it exactly or rejects it. A different tuple requires a
new proposal. There is no implicit counterproposal, fallback, downgrade or
upgrade. Crossed changes use that specification's deterministic state machine.

Rejection in one direction does not close the relationship unless another
policy requires discovery. Full discovery requires authenticated AOP. A
relationship using either registered `NONE` session suite MUST reject every
discovery-policy proposal, acceptance, restart and lane binding. Carrier
protection is not a substitute for authenticated PRP session protection.

## 6. Protected discovery lane

Each active direction has one runtime-internal ordered and reliable discovery
lane. It has no service ID, application-visible endpoint or separate identity.
*PRP Protected Discovery Lane*, version 1, binds that lane to the exact
authenticated E2E session, relationship, direction and accepted policy
generation before payload admission. Its messages use an explicit outer
version and kind registry.

Every message MUST fit one E2E item. Bloom data is shard-native; a logical
filter is a collection of independently authenticated bounded shards. No
generic discovery fragmentation is defined. Future PIR profiles MAY define
their own bounded transport after registration.

Durable objects retain their canonical internal versions and authentication.
Ephemeral messages are identified only by the lane registry. Version 1 assigns
queries, challenges and proofs but withholds terminal results, introduction
and cancellation until their complete bodies and state transitions exist.

## 7. Unified Bloom view

One directional Bloom view per discovery generation contains all admitted
target kinds, including published strong IDs, explicitly relayed bootstrap
strong IDs, weak aliases, strong aliases and content IDs. Target-kind, profile and
canonical-length domain separation occurs before lookup-tag and coordinate
derivation. Different input lengths do not require separate filters.

The exact index MAY retain a bounded candidate set for a shared weak alias or
for several providers of one content ID. Weak aliases and content IDs share
exactly one allocation-domain fair-share policy and common candidate and result
limits. Availability of a matching content ID asserts neither ownership nor
semantic meaning.

A participant MAY expose a bounded local content catalog to a lookup
deliberately directed to it without emitting one publication per file or
requiring a prior Bloom positive. Such lookup is local and MUST NOT recursively
fan out to adjacent relationships. Policy MAY contribute the catalog to the
ordinary Bloom aggregate; absence of that contribution leaves it discoverable
only by deliberate direction. Content targets MUST NOT create a parallel Bloom
filter whose density is compared separately.

Filter density remains the common requester-visible route-ranking signal;
less dense matching views are normally explored first. Resource-specific
quotas MUST NOT produce separate filters that make density incomparable.

## 8. Exact introduction

A Bloom positive is only an elimination result. The protected identity-target
flow is:

```text
requester -> rendezvous: LOOKUP_QUERY
rendezvous -> destination: CHALLENGE
destination -> rendezvous: PROOF plus introduction material
rendezvous -> requester: INTRODUCTION
```

The 128-bit correlation identifier is the sole query nonce. There is no
provisional positive response. Duplicate queries with the same correlation
identifier are idempotent and do not renew state or create another challenge.

For identity targets, the proof binds the rendezvous, discovery generation,
selected tuple, lookup tag, correlation identifier, challenge and concrete
published identity. Strong and weak aliases ultimately validate concrete
strong IDs. A content ID never signs that challenge: its provider supplies the
bounded PRCD descriptor through the selected content service, and the requester
validates the requested content ID and Merkle proofs before accepting bytes.
No forwarding or storage handle is released before target-specific validation.

The exact request remains `lookup_tag[16] || query_nonce[16]` under existing
protected kind `0x0a`. The target key is retained in authenticated pending
state and is absent from the request. No generic `ACCEPTED`, terminal result,
introduction or content payload kind is assigned here.

Conceptual terminal failures are `UNAVAILABLE`, `RETRY_AFTER` and
`STALE_GENERATION`; their protected signaling remains unassigned.
`UNAVAILABLE` intentionally hides false positives, missing records, denial,
destination state and proof failure. Local kernel and runtime handles never
cross a protected session.

## 9. Freshness and implementation boundaries

Carrier resources carry member generation, resource generation, refresh
sequence and relative validity. Duplicate refreshes do not renew leases;
higher resource generations replace lower generations; member-generation
changes invalidate associated resources. A referral cannot outlive its source.

Carrier providers own direct enumeration. The runtime owns equivalence
classes, policy, persistence, Bloom views, exact indexes, search budgets,
referrals and introductions. The kernel receives only concrete authenticated
sessions, paths and trajectories. Events are notifications; canonical
operational state is reconciled by GET/LIST.

All stages require bounded state, rates and lifetimes. Replay, stale evidence,
wrong relationship, wrong attachment binding, invalid proof or expiry fails
closed and creates no durable path.

## 10. Conformance requirements

Conformance evidence MUST cover direct versus relayed observation, one
aggregator on multiple carriers, overlapping authenticated attachment sets,
unknown path correlation, weak aliases without strong IDs, default-deny relay,
directional policy, unified Bloom density, bounded shard delivery, Bloom false
positives, all four canonical target kinds, common weak-alias/content fairness,
local non-recursive catalogs, availability-not-ownership, target-specific
validation, replay isolation, rekey, path replacement, restart and revocation.
Canonical carrier-attachment bytes
and local-evidence decisions are fixed by
`vectors/carrier-attachment-binding-v1/manifest.sha256`.
