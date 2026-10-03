# PRP reference classes and authorized representation — successor decision

- Version: 2, revision 18, 2026-09-22
- Status: Working Draft; user-approved semantic direction, wire admission pending
- Baseline: prp-spec `02cedd9eb1a01b8ea32e620aa500c73bcbd0a239`

## 1. Authority and compatibility boundary

This document formalizes the decisions reached during the DNS/identifier review.
It defines the intended successor semantics, not permission to reinterpret
wire-v4, discovery-target-v1 or HS2 bytes. Uppercase requirement words follow
the repository's BCP 14 convention. Numeric class assignments, successor
discovery domains and authenticated representation carriage remain gated by
REF-001 in OPEN-ISSUES.md. No placeholder wire values are allocated here.

The published architecture -00 snapshot and existing registries/vectors remain
unchanged. Earlier alternatives discussed in the review are superseded by this
decision: weak alias as variable text; a separate strong-alias reference class;
principal/representative pair as an identifier; and expiry checked only at
connection opening. DNS revision9 now carries the complete explicit reference
by subsequent user decision; earlier IDENTITY-only registration preparation is
historical. The earlier class-less DNS revision6 is historical, not current RDATA.
Unadmitted representation remains unavailable regardless of DNS publication.

## 2. Four SCIR classes and the earlier REF-001 baseline

The original REF-001 work admitted IDENTITY and ALIAS first. Subsequent user
decisions admitted MULTICAST and GROUPCAST as semantic reference classes and
generalized SCIR to mean Self-Certifying Identifier Reference. The normative
envelope and formation boundary are in
`prp-self-certifying-identity-reference-v1.md`; this document preserves the
representation and discovery decisions without redefining those bytes.

| Class    | Logical encoding after class:u8                    | Meaning |
| :------- | :------------------------------------------------- | :------ |
| IDENTITY | reference_suite:u16 + strong_id                     | Expected principal, served directly or by an authorized representative |
| ALIAS | reference_suite:u16 + alias_identifier | Global canonical-text rendezvous, potentially with multiple discovery results |
| MULTICAST | reference_suite:u16 + multicast_identifier | Stable logical unidirectional transmission |
| GROUPCAST | reference_suite:u16 + groupcast_identifier | Autonomous group with separately authenticated mutable state |

Class names above are symbolic, not assigned numbers. Integers are unsigned
big-endian. No additional identifier-length or format-version field is intended
inside these references. The surrounding protocol must select the successor
format unambiguously; absence of an inner version is not permission to guess.

For IDENTITY, Strong-ID formation and suite semantics are unchanged. For ALIAS,
the suite supplies its hash and output length over the globally canonicalized
text; it supplies no key or proof-of-possession operation. MULTICAST and
GROUPCAST use the same suite hash/length over their admitted immutable genesis.
Existing suite values are reused, not new application or geographic suites.
Current lengths are 32 bytes for
0001/0002/0003/0005 and 48 for 0004. The suite is not the session suite; session
negotiation/rekey does not change a reference. Suites with equal lengths still
produce distinct references because the suite field participates in equality.

All four classes have reference lengths of 35/51 bytes, including class, for the
current 32/48-byte suites. These are logical sizes, not currently admitted
target bytes. There is no CONTENT class or content-profile field in this
successor. This does not reinterpret or remove legacy CONTENT_ID assignments.

Unknown classes/suites/profiles, incorrect sizes and trailing fields MUST NOT
be admitted or inferred from length. A stream parser cannot skip an unknown
reference merely by guessing its length; it needs bounded outer framing or
must reject that enclosing unit. Removing a length field does not remove
validation of available bytes against the selected class/suite/profile.

Aliases carry no intrinsic claim of exclusive ownership, membership or identity.
PRP defines canonical-text formation so independently authored applications can
meet at the same ALIAS, but it does not assign meaning to that text. Geographic,
activity, category, prefix and composite conventions are ordinary alias-text
conventions rather than application-profile fields. Formation remains distinct
from authenticated declaration admission and from internal discovery indexes.

