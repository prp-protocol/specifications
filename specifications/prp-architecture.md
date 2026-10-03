# PRP Architecture

- **Version:** 2 (working revision)
- **Status:** Informational architecture candidate
- **Category:** Foundational architecture
- **Author:** Gustavo Junior Alves
- **Date:** 2026

## Abstract

This document defines the architecture of the Participant Relationship Protocol
(PRP). PRP treats a relationship, rather than an address or route, as the
primary context from which communication emerges. It separates persistent
relationship semantics from replaceable identities, sessions, trajectories,
routes, and carriers.

PRP is a relationship-centric, carrier-independent communication architecture.
It is independent of IP and of any other particular network-layer protocol.
IP-based and non-IP carriers are optional realizations of the same
relationship-centric architecture.

PRP can provide a Layer-3 realization that replaces conventional addressing,
but the architecture itself is not defined by an OSI layer. A relationship is
not a per-packet forwarding object, Layer-3 address, routing-table entry,
carrier identifier, or cryptographic session.

This document specifies architectural terms, invariants, layers, boundaries,
and conformance requirements. It intentionally does not specify a wire format,
routing algorithm, carrier technology, discovery mechanism, governance system,
or application protocol.

## 1. Status and requirements language

This document is an Informational architecture candidate. It defines the
architectural boundaries against which PRP protocol profiles and
implementations can be evaluated; it does not claim IETF consensus or Internet
Standard status.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in this document are to be interpreted as described by BCP 14 when,
and only when, they appear in all capitals.

## 2. Scope

This document defines:

- the relationship-centric communication model;
- the distinction among relationships, identities, sessions, trajectories,
  routes, paths, adjacencies, transports, and carriers;
- the architectural layers and their dependencies;
- continuity, authorization, and trust boundaries;
- architectural security and privacy properties; and
- requirements for architectural conformance.

This document does not define:

- packet or record layouts;
- cryptographic algorithms or numeric registries;
- session-establishment messages;
- routing or discovery algorithms;
- carrier bindings;
- application object models;
- persistence or deployment mechanisms;
- governance, economic, evidence, or custody systems; or
- administrative interfaces.

Separate PRP specifications may define those subjects provided they preserve
the invariants in this document.

## 3. Foundational model

The foundational statement of PRP is:

> Communication becomes possible because a relationship already exists.

A relationship supplies the context in which communication has continuity and
authorization semantics. Trajectories, scoped route decisions, paths,
transports, and carriers realize communication for that context; they do not
create the relationship merely by providing reachability.

An address-centric sequence is commonly expressed as:

```text
address -> route -> transport -> communication
```

PRP instead places the relationship first:

```text
relationship -> continuity
                    ^
                    |
 trajectory / route decisions / transport / carrier
```

The realization mechanisms may change without replacing the relationship.

The intended cardinality is:

```text
participant relationship
        -> one or more cryptographic sessions
           -> one or more compatible routing contexts
```

This cardinality does not require relationship evaluation for every packet.
Relationship parsing, policy evaluation, identity constraints, governance
decisions, and session establishment are control operations. Forwarding uses
only derived, bounded, local dataplane state.

## 4. Terminology

### 4.1 Participant

A **participant** is an entity capable of taking part in a relationship. This
architecture does not constrain whether a participant is represented by a
person, process, service, device, organization, or another entity type.

### 4.2 Relationship

A **relationship** is a persistent communication context among participants.
It defines continuity rules, authorization boundaries, communication
permissions, trust material, and any identity constraints.

Persistence is semantic, not a requirement for a particular storage mechanism
or uninterrupted availability. A relationship may exist while no trajectory,
path, session, or carrier is available.

### 4.3 Continuity

**Continuity** is the ability to determine, under relationship policy, whether
an interaction belongs to the same relationship context across time or change
of realization mechanism.

Continuity may be pseudonymous, identity-bound, or based on another explicit
relationship rule. "Anonymous" continuity means that no external or civil
identity is required; it does not imply the absence of a demonstrable protocol
principal when a selected protocol profile requires one.

### 4.4 Identity and identity constraint

An **identity** is a representation by which a participant or service may be
distinguished within a defined scope. An **identity constraint** is
relationship policy specifying which identity properties are required,
permitted, or prohibited.

PRP architecture does not require a universal identity namespace, external
identity authority, civil identity, or globally resolvable name. A protocol
profile MAY require a self-certifying identity and proof of possession. The PRP
Self-Certifying Identifier Reference (SCIR) is the separately specified
canonical typed reference. An IDENTITY SCIR commits to canonical public-key
material; it is not a machine address, session, credential or relationship.

