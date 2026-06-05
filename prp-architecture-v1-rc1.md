# PRP Architecture v1

## Participant Relationship Protocol

### A Relationship-Centric Communication Architecture

**Status:** Release Candidate 1
**Author:** Gustavo Junior Alves
**Date:** 2026
**Document Type:** Foundational Architecture Specification

---

# Purpose

This document defines the architectural foundations of PRP.

It describes the principles, abstractions and boundaries that characterize the architecture independently of any specific implementation, protocol version, routing mechanism, carrier technology, governance framework or application model.

This document is normative with respect to architectural intent.

Protocol specifications, implementations, services and future extensions should be interpreted in a manner consistent with the architectural principles defined herein.

---

# Scope

This document defines:

* the relationship-centric communication model;
* architectural principles;
* architectural layers;
* threat assumptions;
* design goals;
* architectural boundaries.

This document does not define:

* wire formats;
* packet structures;
* routing algorithms;
* application protocols;
* object models;
* artifact semantics;
* persistence semantics;
* lifecycle rules;
* economic systems;
* governance systems;
* benchmarking methodologies;
* custody models.

Such systems may be implemented above the architecture without modifying its foundational model.

---

# Foundational Statement

> Communication becomes possible because a relationship already exists.

PRP is based on the assumption that relationships are more fundamental than addresses, routes, transports, carriers, governance structures or participant identities.

Communication therefore emerges from relationships rather than from the ability to locate participants through globally coordinated addressing systems.

---

# Architectural Position

PRP is a relationship-centric communication architecture.

It is not:

* an address-centric architecture;
* an identity-centric architecture;
* a routing-centric architecture;
* a governance-centric architecture.

Instead, PRP treats relationships as the primary architectural object from which communication emerges.

Routing, transport, carrier selection, identity constraints and governance participation are derived concerns used to realize communication between participants.

---

# Versioning Policy

The architecture described in this document is considered stable unless a future revision modifies one or more of the following:

* the relationship-centric model;
* the architectural principles;
* the architectural layers;
* the architectural boundaries.

Changes to protocol implementations, wire formats, routing strategies, service systems or governance mechanisms do not constitute architectural revisions unless they alter the foundational model itself.

---

# 1. Introduction

## 1.1 The Address-Centric Assumption

Modern communication systems are built upon a common assumption:

Communication requires the ability to locate participants.

This assumption permeates addressing, routing, transport establishment and network operation.

Identity, location, routing and transport become tightly coupled.

While successful, this model introduces dependencies on globally coordinated infrastructure and makes communication continuity dependent upon addressing continuity.

The resulting architecture assumes that communication emerges from the ability to discover, locate and reach network endpoints.

---

## 1.2 An Alternative Perspective

PRP begins from a different premise.

Communication does not emerge from addresses.

Communication emerges from relationships.

Participants communicate because a relationship already exists.

Routes, transports and carriers merely realize that relationship.

Instead of treating addresses as primary and communication as a consequence, PRP treats relationships as primary and networking as one possible mechanism through which those relationships are realized.

---

## 1.3 Beyond Digital Networking

The relationship-centric model intentionally avoids assumptions that are specific to contemporary digital networks.

Communication existed long before digital networks.

Historically, participants established relationships, expectations, permissions and trust long before the emergence of packet switching, routing protocols or globally coordinated addressing systems.

From this perspective:

> Communication is not fundamentally a property of networks.
>
> Communication is a property of relationships.

Networks are merely one possible mechanism used to realize those relationships.

A carrier may be:

* a packet network;
* a stream transport;
* removable media;
* a physical logistics system;
* a delay-tolerant communication path;
* an intermittent communication channel;
* a future communication technology.

The relationship remains unchanged.

Only the mechanism used to realize communication evolves.

The architecture therefore seeks to separate communication from assumptions introduced by specific network technologies.

PRP is not intended to redefine communication.

Instead, it attempts to recover communication as a relationship-centered concept and treat networking as one possible implementation of that concept rather than its defining characteristic.

---

# 2. Terminology

## Participant

A participant is an entity capable of participating in a relationship.

The architecture does not impose any particular participant model.