The existing common bounded multi-result policy remains an ALIAS successor
requirement, including allocation-domain fairness and 4096/32 limits. Aliases
used for content receive no separate fairness class, quota or lookup mechanism.
Fixed-size input bounds per-query work; it does not guarantee constant wall-clock
latency or remove query-rate, candidate, density and memory limits. Removing
identifier_length from a hashed target key changes its preimage and therefore
requires explicit coordinated discovery cutover.

### 2.1 External content semantics and generic catalogs

External standards/applications define content identities, canonical names,
hash algorithms, descriptors, Merkle constructions and proofs, and any mapping
to canonical ALIAS text. PRP forms the ALIAS SCIR from that text but neither
interprets its application meaning nor validates content commitments. No particular external namespace,
content suite, naming registry or application is mandatory for PRP conformance.
An application MAY place an externally defined digest in alias text; that does
not alter the normative ALIAS preimage. Content, reference and session suites
are separate choices and MUST NOT be silently substituted for each other.

Candidate authentication and authenticated alias association do not prove
possession, availability, authorship, ownership or redistribution rights for
content. Content verification and partial-proof/transfer interoperability belong
to the selected application contract. A discovery match MUST NOT substitute
for those checks or authorize release of content bytes.

The successor MUST preserve bounded directed local-catalog lookup for opaque
aliases without requiring per-object network publication or a prior Bloom
positive. Such a lookup MUST NOT recursively fan out to adjacent peers. A local
catalog MAY contribute keys to the ordinary Bloom aggregate, subject to common
density, admission and rate limits; no content-specific filter is added.
Anyone may declare membership in an ALIAS. Legitimacy of that membership and
resulting service authorization belong to the service, not PRP. Catalog resource
controls and message integrity remain distinct from certification of membership.
Candidate authentication and exact query/result/selected-identity binding remain
required; their final bytes must be coordinated under REF-001. Local indexing
alone proves neither remote identity nor the truth of a membership claim.

Old content IDs MUST NOT be cast, truncated, padded or automatically treated as
successor ALIAS identifiers. Any mapping to canonical ALIAS text requires its own explicit
contract. Neither dual publication nor fallback/cross-query is implicit. Legacy
PRCD and binary proofs remain evidence for the old format only: this decision
does not claim that an external standard already supplies equivalent partial
verification. Legacy registries/vectors remain intact until an explicit
coordinated migration; their numeric values MUST NOT be recycled.

### 2.2 Alias discovery without redundant identity Bloom entries

Subsequent user-approved refinement (wire admission still pending): an ALIAS query
MUST explicitly identify the requested identity suite. Catalog results MUST be
limited to that suite, and the consumer MUST verify the original filter against
the candidate before selection/admission. Equal identifier lengths do not imply
suite compatibility; no cross-universe fallback is permitted. The requested
identity suite is not the negotiated session-protection suite. The complete
reference already contains a suite field; final query encoding must reconcile
that field without an inferred duplicate selector. See
`../docs/ref-001-alias-suite-filter-decision.md` for bindings and finite negatives.
This refinement does not approve a catalog trust anchor or portable receipt.

Discovery through ALIAS MUST NOT require a redundant IDENTITY Bloom entry for
each candidate. The bounded exact result must distinguish selectable candidates
and provide the admitted forwarding context needed to reach the chosen one
without a second Bloom search for its identity. Multiple candidates may share
the same alias and carrier; neither the alias nor carrier alone selects a peer.

