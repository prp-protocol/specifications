# PRP Multicast and Groupcast Reference Model

- Version: 1
- Status: Working Draft; core semantic and local-resource model approved,
  exact wire encoding and event grammar pending
- Author: Gustavo Junior Alves, GJ LABS, <gjalves@gjalves.com.br>

## 1. Scope

This document defines the PRP semantic model for persistent MULTICAST and
GROUPCAST references. It distinguishes an identity, a logical transmission, a
group and a particular cryptographic session. It also defines authority
boundaries, nested group composition, untrusted aggregation and the local
resource lifecycle of a group instance.

This is an application-neutral PRP specification. Push-to-talk, media, floor
control and application-specific participant rights are outside its scope and
MUST NOT be inferred from the examples that motivated the model.

This version does not yet assign the canonical serialization of genesis records
or administration events. It therefore does not by itself admit MULTICAST or
GROUPCAST bytes on an existing wire protocol. Section 12 lists the remaining
wire gates.

Uppercase requirement words are to be interpreted as described by BCP 14
(RFC 2119 and RFC 8174).

## 2. Reference classes

The common PRP reference envelope is:

```text
reference_header:u8 || reference_suite:u16be || identifier:L(reference_suite)
```

The approved class codes are:

| Header | Class | Meaning |
| ---: | --- | --- |
| `0x00` | IDENTITY | An identity SCIR committing to canonical public-key material |
| `0x01` | ALIAS | A global rendezvous SCIR committing to canonical text |
| `0x02` | MULTICAST | A SCIR for a stable logical unidirectional transmission |
| `0x03` | GROUPCAST | A SCIR for a group and its independently governed participation state |

Bits 7 through 2 of `reference_header` are zero. A parser MUST reject a
nonzero reserved bit and MUST NOT coerce one class into another.

All four are PRP Self-Certifying Identifier References (SCIRs). Their compact
identifiers commit to different immutable certifying material; none contains an
embedded signature. Only IDENTITY commits to a public key and supports the
corresponding proof-of-possession interpretation. Authority claims concerning
ALIAS, MULTICAST and GROUPCAST remain separate authenticated statements made by
IDENTITY SCIRs.

For a given `reference_suite`, all four classes have the same identifier length.
Suites `0x0001`, `0x0002`, `0x0003` and `0x0005` use a 32-octet identifier and
therefore a 35-octet reference. Suite `0x0004` uses a 48-octet identifier and
therefore a 51-octet reference. Equal length does not make classes or suites
interchangeable.

For non-identity classes, `reference_suite` selects the identifier-formation
hash, output length and security profile. It does not select the signature
algorithm used by an external authority statement. Such a statement is verified
under the identity suite of the signing SCIR. A session-protection suite is a
third, independent selection.

## 3. Identity, multicast, groupcast and session

These objects answer different questions:

| Object | Question |
| --- | --- |
| IDENTITY SCIR | Who is the service, administrator, controller or participant? |
| MULTICAST | Which logical unidirectional transmission is meant? |
| GROUPCAST | Which governed group is meant? |
| Session | Which current cryptographic execution carries data? |

A controller can operate multiple simultaneous multicasts for different
application uses. A multicast can survive session replacement. A groupcast can
outlive its creator, its current participants, every associated multicast and
every current session.

Neither a MULTICAST nor a GROUPCAST reference proves its creator, controller,
administrators, participants or current session. Those facts require valid
external evidence.

## 4. Common managed-reference genesis

MULTICAST and GROUPCAST use one common immutable genesis object:

```text
reference_genesis = {
    application_profile,
    authority_scir,
    creation_nonce
}
```

There is no `format_version` field. The identifier-formation domain and its
canonical encoding define the format generation. A future incompatible format
uses a new domain or an explicitly assigned suite rule rather than an in-band
version byte whose interpretation would itself require prior agreement.

The candidate common formation is:

```text
reference_id = REFERENCE_HASH_S(
    ASCII("PRP-MANAGED-REFERENCE-v1") ||
    reference_class:u8 ||
    S:u16be ||
    canonical(reference_genesis)
)

reference = reference_class:u8 || S:u16be || reference_id
```

`reference_class` is `0x02` for MULTICAST and `0x03` for GROUPCAST. Including it
in the hash domain prevents equal genesis bytes from producing a cross-class
collision. `REFERENCE_HASH_S` produces exactly `L(S)` octets.

The exact per-suite hash definitions, canonical genesis encoding,
`application_profile` representation and `creation_nonce` requirements remain
wire-admission gates. An implementation MUST NOT invent a local encoding and
describe the resulting reference as interoperable.

`application_profile` identifies an external application contract. PRP assigns
no application-specific semantics to it and this document makes no initial
application-profile assignment. A separate `purpose_id` is not present.

For MULTICAST, `authority_scir` names the initial controller. For GROUPCAST, it
names the initial administrator. The authority signs creation evidence over a
distinct purpose domain, the complete reference and the exact canonical
genesis. The signature is not stored inside the reference.

