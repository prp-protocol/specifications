# Participant Relationship Protocol (PRP) Core Semantics

Status: Internet-Draft-style working document
Version: draft-prp-core-00
Author: Gustavo Junior Alves
Date: 2026
Intended status: Informational / Experimental

## Abstract

This document defines the core semantics of the Participant Relationship Protocol (PRP).

PRP is a relationship-centric communication protocol family. It treats relationships as the primary architectural objects from which communication emerges. Routes, transports, carriers, discovery systems, governance systems, and services are mechanisms used to realize or support relationships; they are not the foundation of communication.

This document specifies the semantic requirements that valid PRP implementations and extensions must preserve. It defines participants, relationships, continuity, identity constraints, trust material, authorization boundaries, PRP units, session boundaries, carrier boundaries, routing boundaries, and service boundaries.

This document does not define a byte-level wire format, packet structure, routing algorithm, discovery protocol, governance system, economic system, service protocol, or application object model. Those mechanisms may be specified separately, provided that they preserve the core semantics defined here.

## 1. Introduction

Modern communication protocols are commonly organized around reachability.

A participant communicates by locating another participant, address, endpoint, service, or routeable object. Communication therefore becomes dependent on addressing, routing, naming, locator stability, and infrastructure reachability.

PRP adopts a different premise:

> Communication becomes possible because a relationship already exists.

A relationship defines the continuity and authorization context within which communication is meaningful. Routes, transports, carriers, discovery systems, aggregation systems, and services are mechanisms used to realize that relationship.

The PRP core is therefore not address-centric, identity-centric, route-centric, carrier-centric, or governance-centric. It is relationship-centric.

This document defines the core semantic boundary of PRP. It specifies what must remain true across all valid PRP implementations, profiles, carriers, routing systems, and services.

The objective is to provide a stable semantic foundation for:

* protocol specifications;
* reference implementations;
* carrier bindings;
* routing extensions;
* aggregation mechanisms;
* service-layer systems;
* future Internet-Draft-style documents.

## 2. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **MAY**, and **OPTIONAL** are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

### 2.1 Participant

A participant is any entity capable of participating in a PRP relationship.

The PRP core does not impose a specific participant model. A participant may be a process, device, user agent, service, organization, software component, hardware device, or future entity type.

Participation in a relationship is the only core requirement.

### 2.2 Relationship

A relationship is a persistent communication context between participants.

A relationship defines continuity, authorization boundaries, trust material, communication permissions, and optional identity constraints.

Relationships are the primary architectural objects of PRP.

### 2.3 Continuity

Continuity is the ability to determine whether communication is occurring within the same relationship context across time.

Continuity may be anonymous, pseudonymous, identity-bound, or defined by another relationship policy.

PRP requires continuity.

PRP does not require identity.

### 2.4 Identity Constraint

An identity constraint is a relationship policy that defines identity requirements for a relationship.

A relationship may require identity, permit identity, or operate without identity.

Identity is therefore a property of a relationship, not a prerequisite for PRP communication.

### 2.5 Trust Material

Trust material is information used by a relationship to establish or verify continuity, authorization, cryptographic state, or communication permissions.

Trust material supports relationships.

It does not define a mandatory global identity system.

### 2.6 Authorization Boundary

An authorization boundary defines the set of actions permitted within a relationship.

Authorization is a property of relationships.

### 2.7 Route

A route is an operational trajectory used to realize a relationship.

A relationship may exist without an active route.

A route may change without changing the relationship.

### 2.8 Routing Identifier

A routing identifier is an operational selector used by a routing mechanism.

Routing identifiers are local to the entities that interpret them.

A routing identifier is not a participant identity.

### 2.9 Transport

Transport is the authenticated exchange of information used to realize a relationship.

Transport belongs below relationship semantics and above carrier movement.

### 2.10 Carrier

A carrier is any mechanism capable of moving PRP units.

