# PRP Relationship Retention

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies bounded, directional retention of reliable PRP items
across path loss, rekey and delayed contact. It defines policy intersection,
admission, retained-item identity, end-to-end feedback, release conditions,
cryptographic limits, handover, drain and failure behavior. Retention is
replicated recovery state; it is neither exclusive custody nor proof of
application acceptance.

## 1. Status and requirements language

This document is a Working Draft. Deterministic vectors cover policy
intersection, bounded admission, feedback and lifecycle outcomes. The
authenticated relation-control encoding that carries and activates retention
policy is not assigned. General policy activation therefore remains
unavailable under `RET-001`; implementations MUST NOT infer a section type,
service or HS2 field.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** are interpreted as described by BCP 14 when they appear in all
capitals.

## 2. Scope

Version 1 retention applies to complete E2E items on a wire-v4 connection
negotiated as `ORDERED|RELIABLE`. It preserves independently identifiable work
while a session, trajectory or carrier path is replaced or temporarily
unavailable.

Retention does not:

- create a relationship, session, connection or path;
- extend adjacent-fragment reconstruction timeouts;
- reserve a transmission rate or promise eventual reachability;
- transfer exclusive custody to an adjacent participant;
- report destination application processing; or
- introduce an identity-encrypted asynchronous envelope.

Application acceptance, E2E receipt and adjacent recoverability are distinct
events. A profile that needs application-level completion defines it above this
document and MUST NOT reinterpret protocol feedback as that completion.

## 3. Directional retention policy

Each participant offers this complete policy independently for each local
transmit direction:

```text
retention_ms:u64 || max_items:u32 || max_bytes:u64
```

The complementary receive offer from the peer applies to the same
originator-relative direction. All three unsigned integer fields are
mandatory. If either offer has zero in any field, retention is disabled for
that direction and all three effective fields are zero. Otherwise each
effective field is the unsigned minimum of the two corresponding offers.

The effective policy is durable relationship state. It is scoped by the exact
relationship, service and direction and is independent of session handles,
session generations, trajectories, route aliases, carrier locators and local
execution identifiers. The policy does not imply an obligation in the reverse
direction.

Policy replacement or disablement uses an authenticated, monotonic transition.
It affects new admission only. Already admitted items retain the expiry and
limits accepted at admission unless an explicit terminal outcome permitted by
Section 11 occurs. The missing encoding, generation and confirmation rules are
tracked by `RET-001`.

## 4. Admission and accounting

An item is admitted only if retention is enabled and adding it would leave
both:

```text
retained_item_count <= effective_max_items
retained_byte_count <= effective_max_bytes
```

For wire v4, one retained item's byte count is the length of its complete
canonical E2E item from `connection_alias` through the end of `body`. Adjacent
AOP, cryptographic expansion, fragments, retransmissions and storage metadata
do not change this protocol byte count. An incomplete item or fragment is not
independently admissible.

Admission is atomic. Refusal or backpressure occurs before the item identifier
or E2E sequence is consumed. Once admitted, ordinary resource pressure MUST
NOT silently discard the item before a release condition. Implementations
MUST bound non-item metadata separately and refuse new work before either the
negotiated obligation or a stricter local resource bound would be violated.

Retention duration starts at admission. Arithmetic that computes expiry or
aggregate bytes MUST reject overflow rather than wrap. Duration alone does not
reserve bandwidth, contact opportunities or replica count.

## 5. Retained item and identity

The retained item is the complete canonical E2E item above adjacent wrapping
and fragmentation. Its stable identity is:

```text
relationship epoch || service || direction || connection generation ||
connection alias || item sequence
```

Every component is derived from authenticated protocol state. The identity
MUST NOT include a session handle, peer context, trajectory, route alias,
carrier locator, path handle, process identifier, descriptor or credential.

The payload bytes and stable identity are immutable after admission. A
retransmission reuses the item identity but applies protection and
fragmentation appropriate to the selected current path. It MUST NOT reuse an
AEAD nonce or replay an adjacent ciphertext wrapper tied to an earlier path.

Replay state at the receiver is scoped to the same relationship, service,
direction and connection generation. A retained retransmission MUST NOT cause
duplicate application delivery.

A protected control checkpoint MAY be retained when it is necessary to
reconcile admitted items or prevent duplicate delivery. It is scoped by the
exact relationship, control type, direction and logical generation. It is not
an application item, does not consume an E2E item sequence and is bounded as
non-item metadata under Section 4. A checkpoint MUST NOT contain or restore a
volatile path selector or obsolete cryptographic session.

## 6. Replica and custody semantics

Retention creates one or more recoverable replicas; it does not move a unique
object between custodians. Adjacent acknowledgment states only that the
adjacent participant currently has the recoverable state promised by the
selected adjacent profile. It:

- does not acknowledge destination E2E receipt;
- does not acknowledge application acceptance;
- does not authorize the origin to release its independent copy; and
- does not transfer retransmission or expiry authority exclusively.

Loss of an intermediary replica does not manufacture E2E feedback and does not
release the origin. An intermediary that retains an object learns no protocol
authority beyond the authenticated metadata required to expire, deduplicate
and forward that object.

## 7. E2E feedback checkpoint

Retention uses wire-v4 connection control `FEEDBACK`. Its
`connection_alias` and `generation` bind the checkpoint to the reliable
connection. `next_sequence` is the smallest sequence not cumulatively
received. Every sequence lower than it is confirmed. Bit `i` of
`received_bitmap`, with bit zero least significant, confirms sequence
`next_sequence + i`; overflow in that addition is invalid.

Bit zero MUST be clear. If sequence `next_sequence` has been received as a
complete item, the cumulative prefix advances instead of representing that
sequence selectively.

The effective confirmation set is therefore:

```text
{ s | s < next_sequence } union
{ next_sequence + i | 0 <= i < 64 and bitmap bit i is 1 }
```

A valid checkpoint is cumulative, selective and idempotent. A later
checkpoint MUST NOT revoke a sequence already confirmed by an accepted
checkpoint for the same connection generation. A checkpoint that lowers
`next_sequence`, changes an already confirmed bit to unconfirmed without a
larger cumulative prefix, refers to another generation, or overflows its
selective range MUST NOT release retained items.

Feedback can be repeated on any authenticated reverse path authorized for the
same relationship and connection. Long delay does not require per-item
feedback or periodic traffic while no contact exists. Compact feedback state
MUST remain available for at least as long as it can prevent duplicate
delivery or release corresponding retained items.

A receiver MUST NOT advertise a checkpoint unless it can preserve the
checkpoint's confirmed set and corresponding replay obligation for their
useful lifetime. That lifetime is bounded by the reliable connection and its
retained or tombstoned state; wire-v4 `FEEDBACK` does not encode a wall-clock
validity field.

E2E feedback proves protocol receipt under the accepted reliable connection.
It is still not destination application acceptance.

## 8. Release conditions

The origin releases an admitted item only upon one of:

1. authenticated, nonregressing E2E feedback confirms its sequence;
2. its accepted expiry is reached;
3. an authorized administrative revocation explicitly covers it; or
4. a terminal local failure makes the retained representation unusable.

Every release other than confirmed E2E receipt has an observable reason.
Adjacent acknowledgment, path replacement, rekey initiation, clean drain,
local storage pressure and loss of one replica are not release conditions.

Confirmation is monotonic. Duplicate or replayed feedback is idempotent.
Feedback for an alias or generation that is no longer the item's exact scope
cannot confirm that item.

## 9. Cryptographic horizon

The effective useful horizon of an item cannot exceed the interval for which
the item remains cryptographically forwardable and acceptable. Rekey MUST
either preserve the required prior-generation material for admitted items or
produce an equivalent generation-safe retained representation without nonce
reuse.

Retention does not authorize key export, reconstruction of lost traffic
sequence state, restoration of an obsolete session or weakening to a `NONE`
suite. Loss of all cryptographic material required by one replica is a
terminal failure for that replica. The origin's independent application record
or another viable replica remains independently authoritative.

Version 1 makes no delivery claim after irreversible loss of both viable
endpoint cryptographic state and the origin record.

## 10. Path replacement, rekey and delayed contact

Handover changes a path, not the relationship or item identity. An admitted
item survives make-before-break replacement when its exact relationship,
service, direction and connection generation remain valid and it can be
reprotected safely. Its next transmission uses the selected path's current
adjacent protection and fragmentation.

If rekey changes only the replaceable session epoch, the reliable connection
and retained item identity remain stable. If continuity cannot be proven or
cryptographic viability cannot be preserved, the affected replica fails
explicitly; it MUST NOT reset the item sequence, silently create a new
relationship or deliver under a different authenticated peer.

Absence of a current path leaves admitted work pending until feedback, expiry
or a permitted terminal outcome. It is not by itself confirmation or failure.

## 11. Drain, restart and failures

Clean drain stops new admission before releasing volatile execution state. It
MUST preserve admitted items and feedback checkpoints that are required after
the drain, or explicitly complete, expire, revoke or fail them before the
drain completes. Restart restores only retained state that remains unexpired,
cryptographically viable and bound to the exact durable relationship state.

Abrupt loss may destroy a volatile replica and is not converted into an E2E
delivery guarantee. Corruption, administrative revocation, expiry,
cryptographic invalidation and storage failure are distinct observable
outcomes. Catastrophic loss of every replica and the origin record is outside
the version 1 guarantee.

Storage medium, persistence threshold, memory watermark, placement policy,
process boundary and recovery mechanism are local choices. A peer cannot
negotiate or infer them. A conformance claim concerns the externally visible
retention outcomes, not a particular storage architecture.

## 12. Error behavior

An incomplete policy, unauthenticated policy, wrong direction, stale policy
generation, zero-field policy treated as enabled, arithmetic overflow,
unbounded admission, incomplete retained item, feedback for the wrong alias or
generation, feedback regression and selective-range overflow fail closed.

Failure MUST NOT consume a refused sequence, release unrelated retained work,
fall back to adjacent-only acknowledgment or create application acceptance.

## 13. Security and privacy considerations

Long retention increases resource-exhaustion and traffic-analysis exposure.
Both item and byte caps are mandatory, metadata is bounded, and local policy
may impose stricter admission limits. Policy authentication and monotonic
generation prevent an attacker from extending an obligation or replaying a
larger allowance.

Retained representations SHOULD reveal no plaintext or private key to an
intermediary that does not terminate the E2E session. A retained ciphertext
does not grant nonce-generation authority. Fresh protection after handover is
required to prevent nonce reuse and cross-path replay.

Stable item identity and monotonic feedback prevent duplicate delivery and
premature release. Feedback reveals receipt gaps to the authenticated peer;
its exposure is limited to the relationship and reliable connection where it
is authorized.

## 14. Conformance

`vectors/relationship-retention-v1` fixes deterministic policy-intersection,
admission, feedback and lifecycle cases. The checker derives every result from
the rules in this document and verifies the vector manifest.

Passing these semantic vectors does not assign the policy carrier or permit
general activation while `RET-001` remains open. Conformance MUST NOT depend on
a kernel interface, operating system, daemon, database, storage tier or test
orchestration mechanism.

## 15. IANA considerations

This document currently requests no IANA action.

## 16. References

### 16.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Wire Protocol Version 4*.
- *PRP Relationship Lifecycle and Continuous Control*, version 1.