Genesis excludes mutable facts, including locators, session suites and keys,
participants, later administrators, group associations, aggregators,
application-specific rights, sequence numbers, online status and human-readable
labels. Multiple references with the same profile and authority are
distinguished by `creation_nonce`.

## 5. Multicast authority and distribution

A MULTICAST is autonomous and does not require a GROUPCAST. Its creation
evidence names the controller of the logical transmission. A sender MUST prove
that controller identity or present separately authorized representation
evidence. A listener MUST NOT infer controller authority from reachability,
possession of reading keys, an aggregator announcement or knowledge of the
MULTICAST reference.

A MULTICAST has one authorized logical writer at a time under this base model.
Session replacement, route changes and aggregation do not change that writer.
Controller migration requires a separate authenticated transition and does not
alter the historical creation evidence.

Aggregators can receive an already authorized encrypted unit and replicate it.
Aggregation grants no authority to originate content, admit listeners, issue
credentials or represent the controller. Content authenticity MUST remain
verifiable independently of possession of a shared reading credential.

## 6. Groupcast formation and equal administrators

A GROUPCAST is an autonomous group object. Any IDENTITY SCIR can create one.
The `authority_scir` in its genesis is its sole initial administrator; there is
no `initial_administrators` field.

The initial administrator is a bootstrap authority, not a permanent superior.
Additional administrators are introduced by authenticated administration
events. Every active administrator has the same rank, and one active
administrator is sufficient to issue an action. There is no quorum requirement
and no permanent creator privilege.

Subject to the final event grammar, an active administrator can:

- admit or remove a direct participant;
- add or remove an administrator; and
- add or remove an incorporated GROUPCAST.

An administrator cannot act merely by presenting the GROUPCAST reference. Each
action MUST be signed by its author IDENTITY SCIR, bound to the exact GROUPCAST and
validated against a causal state in which that author was active.

The creator can grant administration to other identities and then leave. The
GROUPCAST remains operable while a valid nonempty administrator state exists;
the creator has no special right after departure.

## 7. Incorporating groupcasts

A GROUPCAST can incorporate another GROUPCAST. Incorporation is a mutable,
authenticated relationship between two complete references; it is not part of
either genesis and does not change either identifier.

Let `D(G)` be the active direct participants of G and `I(G)` its active child
incorporations. Derived participants are the least fixed point:

```text
P(G) = D(G) union union(P(H) for H in I(G))
```

An implementation MUST track visited references, bound graph traversal and
deduplicate identities. A cycle contributes no participant beyond the fixed
point and grants no authority to synthesize members.

An incorporation:

1. is authorized by an active administrator of the parent;
2. makes the child's current participants eligible as participants of the
   parent while the incorporation and required membership evidence are valid;
3. does not transfer administration in either direction;
4. does not inherit any application-specific right;
5. grants no right in the child and requires no child-admin approval under the
   base model; and
6. can be removed only by a valid parent administration action.

The parent thereby delegates part of its membership eligibility to the child's
administration. A later valid child membership change can affect the parent's
derived participant set. Applications MUST expose that consequence rather than
presenting incorporation as a static copy.

An access proof for a derived participant MUST identify a currently valid chain
from the parent incorporation to a direct membership statement. Aggregators can
transport the chain but cannot create or repair its authority.

Removal prevents new authorization through that path. It does not erase
previously received data or automatically invalidate an established session.
Applications requiring immediate exclusion must replace the relevant
credentials.

## 8. Application boundary

The base GROUPCAST model defines administrators, direct participants and
incorporations. It does not define media, floor control, publishing rights,
priorities, roles or other application operations. Those semantics belong to
the contract identified by `application_profile` and MUST NOT be inferred from
GROUPCAST membership alone.

An association between a GROUPCAST and a MULTICAST, when an application needs
one, is an application-defined signed statement external to both references.
It does not make the GROUPCAST necessary for the MULTICAST's existence, confer
writer authority on group administrators or become part of either genesis.

This separation permits an autonomous MULTICAST, multiple simultaneous
MULTICAST references for different purposes, and applications that use
GROUPCAST without any MULTICAST object.

## 9. Administration events and convergence

Administration events require canonical purpose domains, author IDENTITY SCIRs, target
references, operation-specific subjects, causal predecessors, uniqueness,
validity and signature verification. Replicas can store and exchange events,
but storage does not make an invalid event valid.

For concurrent grant and removal of the same participant, administrator or
incorporation, removal wins. A later grant is effective only when it causally
observes the removal it supersedes. This remove-wins rule prevents a stale grant
from reviving an intentionally removed right.

An action authored by an administrator concurrently removed from that role MUST
NOT create a new right until the causal conflict is resolved. Implementations
retain the conflicting evidence and fail closed rather than choosing a branch
from arrival order.

The settled administrator set MUST NOT be empty. The exact interoperable rule
for concurrent removals that would collectively remove every administrator
remains an event-grammar gate. The candidate fail-closed rule retains the last
common nonempty administrator state for conflict-resolution actions only; no
ordinary administration proceeds until a nonempty resolution causally
references every conflicting branch.