A carrier may be a packet network, stream transport, radio link, file transport, removable medium, delay-tolerant path, intermittent channel, local IPC mechanism, or future communication mechanism.

Carriers move PRP units.

Carriers do not define relationship semantics.

### 2.11 Service

A service is a capability implemented above the PRP core.

Services may include discovery, notarial validation, governance, evidence publication, economic settlement, application protocols, storage systems, or other higher-layer functions.

Services extend PRP.

They do not define the PRP core.

### 2.12 Evidence

Evidence is a verifiable observation of communication activity.

Evidence may be shared.

Interpretation remains local unless a higher-layer service defines otherwise.

### 2.13 Evaluation

Evaluation is the interpretation of evidence.

The PRP core does not define a globally authoritative evaluation system.

### 2.14 Governance System

A governance system is an optional coordination service operating above the PRP core.

Participation in governance systems is optional.

A governance system does not create PRP communication.

## 3. Core Model

PRP is relationship-centric.

A valid PRP implementation MUST preserve the following model:

```text
Relationship
    defines continuity and authorization

Transport
    realizes authenticated exchange

Carrier
    moves PRP units

Services
    extend or support relationships
```

Routes, transports, carriers, discovery services, governance systems, identity systems, and application protocols are derived or supporting mechanisms.

They MUST NOT be made more fundamental than relationship continuity.

### 3.1 Relationship First

Communication in PRP emerges from a relationship.

A PRP implementation MUST NOT require globally meaningful participant addresses as a prerequisite for communication.

A PRP implementation MUST NOT require globally coordinated route allocation as a prerequisite for communication.

A PRP implementation MUST NOT require mandatory governance participation as a prerequisite for communication.

### 3.2 Continuity Before Identity

PRP requires continuity.

PRP does not require identity.

An implementation MAY support identity-bound relationships.

An implementation MAY support pseudonymous relationships.

An implementation MAY support anonymous relationships.

An implementation MUST treat identity as a relationship constraint rather than a core precondition for communication.

### 3.3 Routing Is Derived

Routing is a mechanism used to realize relationships.

A route MUST NOT define the relationship.

A route MAY change without changing the relationship.

A relationship MAY exist without an active route.

A PRP routing extension MUST preserve the distinction between relationship continuity and route realization.

### 3.4 Carrier Neutrality

A carrier moves PRP units.

A carrier MUST NOT define participant identity.

A carrier MUST NOT define relationship semantics.

A carrier MUST NOT define service authorization.

A carrier MUST NOT define application payload semantics.

A carrier binding MAY provide framing, ordering, packetization, fragmentation, retransmission, media checksums, or carrier-specific delivery behavior.

Such behavior MUST remain below PRP relationship semantics.

### 3.5 Governance Optionality

Governance systems MAY be used by PRP deployments.

Governance systems MUST NOT be required by the PRP core.

Communication remains possible whenever sufficient local relationship material exists, even if no governance system participates.

## 4. Relationship Semantics

A PRP relationship defines the semantic context in which communication occurs.

At minimum, a relationship may include:

* continuity material;
* authorization boundaries;
* trust material;
* cryptographic context;
* communication permissions;
* optional identity constraints;
* optional policy constraints;
* optional service permissions.

The PRP core does not mandate a specific relationship object format.

However, any relationship representation MUST preserve the distinction between relationship semantics and routing, transport, carrier, or service mechanisms.

### 4.1 Relationship Existence

A relationship may exist before an active transport session exists.

A relationship may exist before an active route exists.

A relationship may exist across changes in carrier, route, aggregation context, or governance service.

### 4.2 Relationship Realization

A relationship is realized when participants exchange authenticated communication under the relationship context.

Realization may involve:

* direct carrier exchange;
* adjacent PRP sessions;
* routing mechanisms;
* recursive aggregation;
* service-layer discovery;
* notarial services;
* future transport or carrier systems.