The selected principal identity and candidate/forwarding context MUST be bound
to the initiating request and authenticated HS2 outcome. The peer proves that
principal directly or proves its concrete representative identity together with
the required principal authorization. The authenticated identity MUST equal the
complete principal declared in the selected ALIAS result, not merely another
identity using the requested suite. Discovery validation here means message
integrity, query correlation, suite compatibility and frozen candidate/context
binding; it MUST NOT require a catalog or external issuer to certify legitimate
ALIAS membership. HS2 proves identity and this binding, not membership truth or
service access. Self-declaration never permits impersonation of another principal
or substitution for valid representation proof. See
`../docs/ref-001-nc02-relationship-authority-proposal.md` for the controlling
user decision and exact in-scope handoff requirements.
Replacing the candidate, context or intended identity MUST NOT silently satisfy
the original request. Stale or unavailable context requires rejection or a new
explicit selection, not an unbound fallback. A discovery result grants no
forwarding or application authority beyond the admitted mechanism.

An IDENTITY Bloom entry MAY still be advertised for direct identity discovery.
Its absence does not guarantee direct discoverability along that path and does
not waive identity proof, relationship state, policy or forwarding validation.
Existing valid relationships/forwarding contexts may provide reachability
without Bloom discovery. Exact result encoding, context lifecycle, selection
binding and HS2 transcript/KDF details remain coordinated admission work; no
new result kind, field or carrier is assigned by this requirement.

## 3. Identity is independent of who serves it

Strong alias is no longer a reference class in the successor. Its intended
function becomes authorized representation. An identity reference always names
the expected principal. Direct service proves that principal's identity;
represented service proves a distinct representative identity plus the
principal's signed authorization. No shared controller private key is required.
Multiple representatives may simultaneously serve the same stable reference.
This permits decentralized candidate selection, not guaranteed load balancing.

Principal and representative identity suites may differ. Both must satisfy the
client's policy; declaration signatures use the principal's identity suite and
representative proofs use the representative's. Public keys, signature domain
and canonical authorization bytes must be fully specified before deployment.

The representative carries and presents the declaration. A rendezvous is not
required to store it or participate in its authorization verification. Discovery
may still return candidates claiming to serve a principal; such a claim alone
grants nothing. A stolen declaration without proof of its exact representative
identity grants no authority. No controller contact is required during use.

The declaration is per principal/representative authorization, not per client.
It MUST NOT require a client identity or client public key as a signed audience
field. The same valid declaration may be presented to different clients and in
multiple authenticated sessions without reissuance for each client. Removing
per-client issuance does not remove its signed principal, representative, scope
or validity, and does not authorize anyone other than that representative.

Client-specific binding belongs to the authenticated presentation: each HS2
establishment binds the actual peers, expected logical principals, fresh context,
policy and exact authorization evidence. A captured presentation cannot establish
authority in another client's session. Reuse of the declaration is permitted;
replay of a session-bound proof is not. Renewal likewise needs a fresh signed
declaration for the representative, not a separate declaration per client; each
session independently validates its protected presentation and validity.

The declaration authorizes responding for the exact principal, not transfer of
control or subdelegation. A representative cannot issue replacement declarations
on the principal's behalf. Client policy may require direct service. The expected
principal, concrete peer, representation mode, authorization evidence and
applicable policy must be bound into the authenticated establishment so they
cannot be silently substituted. Identity naming alone is not application/service
permission, and the client must retain both logical and concrete principals.

This requires an explicit HS2 extension or successor with downgrade-resistant
negotiation. Current HS2 proofs cannot silently acquire these meanings. In
particular, Application Agreement v1 binds concrete HS2 principals and does not
currently authorize replacing them with represented principals. Its successor
integration must bind and preserve both before admitting delegated application
records; this decision does not enable POST or delegate social authorship.

### 3.1 Application-neutral representation; service permissions outside PRP

The declaration authorizes the exact representative to act for the exact
principal, independently of application
or service. It MUST NOT require per-application, application-profile/version or
INIT/RESP-role issuance. Actual session roles remain authenticated presentation
inputs, not separate service permissions in the declaration. The same authority
may be presented in either role, subject to the peer's direct/represented policy
and all identity, validity and session-binding checks.

Service permissions and application authorization are outside the normative
scope of this PRP representation mechanism. They belong to higher-layer
contracts, such as PRP Social. PRP MUST NOT infer service access, social authorship
rights or permission to perform an application operation merely from successful
identity authentication or representation. No application-specific permission,
role or quota field is introduced into the base declaration by this decision.