Participation in a relationship is the only architectural requirement.

---

## Relationship

A relationship is a persistent communication contract between participants.

A relationship defines:

* continuity constraints;
* identity constraints;
* trust material;
* authorization boundaries;
* communication permissions.

Relationships are the primary architectural objects of the system.

---

## Continuity

Continuity is the ability to determine whether communication is occurring within the same relationship context across time.

Continuity may be:

* anonymous;
* pseudonymous;
* identity-bound.

The architecture requires continuity.

The architecture does not require identity.

---

## Identity Constraint

An identity constraint is a relationship policy that defines identity requirements.

A relationship may:

* require identity;
* permit identity;
* operate without identity.

Identity is therefore a property of a relationship rather than a prerequisite for communication.

---

## Trust Material

Trust material consists of information used by a relationship to establish continuity and communication permissions.

Trust material exists to support relationships rather than to define identity systems.

---

## Authorization Boundary

An authorization boundary defines the set of actions permitted within a relationship.

Authorization is a property of relationships.

---

## Route

A route is a communication trajectory used to realize a relationship.

Routes are operational mechanisms.

A relationship may exist without an active route.

A route may change without affecting the relationship.

---

## Routing Identifier

A routing identifier is an operational label used by routing systems.

Routing identifiers are local to the entities that interpret them.

Routing identifiers are not participant identities.

---

## Transport

Transport is the authenticated exchange of information between participants.

Transport exists to realize relationships.

A participant relationship may be realized by one or more cryptographic sessions. A cryptographic session may carry one or more routing contexts when those contexts share the same effective cryptographic policy, direction sequence space, replay domain, and session lifecycle. Session establishment is per cryptographic session, not per route.

---

## Carrier

A carrier is any mechanism capable of moving authenticated communication data.

Carriers are implementation details.

Carrier selection does not alter relationship semantics.

---

## Evidence

Evidence is a verifiable observation of communication activity.

Evidence may be shared.

Interpretation remains local.

---

## Evaluation

Evaluation is the interpretation of evidence.

The architecture does not define globally authoritative evaluation systems.

---

## Governance System

A governance system is an optional coordination service operating above the core architecture.

Participation in governance systems is optional.

---

## Service

A service is a capability implemented above the core architecture.

Services extend the architecture.

They do not define it.

---

# 3. Core Thesis

## 3.1 Relationship-Centric Communication

Traditional communication architectures are primarily organized around locations.

PRP adopts a different model.

Communication does not emerge from addressing.

Communication emerges from relationships.

The primary architectural object is a relationship.

Relationships define the continuity required for communication.

Routing, transport and carriers exist to realize that continuity.

Address-centric architectures:

```text
Address
    ↓
Route
    ↓
Transport
    ↓
Communication
```

Relationship-centric architectures:

```text
Relationship
      ↓
Continuity

Route
Transport
Carrier

(realize the relationship)
```

Relationships may:

* exist without active routes;
* exist without participant identity;
* survive route changes;
* survive carrier changes;
* survive governance changes.

---

## 3.2 Continuity Before Identity

The architecture requires continuity.

The architecture does not require identity.

Relationships may establish continuity through:

* anonymous continuity;
* pseudonymous continuity;
* authenticated continuity.

Identity becomes one possible policy of a relationship rather than a prerequisite for communication.

Relationships are therefore more fundamental than identities.

---

## 3.3 Governance Independence

Governance systems may provide valuable services.

They do not create communication.

Communication remains possible whenever sufficient local relationship information exists.

Governance therefore becomes optional infrastructure rather than a mandatory architectural foundation.

---

## 3.4 Formal Definition

A communication architecture is relationship-centric when communication remains possible through pre-existing relationships independently of:

* participant location;
* routing topology;
* carrier technology;
* governance participation.

In a relationship-centric architecture:

* relationships are primary architectural objects;
* continuity is a property of relationships;
* identities are optional constraints imposed by relationships;
* routes are mechanisms used to realize relationships;
* transports are mechanisms used to exchange authenticated information;
* carriers are mechanisms used to move communication data.

---

## 3.5 Relationship-Centric Test

An implementation remains compatible with the PRP architecture if communication remains possible after the removal of:

