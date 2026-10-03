# PRP Authenticated Member Route Intent

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies an authenticated adjacent relation-control operation
by which one PRP member requests bilateral scoped route decisions toward an
exact other authenticated member identity. Those local decisions can
participate in a recursive trajectory. This document defines the request and
result records, bilateral activation preconditions, alias authority, recursive
HS2 boundary, replay behavior and failure semantics.

## 1. Status and requirements language

This document is a Working Draft. Canonical request and result bytes and
semantic rejection cases are covered by deterministic vectors. General
`ACCEPTED` emission remains unavailable until `ROUTE-001` defines reciprocal
generation coordination, deferred-result convergence and alias lifetime.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Model and scope

A **source member** is the authenticated peer that sends the request over an
active adjacent session. A **target member** is another active adjacent peer
whose authenticated identity exactly matches the request. The participant
receiving both adjacent sessions is the **intermediary**.

The operation requests routing authority for a recursive direct or E2E HS2
exchange. It does not authenticate the recursive endpoint, establish an E2E
session, publish reachability, reveal topology or grant application-service
authority.

An external carrier, attachment or gateway is not a participant in trajectory
realization. Carrier locators and operational selectors remain outside the
control.

## 3. Relation-control profile

Member route intent is one wire-v4 relation-control record with:

| Property | Required value |
| --- | --- |
| authenticated session scope | adjacent |
| outer route alias | control alias `0xffff` |
| section count | one |
| section type | `0x0100` |
| section version | `0x01` |
| section flags | `CRITICAL=0x01` |
| request operation | `PROPOSE=0x02` |
| result operation | `RESULT=0x04` |
| base generation | zero |

The transaction ID and section generation are nonzero. No other section may
share this control. Unknown version is `UNSUPPORTED`; malformed framing fails
closed.

Section `0x0100` is transaction-scoped and nonpersistent. Its body identifies
one requested target. Its generation is an intent-generation discriminator,
not a persistent relation-section lineage or route-alias lifecycle generation.
`ROUTE-001` must define how reciprocal members select a common discriminator
before general activation.

## 4. Request body

The request body is:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 2 | target identity suite |
| 2 | 2 | target Strong-ID length |
| 4 | variable | exact target Strong ID |

Integers are unsigned big-endian. The identity suite is registered for wire
v4 and fixes the Strong-ID length. The body has no trailing bytes.

The complete control length is `40 + strong_id_length` bytes and is currently
at most 88 bytes. The request contains no source identity, source selector,
session handle, peer context, path handle, trajectory handle, service, carrier
locator or requested route alias.

The receiver derives the source identity exclusively from the authenticated
adjacent session that delivered the control. A body-supplied or externally
asserted source identity MUST NOT participate in admission.

## 5. Exact target resolution

The target identity-suite and Strong-ID pair is matched only against active,
authenticated adjacent sessions. Equality of locator, member label, local
handle, weak alias or public advertisement is insufficient.

Several authenticated paths MAY represent the same exact target identity.
Path selection is local and bounded but MUST NOT weaken exact identity
matching. The source and target adjacent contexts must be distinct.

An absent or not-yet-active source or target produces `DEFERRED` while bounded
policy permits retry. A request targeting the authenticated source itself is
`REJECTED`.

## 6. Bilateral intent and activation

An intermediary MUST NOT create cross-member routing authority from one
member's unilateral assertion alone. `ACCEPTED` requires reciprocal intent:

- the source requests the exact target identity;
- the target requests the exact source identity;
- both requests use the common intent generation;
- both adjacent identities and sessions remain active;
- the same nonreserved recursive route alias is collision-free in both
  directions; and
- both directional route decisions are installed and observable as active
  before either result is accepted.

Installation is atomic as a pair. Failure before both directions are active
removes any staged direction and returns no alias authority. A pending request
may remain `DEFERRED` under bounded policy.

The method for selecting the common intent generation, correlating the two
transactions and converging previously returned `DEFERRED` results is open
under `ROUTE-001`. Until it is closed, the conditional `ACCEPTED` format is
specified for parsing and testing but MUST NOT be advertised as generally
interoperable activation.

## 7. Results

Every result repeats the request's exact transaction ID, section type,
version, flags, intent generation and base generation zero.

Allowed statuses and bodies are:

| Status | Body | Meaning |
| --- | --- | --- |
| `ACCEPTED=0x02` | recursive route alias, `u16be` | bilateral route-decision authority is active |
| `DEFERRED=0x03` | empty | required authenticated or reciprocal state is not yet available |
| `REJECTED=0x04` | empty | policy, self-target, collision, conflict or terminal admission failure |
| `STALE=0x05` | empty | intent generation is no longer admissible |
| `UNSUPPORTED=0x06` | empty | section version or operation is unavailable |

The accepted alias is neither zero nor `0xffff`. No other status or body is
valid for this section version. In particular, the alias body is a
transaction-specific result, not the effective request body described for
persistent relation policy.

## 8. Alias authority and recursive HS2

An accepted alias is receiver-interpreted routing authority scoped to the
adjacent session generation that returned it, the member pair and the accepted
intent generation. It is not a participant, identity, service, relationship,
path or carrier locator.

This assignment is specific to section `0x0100`; it does not activate or renew
the generic route-alias lifecycle. The intermediary MUST already have
installed the bilateral route decisions before returning `ACCEPTED`.

Recursive HS2 is carried through the accepted alias using the scoped HS2
mechanism defined by wire v4. The requester MUST require the recursive HS2
identity proof to equal the exact target from its request. The intermediary
forwards the protected exchange but does not install an endpoint identity
expectation on behalf of the requester.

An accepted alias therefore permits an authentication attempt. It never
substitutes for the recursive endpoint proof.

## 9. Correlation and replay

The request identity is:

```text
source adjacent relationship epoch || source adjacent session generation ||
transaction_id
```

An exact byte-for-byte retransmission is idempotent and returns the current
canonical result for that request. A `DEFERRED` request may become eligible for
a later result only according to the convergence rule required by
`ROUTE-001`. Reuse of the request identity with different generation, target
or bytes is a conflict and MUST NOT mutate route state.

Transactions, pending requests, result cache and aliases are bounded. Timeout
or cancellation never implies acceptance. A retry after a terminal result or
expired transaction uses a fresh transaction ID.

## 10. Lifetime, handover and failure

Route-decision authority is operational, not durable relationship policy. Loss,
replacement or reauthentication of either adjacent session invalidates every
member-route alias whose authority depends on that incarnation. Restart MUST
remove such route state before accepting new requests.

If another authenticated path to the same exact identity is available, the
intermediary MAY perform make-before-break replacement while preserving the
accepted member pair, alias and intent generation. The replacement becomes
eligible only after authentication and activation in both directions.

Loss of the final eligible source or target path invalidates both directions.
Delayed traffic under the old alias fails closed. Alias collision, partial
installation, reciprocal loss and reconciliation ambiguity remove staged
state rather than selecting an unrelated identity or alias.

`ROUTE-001` must define an interoperable explicit revocation or bounded
lifetime before general activation. Local cleanup alone is not a portable
revocation protocol.

## 11. Error behavior

Unauthenticated carriage, a `NONE` suite, wrong scope, non-control outer alias,
multiple sections, wrong type or version, missing critical flag, zero
transaction or generation, nonzero base generation, invalid identity suite,
wrong Strong-ID length, trailing bytes, invalid result status, forbidden result
body and reserved accepted alias fail without changing route state.

Failure MUST NOT fall back to a public request, carrier identity, weak alias,
unauthenticated routing or recursive HS2 without an exact target expectation.

## 12. Security and privacy considerations

The request exposes the exact target identity to the authenticated
intermediary. It is not a private discovery mechanism. Rate, pending-state,
trajectory and alias bounds are required to resist authenticated resource
amplification.

Reciprocal intent prevents one attached member from unilaterally installing a
route decision involving another member. Exact recursive HS2 prevents a
compromised or mistaken intermediary from substituting a different endpoint
behind an accepted alias.

Alias and transaction scoping prevent cross-session reuse. Alias derivation
MUST NOT expose local handles, create a portable identity claim or permit a
predictable collision attack.

## 13. Conformance

`vectors/member-route-intent-v1` fixes canonical request and all permitted
result forms, structural rejection cases and conditional activation semantics.

Passing the codec vectors demonstrates wire interoperability only. A claim of
activation conformance is unavailable while `ROUTE-001` remains open.

## 14. IANA considerations

This document currently requests no IANA action.

## 15. References

### 15.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Wire Protocol Version 4*.
- *PRP Relationship Lifecycle and Continuous Control*, version 1.
- *PRP Directional Carrier Semantics*, version 1.

### 15.2 Informative references

- *PRP Bootstrap Candidates and Reachability Domains*, version 1.