References in this document to representation scope mean the exact principal /
representative relationship and its validity and non-subdelegation constraints,
not a PRP-defined service permission model. Protocol resource bounds, peer
acceptance policy and authenticated application/profile agreement remain
distinct from service authorization. Application Agreement v1 remains bound to
concrete principals; representation-aware integration must preserve logical and
concrete identities without defining the higher layer's permission rules.

Earlier review candidates requiring signed application/profile/version, service
quotas or per-connection-role declarations are not adopted. Their codecs and
vectors need a coordinated successor reflecting this decision; no legacy bytes
or existing admitted agreement are changed by analogy.

## 4. Expiry, renewal and data admission

Declarations have signed validity intervals. There is no protocol mechanism for
early revocation of a declaration in this model. Publishing another declaration
or deleting a discovery entry does not invalidate distributed valid copies.
Compromise may therefore remain usable until expiry; independent local policy
may refuse service but is not global revocation. Multiple declarations may overlap.

Authorization must remain valid for admission throughout represented communication,
not only at opening. A new principal-signed declaration may be obtained in advance
and presented by the representative before the current one expires. Replaying
the same declaration does not extend its validity. Distinct renewal evidence and
the extended validity must be verified without requiring direct controller contact.

At expiry, no new data may be admitted in the principal's name. This includes
received, buffered or queued records not yet admitted. Work admitted earlier
may complete only its already-authorized processing/effects, not admit further
work on that basis. Admission and expiry must have an unambiguous order across
asynchronous execution and commits. This is distinct from explicit close or
context invalidation, which retains its own lifecycle requirements.

The underlying authenticated connection may remain. Direct communication with
the representative under its own identity is unaffected; represented traffic
must never be silently converted into direct traffic. A fresh valid declaration
may resume represented admission in the same still-valid authenticated context
with the same principal, representative and permitted scope. Closed/replaced
contexts are not resurrected by renewal. Representation expiry is not a crypto
session rekey, nor does rekey independently extend the declaration.

The acceptance model uses half-open validity [start, end): end itself is expired.
Before byte-level admission, the coordinated contract must fix time units,
trusted-clock requirements, uncertainty/backward-jump handling,
renewal replay/cache bounds and the authenticated carrier for renewal. No wire
generation field is justified solely as revocation, since early revocation is
not part of this model.

### 4.1 Bounded authority horizon, not immediate revocation

The model deliberately makes no claim of immediate or universal revocation.
A disconnected verifier cannot distinguish continued approval from a withdrawal
it has not learned. No mandatory online status oracle, revocation list or gossip
mechanism is introduced. Local refusal is always possible but is not a globally
propagated revocation guarantee. Expiry remains dependent on trustworthy time.

The general profile bounds individual lifetime L and total future authority
horizon H. Let I be the principal's signed issuance-time claim, S the signed
validity start and E the signed end. Admission requires:

```
I <= S < E
E - S <= L
E - I <= H
L = H = 604800 elapsed seconds (seven days)
```

Equality at L or H satisfies that bound; end itself remains expired under
[S, E). Seven days is a maximum, not a default or a required issuance duration.
A declaration exceeding an applicable bound MUST be rejected, not rewritten
or silently clamped into a different signed interval. Arithmetic MUST reject
overflow and invalid ordering rather than wrap.

There is no independent advance-start parameter A in this profile. The ordering
and horizon already imply S - I < H: future start consumes the same seven-day
horizon. A start one day after issuance therefore leaves at most six days of
validity. This replaces earlier proposals for a separately bounded A; it does
not permit future-start declarations outside H.

Verifiers may impose stricter local acceptance policy, never enlarge the profile
limits. The profile must be selected unambiguously before admission; a peer
cannot obtain a weaker profile by fallback. Exact profile selection, complete
signed-record encoding and integrated clock-recovery evidence remain pending.
No longer-lived or interplanetary profile is admitted in this phase.