* global addressing;
* global route allocation;
* centralized participant discovery;
* mandatory governance participation;

provided that sufficient local relationship information remains available.

## Recursive Aggregation Test

Aggregators are recursive roles. An aggregator is a participant that assumes
aggregation responsibility within a local routing context. The same actor may be
an aggregator for one relationship, a normal participant relative to another
aggregator, an ingress aggregation role in one route, a transit aggregation role
in another route, and an egress aggregation role in another route.

PRP does not replace global participant addresses with global aggregator
addresses. It replaces global addressing with recursively delegated aggregation
responsibility. Participants may select, authorize, or request ingress and egress
aggregation contexts, but they are not required to compute a complete global path
between all aggregation roles. Each aggregation level performs local admission,
local opaque-label handling, and local delegation to another relationship when
needed.

A compatible routing implementation therefore must not require:

* globally privileged aggregators;
* globally meaningful aggregator addresses;
* a fixed global aggregator namespace;
* route-discovery flooding through an aggregator network;
* edge computation of a complete participant-to-participant path.

The expected routing model is:

```text
Participants choose or authorize aggregation contexts.
Aggregators recursively delegate aggregation responsibility.
Route agreements bound and validate the resulting forwarding graph.
Dataplane forwarding uses local opaque labels only.
No participant or aggregator requires a globally meaningful destination address.
```

## Inter-Aggregator Batch Transfer Test

Inter-aggregator transfer is packet-first by default. Aggregators do not need to
preserve a one-to-one mapping between incoming routed payloads and outgoing
adjacent PRP packets. Immediate forwarding is a valid low-latency policy, but it
is not a base architectural requirement.

A compatible aggregation implementation may retain, batch, pad, shape,
multiplex, or reencapsulate multiple routed payloads into an inter-aggregator
batch envelope carried over an ordinary adjacent PRP packet session. The batch
envelope is adjacent-session payload, not a base PRP packet profile unless a
future profile explicitly defines it as such.

The architecture must keep these objects distinct:

* routed payload;
* inter-aggregator batch envelope;
* adjacent PRP packet;
* carrier packet or carrier unit.

Reliable stream transfer is optional and applies only when the selected transfer
policy requires ordered byte delivery, retransmission, flow control, sustained
backpressure, or virtual-frame preservation. PRP routing must not become
stream-first.

---

# 4. Architectural Principles

* Relationship First
* Continuity Before Identity
* Identity Is Not Location
* Routing Is Derived, Not Fundamental
* Carrier Neutrality
* Governance Optionality
* Local Utility
* Evidence Over Reputation
* Privacy Through Architectural Decoupling
* Architectural Minimalism

---

# 5. Architectural Consequences

The relationship-centric model produces several consequences.

## Mobility Independence

Relationships survive route changes.

## Carrier Independence

Relationships survive carrier changes.

## Governance Independence

Relationships survive governance changes.

## Routing Flexibility

Multiple communication trajectories may realize the same relationship.

## Privacy Emergence

Privacy may emerge from routing diversity, architectural separation and local utility functions without requiring any specific privacy mechanism.

## Infrastructure Resilience

Communication remains possible despite changes in infrastructure ownership, routing topology or transport technology.

## Anonymous Relationships

Relationships may exist without persistent participant identity.

## Pseudonymous Relationships

Relationships may operate through stable pseudonyms without requiring disclosure of real-world identities.

---

# 6. Architectural Layers

## Relationship Layer

Defines communication contexts.

Answers:

> Who is allowed to communicate?

## Routing Layer

Defines communication trajectories.

Answers:

> How does communication move?

## Transport Layer

Defines authenticated exchange of information.

Answers:

> How is information exchanged securely?

## Carrier Layer

Moves communication data.

Answers:

> How are bytes physically moved?

## Service Layer

Provides specialized capabilities above the architecture.

Answers:

> What is the communication for?

---

# 7. Threat Model

The architecture assumes:

* compromised participants;
* hostile transit infrastructure;
* institutional adversaries;
* state adversaries;
* global passive observers.

The objective is not to eliminate observation.

The objective is to reduce architectural dependence on trusted infrastructure while increasing resilience, routing flexibility and communication autonomy.