#### 4.4.1 Protocol reference classes

Specifications above this architecture can distinguish several
non-substitutable reference categories:

- an **IDENTITY SCIR** commits to one concrete principal or service identity;
- an **ALIAS SCIR** commits to global canonical rendezvous text while selected
  concrete members still prove their own IDENTITY SCIRs;
- a **MULTICAST SCIR** or **GROUPCAST SCIR** commits to immutable genesis while
  current authority and participation remain separate authenticated state; and
- a **content ID** is an immutable, suite-bound commitment to finite content
  bytes and is not an entity identity.

These classes are non-substitutable. In particular, a content ID has no
private key, cannot prove possession or sign an identity challenge, and does
not assert an owner, author, provider or semantic meaning. Availability of
bytes matching a content ID is a separate, revocable claim.

The category names in this section describe architectural semantics rather
than allocating wire values. The SCIR specification defines the common envelope
and class-specific formation boundaries. Sharing that envelope does not make
ALIAS, MULTICAST or GROUPCAST inherit IDENTITY proof-of-possession semantics.
Content IDs remain outside these four SCIR classes.

### 4.5 Trust material

**Trust material** is information used to evaluate continuity, authorization,
or identity constraints. Trust material supports a relationship; possession of
transport reachability alone is not trust material unless relationship policy
explicitly makes it so.

### 4.6 Authorization boundary

An **authorization boundary** defines the actions permitted in a relationship.
Authentication establishes or verifies a principal or continuity claim;
authorization determines what that claim is allowed to do. Implementations and
profiles MUST NOT treat successful reachability or authentication as implicit
authorization for arbitrary services.

### 4.7 Session

A **session** is bounded cryptographic and operational state used to exchange
information within a defined scope and generation. A relationship may use
multiple sessions over its lifetime. Rekeying or replacing a session does not
by itself replace the relationship.

A specification defining sessions MUST identify their scope, generation,
replay domain, key lifetime, and replacement behavior.

### 4.8 Route, trajectory, path, and routing identifier

A **route** is a scoped directional forwarding decision used in the
realization of a trajectory. A route selects an eligible next path, adjacency,
or local-delivery outcome. It is one decision leg of a trajectory and is not
the trajectory itself.

A **trajectory** is a validated directional realization of reachability for a
relationship. A trajectory may span multiple route decisions and contains one
or more **paths**. A path is an authenticated directional opportunity to carry
traffic.

A **routing identifier** is an operational selector for local route state,
interpreted within an explicit routing scope and lifetime.

Routing identifiers MUST NOT be treated as participant or service identities.
Their uniqueness MUST NOT be assumed outside their specified scope or lifetime.

### 4.9 Adjacency

An **adjacency** is a context between participants that can directly exchange
units through a carrier under a defined local scope. Carrier enumeration or
physical proximity may produce an adjacency candidate; it does not by itself
establish an authenticated relationship.

### 4.10 Transport

**Transport** is the exchange of information used to realize a relationship.
A transport profile specifies whether and how that exchange is authenticated,
confidential, integrity-protected, and replay-protected. It may provide
unreliable items, reliable items, ordered delivery, streams, or other semantics.
No security or delivery property is universal merely because one transport
offers it.

### 4.11 Carrier

A **carrier** is a mechanism capable of moving PRP units between adjacent
participants. Carrier technology and locators are realization details. A
carrier MUST NOT acquire identity, relationship, authorization, or routing
authority solely because it conveys PRP units.

### 4.12 Service

A **service** is a capability exposed within an authorization context above the
core relationship semantics. Discovery, governance, evidence, evaluation,
benchmarking, economics, custody, and application protocols may be services.
Their importance does not make them prerequisites of every valid PRP
deployment.

### 4.13 Aggregation and aggregator

**Aggregation** is an optional relationship-realization arrangement in which
an authorized participant coordinates or intermediates multiple independently
authorized relationship contexts or trajectories. Aggregation does not merge
relationships, identities, continuity state, or authorization boundaries. It
is distinct from packet batching, data summarization, and group identity.

An **aggregator** is a participant performing aggregation under explicitly
delegated relationship authority. Acting as an aggregator grants no implicit
authority over endpoint identity, relationship continuity, or application
authorization.

### 4.14 Evidence and evaluation

**Evidence** is a verifiable observation concerning communication activity.
**Evaluation** is an interpretation of evidence. PRP does not require one
globally authoritative evaluator.