The signed issuance, start and end instants use International Atomic Time (TAI),
selected in revision 14. This is a time-scale decision, not a requirement for
an atomic clock, external time service or a particular implementation. POSIX/UTC
values MUST NOT be reinterpreted as TAI without a valid conversion. Conversion
uncertainty is part of the verifier's conservative time interval; unavailable
or unbounded conversion cannot authorize represented admission. A permanently
fixed present-day UTC/TAI offset is not a valid general conversion rule.

I, S and E are unsigned 64-bit integer counts of whole seconds since
1970-01-01 00:00:00 TAI, selected in revision 15. Their byte representation is
big-endian, consistently with this document's integer convention. The range is
0 through 2^64 - 1; zero denotes the epoch and no value denotes infinity or
unknown time. Negative values, fractional fields and out-of-range values reject;
verifiers MUST NOT truncate, wrap or reinterpret signed values. Precision of
one second is not a claim of clock accuracy. The signed claims are not rounded
or rewritten by a receiver.

This defines the temporal scalars, not their offsets/order in a complete signed
declaration or an independently usable 24-byte wire message. Profile selection,
declaration framing and signature preimages still require coordinated admission.
The uncertainty limit is specified below. No selector, signature domain or HS2 field is
allocated by this decision.

Renewal requires another actual issuance decision and principal signature.
Receipt, session activity or presentation of an existing declaration cannot
extend its interval. No principal contact with each client is required.

For all declarations already issued to a representative, exposure is bounded by
the latest authorized E, including future-start renewals, not by the lifetime of
the currently presented declaration. Issuers MUST NOT evade H by pre-signing a
chain of short declarations extending arbitrarily far into the future. A future
renewal needs another actual issuance decision, not an automatic extension inferred
from connection state or possession of an earlier declaration.

An issuance timestamp is an authenticated claim, not cryptographic evidence of
when signing occurred. A verifier cannot detect a controller that intentionally
pre-signs declarations with false future issuance times and withholds them until
those times. The bounded-horizon guarantee assumes a controller that follows the
issuance policy and stops issuing when it withdraws trust. It protects against a
representative holding previously issued authority; it is not protection against
a malicious/compromised controller. Stronger issuance-time evidence would require
additional assumptions and is not silently introduced here.

The verifier must establish a trusted current-time interval [lo, hi]. The general
profile's maximum uncertainty U is 60 seconds, selected in revision 16, measured
as the full width hi - lo, not a plus/minus tolerance. Verifiers may require a
smaller width; they MUST NOT accept a larger one under this profile. The bound
includes source, conversion, elapsed-time and precision uncertainty. A width
equal to U satisfies this bound; a larger or unknown width fails closed.
Admission requires the complete interval to lie within
[S, E): S <= lo and hi < E. The issuance claim must not be in the verified future
(I <= lo). Unknown time, excessive uncertainty or clock regression must not grant
extra validity; their fail-closed/resynchronization behavior needs exact vectors.
Neither receiving a declaration nor a fresh HS2 nonce proves current wall time.

U does not extend signed validity: an interval reaching E is rejected even if
its width is less than 60 seconds. Time evidence must remain conservative at
admission; rounding or neglecting elapsed time MUST NOT narrow the interval
artificially. Exact propagation and recovery integration remain coordinated work.

When time is unknown, exceeds the accepted uncertainty bound, regresses or loses
reliable continuity, the verifier MUST suspend represented admission. This
suspension persists until trustworthy time is explicitly re-established under
local policy. Receiving a declaration, renewal, rekey, cancellation or reconnection
MUST NOT by itself clear the suspension. Recreating local state or restarting a
device is not evidence of recovered time.

After trusted recovery, the verifier MUST revalidate the declaration's signed
interval and limits, representative proof and current authenticated context
before resuming represented admission. Previously verified candidates cannot
bypass this revalidation. Recovery neither extends signed validity nor resurrects
a closed/replaced context; existing replay protections remain required.
An otherwise valid direct identity relationship is not automatically invalidated
by this representation-specific suspension. These recovery semantics are approved
in revision 17; they prescribe neither a clock implementation nor a time service.

