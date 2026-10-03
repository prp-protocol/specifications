# PRP Directional Carrier Semantics

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document defines how PRP represents local carrier direction, separates
carrier capability from configured intent and operational readiness, treats
carrier-member observations as non-authoritative evidence, and composes
authenticated PRP relationships from independently selected transmit and
receive paths. It defines no carrier-specific interface or encapsulation.

## 1. Status and requirements language

This document is a Working Draft. Its direction, evidence, bootstrap,
composition and failure rules are covered by deterministic semantic vectors.
Individual carrier bindings remain separate specifications and may impose
additional scheduling and framing constraints without changing these rules.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Scope and model

A **carrier** transports boundary-preserving units between operational
attachments. A **carrier member** is an observed or addressable peer-side
operational selector within one local carrier context. A **path** is admitted
only after PRP authentication binds such operational context to a participant
identity and direction.

Carrier state can establish physical reachability. It does not by itself
establish identity, a relationship, a session, a route, a trajectory,
publication authority or application-service authority.

This specification defines abstract state and behavior. Carrier-specific
handles, locators, attachment identifiers, peer contexts, readiness signals,
event delivery and configuration interfaces are outside its wire semantics
and MUST NOT appear in PRP protocol records unless a separate versioned
specification explicitly assigns them.

## 3. Local direction

Every direction in this document is stated from the observing participant's
point of view:

| Direction | Meaning |
| --- | --- |
| `TX` | the local participant can transmit toward the selected remote member |
| `RX` | the local participant can receive from the selected remote member |
| `TX_RX` | both properties are available, independently |

`TX_RX` is the set union of `TX` and `RX`; it does not assert that both can be
used simultaneously. Full-duplex, half-duplex and simplex operation are
carrier scheduling properties. A half-duplex carrier may expose `TX_RX` while
its instantaneous readiness permits only one operation.

No local direction implies the corresponding remote direction. In
particular, local `RX` is not evidence of local `TX` or remote `RX`.

## 4. Three independent state layers

### 4.1 Structural capability

A carrier declares the nonempty subset of `TX` and `RX` that it can support in
the current local context. Capability is structural. It does not assert that
an attachment is configured, ready, reachable or authenticated.

### 4.2 Endpoint intent and readiness

Configured endpoint intent is a subset of structural capability. A request
that would widen intent beyond capability MUST fail without changing state.
Intent is durable protocol policy and is reconciled idempotently.

Readiness is volatile and generation-bearing. `TX` readiness means that the
endpoint can currently accept a unit for transmission within its advertised
limits. `RX` readiness means that it can currently admit a received unit. It
does not claim that a remote member is sending.

Readiness MUST be a subset of both configured intent and structural
capability. Readiness change cannot widen intent. Loss of one readiness
direction preserves the other when it remains valid.

### 4.3 Member evidence

A carrier member exposes two independent evidence properties:

| Evidence | Meaning |
| --- | --- |
| `TX_ADDRESSABLE` | the carrier can select this member for local transmission |
| `RX_OBSERVED` | a complete carrier unit attributed to this member was observed |

A member with neither property is invalid and MUST NOT be enumerated.
`TX_ADDRESSABLE` does not imply observation, availability, identity or a
response. `RX_OBSERVED` does not imply addressability, identity or a reverse
path. Equal-looking locators do not permit two member entries or directions to
be merged.

## 5. Volatile selectors and generations

A carrier member is selected by the conceptual tuple:

```text
carrier_context || member_handle || member_generation || local_direction
```

The handle, nonzero generation, locator, attachment and pre-authentication
context are local, volatile and opaque. They MUST NOT be used as PRP identity
or authorization. A stale generation MUST NOT be resolved or promoted.

Member events are notifications that state may have changed. A participant
MUST reconcile authoritative current state before acting on an event. Events
MUST NOT be treated as durable state or ordered mutation commands.