These mechanisms are not part of the relationship itself.

They are used to realize it.

## 5. Participant Semantics

A participant is any entity capable of participating in a PRP relationship.

The PRP core does not require a participant to have:

* a globally meaningful address;
* a globally meaningful identity;
* a globally reachable locator;
* a globally unique name;
* a mandatory governance registration;
* a mandatory service account.

A participant MAY have any of those properties if required by a relationship or service.

Such properties remain constraints or attributes of a relationship, service, or deployment. They are not core PRP requirements.

## 6. Continuity and Identity

Continuity is required.

Identity is optional.

A relationship may define continuity through cryptographic state, prior shared material, anonymous session continuity, pseudonymous identifiers, identity-bound credentials, or future mechanisms.

A PRP implementation MUST support the architectural possibility of relationships that do not require real-world identity.

A PRP implementation MUST NOT require state-issued identity as a core precondition for communication.

A PRP implementation MAY expose identity-bound profiles as optional relationship policies.

## 7. PRP Units

The PRP core operates on complete PRP units.

A PRP unit is a complete object passed to a PRP processing layer.

A PRP unit may be:

* a session-establishment handshake unit;
* an encrypted packet unit;
* a profile-defined unit;
* a future extension-defined unit.

The PRP core does not define byte-stream delimitation.

A carrier binding that does not preserve PRP unit boundaries MUST use an encapsulation profile that reconstructs complete PRP units before passing them to the PRP decoder.

### 7.1 Unit Boundary

A PRP decoder MUST receive exactly one complete PRP unit at a time unless a specific profile defines otherwise.

Carrier fragmentation MUST be resolved below the PRP unit decoder.

Carrier reassembly MUST NOT alter relationship semantics.

### 7.2 Packet-First Core

The base PRP core is packet-first.

This does not prohibit stream-oriented extensions.

A stream-oriented extension MUST NOT make byte-stream semantics foundational to PRP.

A reliable stream adapter MAY be defined as an extension above packet-oriented PRP units.

## 8. Session Boundary

A PRP session is an authenticated exchange context used to realize a relationship. A session may also be used for an adjacent forwarding step, but adjacency is an operational property of that step, not a special class of relationship.

A participant relationship may have one or more cryptographic sessions. A cryptographic session may carry one or more routing contexts when all routing contexts are scoped to the same participant relationship, share the same effective cryptographic policy, use the same session-level direction sequence space, share the same replay domain, and have the same session lifecycle.

Session establishment is per cryptographic session, not per route. A handshake unit is one concrete session-establishment unit; a future explicitly specified session agreement may establish a session when it provides equivalent ephemeral exchange, transcript binding, suite and profile selection, policy binding, and cryptographic confirmation.

A routing context MUST NOT reuse the same traffic keys with an independent nonce, sequence, replay, policy, carrier-binding, rekey, or lifecycle domain. If any of those domains differ, the participant MUST establish a separate cryptographic session or derive a domain-separated traffic key bound to the differing context.

A session may be direct, routed, aggregated, or carrier-mediated.

A PRP session MUST bind cryptographic profile selection into authenticated context.

A PRP session MUST NOT rely on unauthenticated carrier metadata to define relationship semantics.

A PRP session MUST NOT rely on globally meaningful address strings to become intelligible to intermediaries.

### 8.1 Cryptographic Profile Selection

Cryptographic profile selection MUST be deterministic for a given relationship or local session context.

If a profile is selected by local configuration, relationship material, carrier binding, or service-layer descriptor, both participants MUST authenticate the selected profile context before accepting the session.

A PRP implementation MUST NOT infer or negotiate a different cryptographic profile from unauthenticated carrier metadata.

A PRP implementation MUST NOT silently downgrade cryptographic profiles.

### 8.2 Adjacent Forwarding Sessions