The semantic checker covers the selected seven-day L/H bounds separately from
illustrative abstract L/H/U cases. These include acceptance of future start
within H without a separate A check. Required vectors include boundary equality,
lifetime/horizon excess, future issuance, pre-issued renewal chains, clock
uncertainty and integer overflow. No actual timestamp attestation or trusted
clock is implemented by it.

### 4.2 Temporal verification boundary for this phase

The optional temporal notary is outside this phase.
No attestation service, trust/bootstrap protocol,
query/response, locator or related wire assignment is required to consolidate
REF-001. This removal is not a claim that such a service was implemented or
that its feasibility was established.

The principal-signed declaration retains its validity interval under sections
4 and 4.1. Removing attestation from scope does not make validity optional or
permit indefinite authority, validity measured anew from receipt, or acceptance
when the verifier cannot establish temporal validity. Exact temporal encoding,
framing and uncertainty bounds remain coordinated decisions; epoch, units and
scalar representation, as well as L/H, are fixed above.

The party accepting represented authority is responsible for checking that
validity. This phase does not prescribe an RTC, time-acquisition service or
particular implementation. If reliable temporal verification is unavailable,
represented admission fails closed; direct identity communication remains
subject to its own authentication and lifecycle requirements. Existing rules
for uncertainty, expiry, renewal and direct-peer isolation remain unchanged.

## 5. Required coordinated increment

REF-001 must return the following, with exact OIDs and executable vectors:

1. Registry/format selection and migration from four kinds to two without
   renumbering or accepting old values by analogy. Old strong-alias declarations
   do not automatically become portable principal authorizations; old text aliases
   do not become fixed bytes by padding, truncation or an invented hash. Native
   CONTENT is absent only in the selected successor, without automatic mapping.
2. New canonical target/Bloom/exact-lookup derivations, publication/candidate
   semantics and bounds. Do not erase required length fields in enclosing
   messages or cryptographic preimages merely because identifier framing changed.
   Include generic local catalogs and alias-only Bloom discovery of multiple
   candidates on one carrier, explicit selection, no second identity Bloom
   search, stale/substituted context rejection and direct/represented HS2 binding.
3. Signed declaration codec: exact principal and representative identities,
   application-neutral authority, signed I/S/E, key/proof binding,
   domain separation, algorithms, bounds and unknown-field rejection. No numeric
   opcode, extension field or signature context is assigned by this document.
   There is no exact-client audience field. Include positive evidence for the
   same declaration with two different authenticated clients, and negatives for
   copied cross-client/cross-session presentations and representative substitution.
   Do not carry forward candidate per-application/role/quota fields or an
   independent A parameter. Use the TAI epoch and u64-second scalars defined
   in section 4.1; specify complete framing, conservative precision conversion
   and overflow rejection. Define U,
   uncertainty propagation and fail-closed clock recovery without prescribing
   an RTC or an external time service.
4. Authenticated HS2 direct/represented selection and transcript binding; renewal
   carriage and application-agreement integration. No legacy fallback after an
   unmet represented-identity or direct-only requirement.
5. Positive and negative vectors for both principal/peer roles, distinct suites,
   forged/wrong-principal/wrong-representative declarations, cross-session replay,
   subdelegation rejection, expiry and renew/admit races, early work completion,
   direct-peer isolation, restart, clock faults, resource limits and downgrade.

The semantic traces in vectors/reference-representation-v2 are abstract expected
outcomes only. They test no signature, HS2 bytes, DNS, browser or provider code.
The next owner sequence is prp-spec decision → parent-routed prp-contracts
coordination → canonical providers → consumer evidence → prp-spec reconciliation.
Reuse existing assignments; this decision is not an instruction to activate or
release any implementation. No speculative third class or future code is reserved.