Configured carrier and endpoint intent MAY survive restart. Observations,
locators, readiness, member handles, generations and pre-authentication
contexts MUST be reacquired. Persisted authenticated relationship policy may
request directions but cannot restore an operational path without current
carrier evidence and authentication.

## 6. Bootstrap and HS2

Carrier enumeration produces only physical bootstrap candidates. A candidate
for sending M1 requires current `TX_ADDRESSABLE` evidence, `TX` intent and `TX`
readiness for the exact member generation. Enumeration MUST NOT publish or
infer a PRP identity.

Public adjacent HS2 is the identity proof for carrier bootstrap. Only a
successful proof of possession for the expected identity can promote the
operational candidate into an authenticated path.

Receiving M1 on an `RX` path does not authorize M2 on that path. The responder
selects an independently current and authorized `TX` path and binds it to the
same HS2 exchange and remote identity. If no return path is available, the
exchange remains pending within bounded policy or fails. It MUST NOT invent
reverse reachability.

Public `ADVERTISE` is not a bootstrap source. Discovery records available
inside an already established protected context do not authorize
pre-authentication carrier promotion.

## 7. Relationship composition

One authenticated logical relationship may use the same carrier path in both
directions or different carriers, attachments, members and operational
contexts:

```text
local TX path  -> remote RX path
local RX path  <- remote TX path
```

A bilateral relationship becomes operational only when it has at least one
authenticated `TX` path and at least one authenticated `RX` path. A
unidirectional relationship is valid only when the negotiated service and
local policy explicitly admit that direction.

Relationship and session identity, generation, replay state and connection
state are logical state shared across their authorized paths. An ingress
operational context MUST NOT be inferred to be an egress selector.

Acknowledgments, feedback, retransmission and relationship control use an
authenticated reverse path whenever their defining specification requires
one. That path need not use the same carrier or member as the forward path.

## 8. Handover and directional failure

Make-before-break replacement installs and authenticates the replacement in
its own direction before draining the old path.

Loss of the final `TX` path makes `TX` unavailable while preserving valid
`RX` state. Loss of the final `RX` path has the complementary effect. The
relationship closes only when its negotiated policy requires bilateral
availability or no useful admitted direction remains.

A readiness or member-generation change invalidates selections that depend on
the old state. It does not silently revoke an independently valid opposite
direction or an unrelated authenticated path.

## 9. Error behavior

Zero direction, zero member generation, evidence with neither property,
intent beyond capability, readiness beyond intent, stale member generation,
unsupported requested direction, unauthenticated promotion, inferred reverse
path and premature path replacement fail without creating identity,
relationship, session, route, trajectory or publication state.

Failure in one direction MUST NOT be converted into success by relabeling the
opposite direction or reusing an ingress-only selector.

## 10. Security and privacy considerations

Carrier capability, intent, readiness, locators and member evidence are
operational facts, not cryptographic claims. Treating observation as identity
or reverse reachability enables impersonation, reflection and amplification.

Generation checks prevent stale operational selectors from being promoted
after member replacement. Independent direction checks prevent an attacker
who can inject an ingress unit from choosing the response path.

Direction metadata exposed outside the carrier boundary SHOULD contain no
remote PRP identity, application service, durable relationship identifier or
topology beyond what the local operation requires.

## 11. Conformance

`vectors/directional-carrier-v1` defines deterministic cases for capability,
intent, readiness, member evidence, stale generations, asymmetric HS2,
bilateral and unidirectional composition, reverse control, restart and
make-before-break behavior.

Conformance does not assign a carrier-specific interface, locator format,
encapsulation or protected discovery service.

## 12. IANA considerations

This document currently requests no IANA action.

## 13. References

### 13.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Architecture*, version 1.
- *PRP Wire Protocol Version 4*.
- *PRP Relationship Lifecycle and Continuous Control*, version 1.

### 13.2 Informative references

- *PRP Discovery Architecture*, version 1.