An adjacent PRP session is an ordinary PRP session used with the next PRP participant that receives the current unit or envelope. Adjacency is operational, not a relationship class.

Adjacent forwarding sessions MAY carry opaque routed payloads.

Adjacent forwarding sessions do not imply visibility into end-to-end payload semantics.

Intermediate actors MUST NOT be required to inspect end-to-end payloads.

## 9. Carrier Boundary

A carrier adapter moves PRP units across a concrete medium.

The PRP core receives complete PRP units from carrier adapters. A carrier
adapter that receives bytes, frames, files, datagrams, symbols, or stream
fragments MUST reconstruct a complete PRP unit before passing it to the PRP
decoder.

The carrier adapter MAY use carrier-specific information internally, such as
frame size, datagram size, file position, interface state, timestamps, queue
state, or delivery errors. Such information is adapter-local. It is not PRP
relationship metadata.

A carrier adapter MUST NOT make carrier-local information part of PRP
relationship semantics.

A carrier adapter MUST NOT define participant identity, relationship identity,
service authorization, routing identity, cryptographic profile selection, or
application payload semantics.

A PRP implementation MAY expose adapter-local diagnostics for logging,
debugging, or operations, but those diagnostics MUST NOT be required for PRP
semantic correctness.

Carrier bindings MAY expose reachability to aggregation contexts during
bootstrap. Such bootstrap reachability discovery is not PRP relationship
routing.

## 10. Routing Boundary

Routing is derived from relationships.

The PRP core does not require:

* global addressing;
* global route allocation;
* centralized participant discovery;
* globally meaningful routing identifiers;
* globally privileged aggregators;
* globally meaningful aggregator addresses;
* open network-wide route discovery.

Routing mechanisms MAY be defined as extensions.

A route or routing context does not require a new cryptographic session merely because it exists. It may share an established session when it remains in the same relationship and the same session cryptographic domain.

Such mechanisms MUST preserve relationship primacy and local routing semantics.

### 10.1 Local Routing Identifiers

Routing identifiers MUST be local to the actor that interprets them.

A routing identifier MUST NOT be treated as participant identity.

A routing identifier MUST NOT be treated as a service identifier unless a higher-layer service explicitly defines such mapping.

### 10.2 Recursive Aggregation

Recursive aggregation MAY be used by routing extensions.

An aggregator is a participant that assumes aggregation responsibility within a local routing context.

Aggregators are roles, not globally privileged routing identities.

A compatible recursive aggregation implementation MUST NOT require:

* globally privileged aggregators;
* globally meaningful aggregator addresses;
* a fixed global aggregator namespace;
* route-discovery flooding through an aggregator network;
* edge computation of a complete participant-to-participant global path.

### 10.3 Route Loops

PRP routing MUST NOT be defined as unconstrained graph traversal.

Aggregation topologies MAY contain cycles.

Route loops are not valid PRP route semantics.

Diagnostic hop budgets or defensive counters MAY be used by implementations, but such mechanisms are safeguards rather than the core architectural loop-prevention mechanism.

## 11. Service Boundary

Services operate above the PRP core.

Services MAY define:

* discovery mechanisms;
* notarial services;
* governance systems;
* evidence systems;
* payment systems;
* economic settlement;
* reputation systems;
* custody systems;
* application protocols;
* object models;
* storage systems.

Services MUST NOT redefine PRP relationship semantics.

Services MUST NOT make governance participation mandatory for all PRP communication.

Services MUST NOT make participant identity mandatory unless identity is explicitly a constraint of the relationship or service.

### 11.1 Discovery Services

Discovery services MAY help participants locate relationship material, aggregation capability, descriptors, or service endpoints.

Discovery services are not part of the PRP core.

PRP core routing does not define arbitrary open route discovery.

A deployment that exposes open discovery, rendezvous, notarial lookup, public first contact, or aggregation offers SHOULD define its own abuse controls.

## 12. Conformance Requirements

