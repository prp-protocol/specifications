# PRP Directional Discovery Policy Version 1

- **Version:** 1
- **Status:** Working Draft
- **Category:** Relationship control
- **Date:** 2026

## 1. Scope

This document assigns the `DISCOVERY_POLICY` Relation Control section and
defines deterministic bilateral state for one discovery direction. It does
not change wire-v4 framing, HS2, or the discovery algorithm registries.

## 2. Assignment and body

`DISCOVERY_POLICY` has section type `0x0006`, section version `0x01`, no
section flags, operation `PROPOSE`, and persistent-directional state. Its body
is exactly eight octets:

| Offset | Width | Field |
| ---: | ---: | --- |
| 0 | 1 | direction (`0x01` initiator-to-responder, `0x02` responder-to-initiator) |
| 1 | 1 | reserved, zero |
| 2 | 2 | discovery index algorithm, unsigned big-endian |
| 4 | 2 | discovery query algorithm, unsigned big-endian |
| 6 | 2 | discovery profile, unsigned big-endian |

Direction is fixed by the canonical roles of the relationship's initial HS2
and does not change across session rekey. The all-zero selector tuple means
disabled. Any tuple containing both zero and nonzero selectors is malformed.
A nonzero tuple is admissible only when every selector and the exact tuple are
available in `registries/discovery-policy-v1`.

## 3. Instance and protection

The instance key is the authenticated relationship identity and epoch plus
direction. The section MUST be carried through
authenticated Relation Control in that relationship. Both registered `NONE`
session suites (`0x0001` and `0x0002`) MUST reject proposals, results,
restored pending state, and lane activation. Protection supplied by a carrier
does not satisfy this requirement.

## 4. Generation and ordinary transitions

The implicit initial effective value is disabled at generation zero. A new
proposal MUST set `base_generation` to the effective generation and
`generation` to exactly `base_generation + 1`. Generation wrap is not
defined; no further proposal may be made at `0xffffffff`.

Before emission, a proposer durably reserves its transaction identifier,
base, successor generation, direction and body. With no crossed local
reservation, a conforming receiver either rejects the exact proposal or
durably installs it before emitting `ACCEPTED`. The proposer installs only
after authenticating that result. `ACCEPTED` carries the complete effective
body. `STALE` carries the current effective body and lineage; `REJECTED`
carries no body, as required by the generic Relation Control state machine. An
exact retransmission returns the cached result; reuse of a transaction
identifier with different bytes is rejected. Skipped generations and
incorrect bases receive `STALE` and change no state.

## 5. Crossed proposals

A crossing exists only when both endpoints durably reserved proposals for the
same instance, base and successor before receiving the peer proposal. It is
resolved without arrival-order preference:

1. Equal bodies coalesce into one accepted successor.
2. Different non-disabled bodies are both rejected and the effective
   generation does not advance.
3. If either body is disabled, disabled dominates and both endpoints install
   disabled at the successor generation. In this crossed case alone, the
   accepted effective body may differ from the local enabled proposal.

The rule is commutative. Proposals for opposite directions are independent.
There is no counterproposal, lexicographic winner, role preference or timeout
fallback. A valid disable proposal immediately suspends new lane admission
while its result is pending.

## 6. Rekey, restart and revocation

Session rekey preserves effective policy and pending durable transactions.
Restart restores the effective tuple, generation, reservations and cached
results before emission; silence is not acceptance. Relationship deletion
deletes all policy state. An accepted disabled successor revokes the lane and
new work immediately; re-enablement requires the next exact successor.

## 7. Conformance

Conformance MUST cover both directions, exact acceptance and rejection,
malformed mixed tuples, both `NONE` suites, duplicates, transaction reuse,
stale and skipped generations, every crossed rule in both operand orders,
disable and re-enable, rekey, restart and relationship deletion. Canonical
evidence is fixed by `vectors/discovery-policy-v1/manifest.sha256`.