## 5. Architectural invariants

### 5.1 Relationship primacy

A conforming PRP design MUST identify the relationship context whose
communication it realizes. It MUST NOT define relationship continuity solely
as continued possession of a locator, route, carrier attachment, or ephemeral
session handle.

### 5.2 Independent lifetimes

Relationships, identities, sessions, trajectories, routes, paths, adjacencies,
and carriers have distinct lifetimes. A design MUST define transitions among
them without silently equating one object with another.

In particular:

- a relationship may outlive any particular session, trajectory, route, or
  path;
- a trajectory may be replaced and its route decisions may change without
  replacing the relationship;
- a session may be rekeyed without changing participant identity;
- an adjacency may exist without authorization to form a relationship; and
- loss of a carrier does not by itself revoke a relationship.

### 5.3 Local scope

Operational selectors are local unless a specification explicitly defines a
larger authenticated scope. Local selectors MUST NOT be interpreted as global
addresses. A forwarding participant MAY rewrite such selectors while
preserving the end-to-end relationship semantics defined by the selected
profile.

### 5.4 Explicit authority

Every protocol decision that affects access to a relationship or service MUST
derive from relationship policy or an explicitly delegated authority. A route,
carrier, discovery result, or successful handshake MUST NOT silently expand
authorization.

### 5.5 Replaceable realization

A conforming design SHOULD support replacement of a failed or obsolete
realization mechanism without replacing the relationship when continuity can
be proven. When continuity cannot be proven, it MUST fail or establish a new
relationship according to explicit policy; it MUST NOT silently reset replay,
ordering, or identity state while claiming continuity.

### 5.6 No mandatory global infrastructure

A conforming PRP protocol suite MUST permit two participants with sufficient
local relationship information and a compatible carrier to establish
communication without consulting a mandatory global naming, routing, identity,
governance, or ledger service.

Profiles MAY depend on external services for their chosen operation. Such a
dependency is a property of that profile or deployment and MUST NOT be stated
as a universal PRP architectural requirement.

### 5.7 Network-layer independence

The PRP architecture MUST NOT require IPv4, IPv6, or any other particular
network-layer protocol as a universal prerequisite. A conforming realization
may operate over any compatible carrier capable of moving protocol units
between adjacent participants.

A profile MAY require IP or another network technology. That dependency is a
property of the profile, not of PRP itself. When an IP-based profile is used,
IP addresses and routing are scoped realization mechanisms and MUST NOT become
participant identity, relationship continuity, or authorization.

## 6. Layers

PRP separates four core architectural concerns:

1. the **relationship layer** defines continuity, trust, authorization, and
   communication permissions;
2. the **routing layer** selects and maintains trajectories and the scoped
   route decisions that realize a relationship;
3. the **transport layer** provides authenticated exchange semantics over
   those trajectories; and
4. the **carrier layer** moves units between adjacent participants.

Services operate above these concerns. A concrete protocol may combine layers
within one component, message, or cryptographic transcript, but it MUST retain
their semantic boundaries.

A function belongs to the architectural core only when all of the following
are true:

1. every valid PRP deployment requires it;
2. it cannot be provided as an optional service; and
3. omitting it would invalidate the relationship-centric model.

## 7. Architecture versus implementation

The architecture defines semantic objects and invariants. An implementation
realizes them using concrete state, code and carrier facilities. The following
objects are distinct even when one implementation stores them in the same
allocation or constructs them in one protocol exchange:

- a **relationship** owns persistent continuity and authorization policy;
- a **cryptographic session** owns bounded keys, generations, replay state and
  transcript bindings;
- a **routing context** owns scoped forwarding choices and local selectors;
- **dataplane state** is the compact derived state needed to process traffic;
- a **carrier binding** associates an adjacency with one mechanism that moves
  complete units; and
- **application or service semantics** determine what authorized traffic means
  above the relationship and transport boundaries.

A relationship is not a packet header, per-packet lookup key, network address,
routing-table entry, carrier identifier, or session. Implementations MAY cache
or compile consequences of relationship policy into dataplane state, but a
packet-processing path MUST NOT obtain new relationship authority merely by
finding such state.

### 7.1 Illustrative realization mapping

The following mapping illustrates one possible realization without describing
the current state or internal API of a particular implementation:

| Architectural concept | Illustrative realization |
| --- | --- |
| Relationship | persistent relationship policy and continuity state |
| Continuity | authenticated policy transitions and replacement bindings |
| Cryptographic session | established, generation-scoped session state |
| Routing context | scoped opaque labels and route bindings |
| Dataplane forwarding | local label lookup plus next-hop/session binding |
| Carrier | complete-unit transport adapter |
| Service | encrypted service and multiplexing dispatch |

Implementations MAY use different data structures or process boundaries. They
remain conforming only if the architectural lifetimes, authority boundaries and
failure behavior remain distinguishable.

### 7.2 Control path and packet path

The control path evaluates relationship membership, identity constraints,
policy, governance, discovery input, session establishment and route
installation. It derives a bounded forwarding entry containing only what the
selected profile needs, such as an active session context, cryptographic key
slot, sequence/replay state, opaque local routing label, next-hop binding and
queue or backend state.

For an installed routing context, the per-packet path performs a local lookup,
validates the applicable generation and packet protection, applies replay and
sequence processing, and dispatches to the selected next hop or local service.
It does not parse or re-evaluate the persistent relationship. An intermediate
participant is responsible for the protection applicable to its own layer and
selected profile; it need not terminate or validate independent protection of
opaque endpoint content. A profile MAY require an additional transit
consistency check over that content. Such a check is packet processing and does
not imply evaluation of the persistent relationship.

Intermediate forwarding participants do not evaluate endpoint relationship
semantics; they process only the minimum authenticated local forwarding state
required for opaque delivery. Relationship semantics remain endpoint-owned
control state. That derived state MAY include local responsibility, custody,
queue, or quota bookkeeping compiled during control operations. Consulting
such bookkeeping does not authorize new participants and MUST NOT invoke
identity, membership, or governance evaluation in the packet path.

This property is narrower than claiming that PRP universally "solves" the
end-to-end argument. A concrete profile must still state what each endpoint and
intermediary can see, change and authorize.

## 8. Performance and scaling model

Let **R** be the number of retained relationships, **S** the number of active
cryptographic sessions, **C** the number of installed routing contexts, and
**P** the number of processed packets in a measurement interval.

- Persistent relationship storage scales with **R** and the bounded policy and
  continuity material retained per relationship.
- Session storage, key material and replay windows scale with **S**.
- Forwarding labels, next-hop bindings and route generations scale with **C**.
- Packet processing scales with **P** and the selected packet-protection and
  local-lookup operations.

Relationship resolution is a control-path operation and is not inherently
**O(P)**. For installed state, the desired dataplane cost is one scoped local
lookup plus the cryptographic, replay, queueing and carrier work of the selected
profile. This is an analytical decomposition, not a claim that lookup is
constant-time or that any implementation has production-scale throughput.

An implementation or experiment claiming scale MUST separately report at
least memory per relationship, active session and routing context; lookup cost
as installed state grows; packet size and payload size; concurrency; CPU cost;
throughput or goodput; loss and reordering; and the hardware, software revision
and methodology used. Overlapping owner-local clocks MUST NOT be summed and
reported as elapsed wall time.

Performance comparisons MUST begin with the same application-observable task.
UDP/IP is a carrier and framing component, not a semantic competitor to PRP.
A direct PRP-item-to-UDP-datagram comparison may diagnose carrier, framing or
copy cost, but MUST NOT be presented as an end-to-end architectural comparison.
This architecture does not select a mandatory comparison protocol. A
comparative study MUST define equivalent application-visible behavior, security
properties and exclusions before selecting baselines. Component-level
comparisons remain valid diagnostics when they are not presented as complete
architectural comparisons.

## 9. Architectural novelty and existing mechanisms

The contribution is architectural rather than algorithmic. PRP does not claim
novelty for cryptographic sessions, forwarding labels, multipath
communication, identity/locator separation, connection migration, or carrier
abstraction individually. The proposed architectural distinction is the
treatment of the persistent participant relationship as the primary
communication object, with addressing, routing, transport, carrier, identity
constraints and governance treated as derived or replaceable realization
mechanisms.

This claim is falsifiable: a purported PRP design that makes continuity depend
only on a locator, route, carrier attachment or ephemeral session handle does
not satisfy the architecture, regardless of the names used by that design.

## 10. Discovery, routing, and aggregation

Discovery produces candidates or information used to realize a relationship.
It does not establish identity, trust, continuity, or authorization unless a
separate authenticated procedure explicitly does so.

Routing determines trajectories. A relationship MAY be direct or transit one
or more forwarding participants. Transit does not grant a forwarding
participant authority over endpoint identity or application authorization.