A system conforms to this core semantics document if it preserves the following requirements.

### 12.1 Required Properties

A conforming PRP implementation:

* MUST treat relationships as primary architectural objects;
* MUST preserve continuity as a property of relationships;
* MUST NOT require globally meaningful participant addresses;
* MUST NOT require globally meaningful participant identities;
* MUST NOT require globally meaningful aggregator addresses;
* MUST keep carrier metadata below relationship semantics;
* MUST keep service semantics above the PRP core;
* MUST distinguish routing identifiers from participant identities;
* MUST process complete PRP units at the PRP unit boundary;
* MUST preserve the distinction between carrier movement and relationship realization.

### 12.2 Prohibited Couplings

A conforming PRP implementation MUST NOT make any of the following more fundamental than relationship continuity:

* address;
* locator;
* route;
* carrier;
* governance system;
* identity registry;
* discovery service;
* application service;
* economic system;
* evaluation system.

### 12.3 Optional Extensions

A conforming implementation MAY support:

* recursive aggregation;
* reliable stream adapters;
* notarial services;
* governance systems;
* discovery systems;
* economic systems;
* evidence systems;
* privacy profiles;
* carrier-specific optimizations.

Such extensions MUST preserve the PRP core semantics.

## 13. Security Considerations

The PRP core reduces architectural dependence on globally meaningful addresses, identities, and trusted infrastructure.

This does not make communication secure by itself.

Implementations MUST use appropriate cryptographic mechanisms for session authentication, confidentiality, replay protection, and transcript binding.

Cryptographic profile selection MUST be authenticated.

Carrier metadata MUST NOT be trusted as relationship semantics.

Routing identifiers MUST NOT be trusted as participant identities.

Services MUST NOT be assumed trustworthy merely because they are reachable.

### 13.1 Replay

Replay protection is a transport or profile responsibility.

A PRP profile that carries encrypted packet units SHOULD define nonce, replay-window, or equivalent replay protection behavior.

### 13.2 Downgrade

A PRP implementation MUST NOT silently downgrade cryptographic profiles, relationship constraints, identity constraints, or privacy policies.

### 13.3 Carrier Attacks

Carriers may be hostile.

A carrier may drop, reorder, delay, duplicate, fragment, observe, or modify carrier-level data.

PRP profiles MUST authenticate PRP units before accepting them as valid.

### 13.4 Service Attacks

Services may be hostile or compromised.

Discovery, notarial, governance, payment, or reputation services MUST NOT be treated as part of the trusted PRP core unless a specific relationship explicitly relies on them.

## 14. Privacy Considerations

PRP reduces semantic exposure through architectural decoupling.

It does not guarantee anonymity.

It does not eliminate traffic analysis.

It does not eliminate timing, size, carrier, bootstrap transport, or service metadata.

PRP implementations SHOULD minimize unnecessary exposure of:

* participant identifiers;
* relationship identifiers;
* routing identifiers;
* notarial targets;
* service targets;
* carrier-derived metadata.

Privacy profiles MAY define padding, batching, jitter, multipath, cover traffic, or other mechanisms above the PRP core.

Such profiles are optional extensions unless required by a specific relationship.

## 15. IANA Considerations

This document makes no IANA requests.

Future carrier bindings or transport profiles may request service names, port numbers, protocol numbers, media types, or other registry entries as needed.

Such requests are outside the scope of this document.

## 16. References

### 16.1 Normative References

* RFC 2119: Key words for use in RFCs to Indicate Requirement Levels.
* RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words.

### 16.2 Informative References

* Alves, G. J. Relationship-Centric Communication: The Participant Relationship Protocol (PRP). Zenodo, 2026. DOI: 10.5281/zenodo.20482932.
* Alves, G. J. Relationship-Centric Routing Without Global Addresses. Zenodo, 2026.
* PRP Architecture v1: Participant Relationship Protocol. Release Candidate 1.