The final grammar must also specify causal gaps, replay, duplicates, competing
incorporations and effects on established sessions. Until that grammar and its
negative vectors are admitted, an implementation MUST NOT claim interoperable
group administration merely because it can verify individual signatures.

## 10. Local creation and demand-driven maintenance

GROUPCAST authority, consumer demand and local resource custody are independent.
None implies either of the others.

At the aggregator where a GROUPCAST is created, the creator consumes an initial
local creation-admission quota. This quota limits creation abuse and can be
implemented as a bounded count, rate limit or token bucket. It is not a
persistent maintenance charge and creates no global debit.

After creation, the creator is an ordinary consumer. Consumers do not sponsor
or pay for GROUPCAST storage, acquire a GROUPCAST-specific maintenance lease or
incur a special charge. An active authorized local consumer, whether an
administrator or not, gives the local instance a reason to exist. Administrator
status alone is not consumer demand. Consumer authorization is the aggregator's
ordinary authorization to serve that consumer; GROUPCAST does not create a
second payment or sponsorship relationship.

While at least one active authorized local consumer exists, the aggregator
maintains the local GROUPCAST state within its ordinary capacity, rate and work
bounds. The aggregator bears that continuing cost because grouping consumers
into shared state or connections is its own distribution optimization.

When no active local consumer remains, the instance becomes quiescent and
evictable. The aggregator may retain a cache at its own cost or discard the
local material. Eviction neither revokes nor destroys the logical GROUPCAST;
a later consumer can cause reconstruction from genesis and valid events.

A replica at another aggregator is likewise demand-driven and creates no debit
against the original creator. An incorporated child is materialized or expanded
only when active demand needs it, subject to ordinary local graph, request and
work limits. A stale incorporation MUST NOT by itself retain an entire nested
tree.

The conceptual local lifecycle is:

```text
ABSENT -> CREATOR_BOOTSTRAP -> DEMAND_MAINTAINED -> QUIESCENT -> ABSENT
ABSENT -> DEMAND_MAINTAINED
```

The second path represents demand-driven reconstruction at any aggregator.
This model does not guarantee continuous availability, global persistence or a
particular local accounting algorithm.

## 11. Carriers and publication

The binary carrier direction is the complete reference followed by optional
application parameters. The textual direction is lowercase `prp:` followed by
canonical unpadded base64url of those same bytes. Application parameters are not
part of the reference and gain no authenticity from proximity.

The generic PRP URI/QR carrier direction accepts all four classes but is not yet
admitted. It dispatches by class, derives the reference boundary from the
recognized class/suite pair, enforces an input bound before decoding or
allocation, and provides no old-format fallback. Carrier recognition does not
admit the formation or authority semantics of a class/suite pair.

A public QR can carry a reference and discovery/bootstrap data. An invitation
or administration QR carries additional signed authorization and MUST be
distinguished from a public reference. If genesis is transported, the receiver
recomputes the identifier and verifies its creation evidence; physical proximity
to the QR code is not authority.

The PRP DNS Reference RR can publish all four reference classes. DNSSEC
authenticates name-to-reference publication, not multicast control, groupcast
administration or participation. DNS remains optional.

## 12. Required work before wire admission

The following items remain open and require positive and negative vectors:

1. exact `REFERENCE_HASH_S` definitions for every admitted reference suite;
2. canonical `reference_genesis`, `application_profile` and nonce encodings;
3. nonce length, generation requirements and collision handling;
4. creation-proof and mutable-event purpose domains and signed bytes;
5. administration-event grammar, including the exact nonempty-admin conflict
   resolution around the approved remove-wins rule;
6. direct and derived membership proof formats;
7. binding and versioning rules for external application profiles;
8. catalog replication, gap recovery and bounded resource behavior;
9. generic binary/URI/QR dispatch and application-parameter boundaries;
10. parser bounds, nesting limits and denial-of-service controls; and
11. complete malformed, cross-class, cross-suite, replay, conflict and cycle
    vectors.

These gates do not weaken the approved distinctions: all four classes are
SCIRs but only IDENTITY has key-possession semantics; incorporation never
inherits administration or application rights, aggregators
gain no authority, consumers incur no GROUPCAST-specific maintenance charge,
and application examples do not become PRP semantics.

## 13. Security and privacy considerations

A stable MULTICAST or GROUPCAST reference permits correlation. Genesis and
creation evidence expose a stable initial authority IDENTITY SCIR when published. Nested
membership evidence can reveal social structure and transitive group
relationships; implementations should disclose only evidence needed for the
current authorization decision.

Reference equality is not current authorization. Verifiers must validate
freshness, causal state, removal and the complete authority chain. They must
bound graph traversal, event count, proof size and input before allocation. A
cycle-safe fixed-point computation prevents infinite traversal but does not by
itself prevent large-graph denial of service.

Demand is not authority, and presenting artificial demand must not create
membership or administration rights. Aggregators must apply their ordinary
admission, abuse and capacity controls to creation and reconstruction work.

Removal cannot retract previously disclosed plaintext. Forward exclusion needs
new credentials or keys; backward exclusion may require an independent retained-
history policy. No wall clock, trusted time, global consensus or continuously
available server is assumed by this semantic model.