Aggregation is an optional scaling and privacy mechanism. Direct relationships
remain architecturally valid. An aggregation profile MUST specify what an
aggregator learns, which statements it authenticates, and which decisions are
reserved to endpoints. It MUST preserve the independent relationship,
identity, continuity, and authorization boundaries defined by this
architecture.

## 11. Security architecture and claim boundaries

PRP separates security claims by scope. A carrier-security claim does not imply
end-to-end security; adjacent authentication does not imply service
authorization; and a self-certifying identity does not imply an externally
validated identity claim.

A protocol profile MUST specify:

- authenticated principals and transcript scope;
- confidentiality, integrity, and replay properties;
- authorization inputs and failure behavior;
- selector and generation scopes;
- state exhaustion and amplification bounds;
- downgrade and cross-protocol protections; and
- metadata visible to carriers and transit participants.

Unknown or malformed security-critical values MUST fail closed unless a
specification explicitly defines a safe extension rule.

Continuity state is a target for fixation, rollback, replay and confused-deputy
attacks. A replacement realization MUST be bound to the relationship,
participants, direction, generation and policy required by its profile. State
from one scope MUST NOT be admitted into another merely because identifiers or
byte lengths coincide. Loss of continuity evidence MUST NOT be repaired by
treating reachability, a local handle or unauthenticated forwarding state as
equivalent authority. Profiles MUST define revocation, recovery, key replacement
and resource bounds.

Relationship-centric operation does not inherently provide anonymity,
unlinkability, traffic-flow confidentiality, resistance to correlation, or
availability. Padding, batching, multipath, private discovery, and cover traffic
are optional mechanisms whose properties require separate specifications and
analysis.

These statements define required threat-model and information-exposure
boundaries. Architectural conformance does not establish resistance to a state
adversary, global traffic correlation, censorship, traffic confirmation or
denial of service. Any such claim requires a separately specified threat model
and evidence.

## 12. Evidence boundary

Implementations and experiments MAY provide evidence for particular
realizations or architectural properties. Such evidence is versioned separately
and does not become an architectural invariant.

An implementation result does not constrain other conforming realizations to
the same data structures, mechanisms or process boundaries. Architectural
conformance does not imply implementation qualification, deployment suitability
or reproduction of a separately reported result.

A document making an empirical claim MUST identify the evaluated realization or
model, inputs, environment, method, acceptance rule and evidence revision.
Updating that evidence does not change this architecture unless an
architectural invariant itself is revised.

## 13. Claim boundaries

This document defines architectural objects, boundaries and invariants. It does
not define minimum throughput, latency, deployment scale, memory capacity,
routing convergence or implementation economics. Architectural conformance does
not establish those properties.

Performance, scalability, privacy, availability and adversarial-resistance
claims are profile- and realization-specific. Such claims require separately
versioned evidence identifying their configuration, environment, methodology
and observed scope.

The absence of an architectural claim neither establishes nor precludes the
corresponding property in a particular realization.

## 14. Extensibility and evolution

Extensions MUST preserve the architectural invariants in Section 5. An
extension that requires globally stable locators, equates routes with identity,
or makes optional governance a universal prerequisite is not architecturally
conforming.

Protocol versions and profiles MUST define explicit selection and failure
behavior. Negotiation MUST NOT silently weaken authenticated properties or
reinterpret an assigned value. Migration SHOULD preserve relationship
continuity through an authenticated make-before-break procedure when the old
and new profiles permit it.

## 15. Architectural conformance

A design conforms to this architecture only if it:

1. treats relationships as the primary persistent communication context;
2. defines continuity independently of any single trajectory, route, path,
   session, or carrier;
3. keeps operational routing and carrier selectors distinct from identity;
4. makes authorization an explicit relationship decision;
5. permits direct communication without mandatory global governance;
6. does not make IP or another particular network-layer protocol a universal
   prerequisite;
7. states the scope and lifetime of identities, sessions, trajectories,
   routes, paths, and selectors; and
8. does not claim security or privacy properties beyond those provided by its
   specified profiles.

Conformance to this architecture does not imply conformance to any PRP wire,
carrier, discovery, routing, or service profile. Each such specification
defines its own additional requirements.

## 16. IANA considerations

This document requests no IANA action.

## 17. References

### 17.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.

### 17.2 Informative references

- Gustavo Junior Alves, *PRP Architecture v1*, Release Candidate 1, 2026.
- Gustavo Junior Alves, *Relationship-Centric Communication: The Participant
  Relationship Protocol (PRP)*, 2026, DOI 10.5281/zenodo.20482932.