---

# 8. Design Goals

* Decouple Identity From Location
* Decouple Routing From Identity
* Decouple Transport From Carrier
* Operate Across Intermittent Networks
* Minimize Infrastructure Trust Assumptions
* Preserve Governance Optionality
* Support Local Decision Making
* Preserve Evidence
* Enable Routing Diversity
* Support Privacy Through Separation
* Remain Extensible

### Architectural Success Criterion

PRP succeeds when communication remains possible despite changes in:

* location;
* topology;
* carrier technology;
* routing strategy;
* infrastructure ownership;
* governance frameworks.

---

# 9. Non-Goals

PRP does not define:

* application semantics;
* application object models;
* artifact semantics;
* persistence semantics;
* lifecycle rules;
* economic settlement;
* reputation systems;
* utility functions;
* custody systems;
* benchmarking systems;
* routing policy;
* privacy guarantees;
* mandatory governance systems.

---

# 10. Architectural Conformance

## Core Components

* Relationship Layer
* Identity Constraints
* Continuity Model
* Routing Layer
* Transport Layer
* Carrier Layer

## Core Extensions

* Reliable Transport Extensions
* Stream-Oriented Extensions

## Service-Layer Components

* Discovery Systems
* Governance Systems
* Evidence Systems
* Benchmarking Systems
* Economic Systems
* Custody Systems
* Advanced Routing Systems

### Architectural Rule

A component belongs to the core only if:

1. It is required by every valid deployment.
2. It cannot be implemented as a service.
3. Its absence would invalidate the relationship-centric model itself.

---

# 11. Protocol Conformance Notes

This architecture does not define byte-level wire formats. Implementations that
claim conformance to the current PRP v1 protocol family should nevertheless
preserve the following architectural constraints:

* cryptographic suite selection is deterministic and bound to the session
  transcript;
* compact numeric suite identifiers are protocol selectors;
* textual suite names are diagnostic aliases only;
* textual suite names must not be required carrier metadata;
* a relationship-selected session must not rely on globally meaningful address
  or suite strings to become intelligible to intermediaries;
* PRP operates on complete PRP units: a session-establishment handshake
  unit or an encrypted packet unit;
* session establishment is per cryptographic session, not per route;
* a compatible cryptographic session may carry multiple routing contexts only
  when they share the same effective cryptographic policy, direction sequence
  space, replay domain, and session lifecycle;
* the base protocol does not define byte-stream delimitation, carrier
  fragmentation, media checksums, physical retransmission, or
  resynchronization markers;
* carrier bindings that do not preserve PRP unit boundaries must use an
  encapsulation profile that reconstructs exactly one complete PRP unit before
  passing it to the PRP decoder;
* encapsulation profiles must not define PRP relationship semantics, peer
  identity, route identity, service authorization, packet replay rules,
  cryptographic suite negotiation, or application payload semantics.

The practical consequence is that a PRP packet, stream, carrier or bootstrap
binding may inherit framing context from lower layers, but the cryptographic
profile selected for the session must still be explicitly bound into the
authenticated transcript.

---

# 12. Future Directions

Future systems may define:

* economic relationships;
* evidence publication systems;
* benchmarking systems;
* custody systems;
* predictive routing;
* temporal routing;
* delay-tolerant routing;
* governance services;
* future communication domains.

Such systems remain compatible with the architecture while remaining outside the core model.

---

# 13. Conclusion

Communication systems have historically been organized around addresses.

PRP proposes a different model.

Communication emerges from relationships.

Relationships define continuity independently of routes, transports, carriers, governance systems and participant identities.

Routing, transport and carrier selection become operational concerns derived from existing relationships rather than architectural prerequisites.

The architecture favors:

* relationships over addresses;
* continuity over identity;
* local coordination over global coordination;
* evidence over centralized evaluation;
* optional governance over mandatory governance;
* architectural separation over architectural coupling.

The central architectural assumption is simple:

> Communication becomes possible because a relationship already exists.

Everything else follows from that principle.

---

**End of Document**
**PRP Architecture v1 — Release Candidate 1**
**Participant Relationship Protocol**
