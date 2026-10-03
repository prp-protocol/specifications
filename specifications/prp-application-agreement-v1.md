# PRP Application Agreement v1

- **Status:** Experimental; normative admission for implementation and testing
- **Version:** 1
- **Date:** 2026-09-05
- **Decision:** Admit a distinct versioned connection-opening variant.

## 1. Scope and authority

This specification extends wire-v4 Section 15 with application-bound connection
opening. BCP 14 requirement words have the meaning used by that specification.
Legacy OPEN, ACCEPT, REFUSE, CLOSE, CLOSE_ACK and FEEDBACK retain their bytes
and meanings. Neither legacy acceptance nor HS2 alone constitutes application
agreement. No item flag, payload prefix or untransmitted parameter provides it.
The architecture Internet-Draft revision 00 is not changed by this increment.

The new kinds below are assigned by this specification in the **connection
control** namespace only. They are not RRL kinds or discovery payload kinds.
Acceptance is experimental protocol admission, not evidence of implementation,
application readiness, persistence, or interoperable deployment.

## 2. Authenticated context

All agreement messages MUST be carried as connection-control bodies inside
authenticated and integrity-protected E2E traffic (scope 0x03), after both HS2
identity proofs and the final transcript have been verified. NONE suites are
forbidden. Adjacent authentication alone is insufficient. Retransmission uses
fresh protected-unit sequence/nonce values; cached control bodies may repeat.

All integers below are unsigned big-endian. LP16(x) is u16 byte length followed
by exactly x. ASCII domain strings exclude NUL. H is SHA-256 with 32-byte
output, fixed independently of the session suite. SHA-256 is a commitment,
not a MAC or substitute for the authenticated E2E carriage.

Derive context C locally from the completed HS2, never from caller assertions:

```
C = H("PRP-APP-CONTEXT-v1" || 0x04 || 0x03 || session_suite:u16 ||
      e2e_generation:u32 || LP16(final_transcript_hash) ||
      initiator_identity_suite:u16 || LP16(initiator_identity_public_key) ||
      responder_identity_suite:u16 || LP16(responder_identity_public_key))
```

Principal order is HS2 M1 sender followed by M2 sender, regardless of who opens
the connection. Public keys and transcript length MUST match their registered
suites. Generation is nonzero. This binds exact verified principals, selected
suite, transcript, direction scope and E2E incarnation. A local attachment and
revocation epoch additionally scope authority; they are not portable wire IDs.
Changing attachment, endpoint context, session generation or revocation epoch
invalidates agreement even if an application can reproduce C. Different social
principals require separately authenticated delegation; this agreement does
not define or imply such delegation.

## 3. Canonical messages and assignments

Every control uses the existing alias-zero E2E item, zero item flags and zero
item sequence. Framing is exact; unknown kinds, versions, fields, trailing
octets, truncation and invalid lengths fail closed. No extension TLVs exist.
The common prefix P is:

```
kind:u8 || agreement_version:u8=1 || opener:u8 || attempt:u64 || C[32]
```

Opener is 1 for the HS2 initiator, 2 for the HS2 responder. Attempt is a
strictly increasing nonzero counter per (C, opener); it MUST never wrap or be
reused. Parallel offers may arrive out of order: a previously unseen attempt
below the receiver's high-water mark is rejected. Retry then uses a fresh ID.
Exact cached duplicates are the sole exception. This trades retry availability
for bounded replay state. Neither side restores counters or agreements after
loss of volatile session state; full fresh HS2 is required.

| Kind | Name | Body after P | Total bytes |
| --- | --- | --- | --- |
| 0x07 | OPEN_APP | LP16(application) \|\| profile:u16 \|\| application_version:u16 \|\| flags:u16 \|\| window:u16 \|\| max_queued_bytes:u32 \|\| tombstone_ms:u32 | 62..125 |
| 0x08 | ACCEPT_APP | offer_hash[32] \|\| alias:u16 \|\| generation:u32 \|\| acceptance_hash[32] | 113 |
| 0x09 | CONFIRM_APP | acceptance_hash[32] | 75 |
| 0x0a | REFUSE_APP | offer_hash[32] \|\| reason:u16 | 77 |

Application is 1..64 ASCII bytes, each in [A-Z0-9-], with no case folding,
normalization or NUL. Profile and application_version are nonzero. Exactly
one triple is offered; no lists, alternatives or counteroffers are permitted.
An endpoint MUST recognize and explicitly authorize the entire triple.

Connection parameters retain wire-v4 semantics: flags use bits 0..2 only;
RELIABLE requires ORDERED; ordered window is 1..64, otherwise window and queue
are zero. This variant additionally limits ordered max_queued_bytes to
1..1048576 and tombstone_ms to 1000..600000. ACCEPT_APP accepts all offered
parameters exactly. Any different parameters require refusal and a fresh offer.

```
O = complete OPEN_APP body including P
offer_hash = H("PRP-APP-OFFER-v1" || length(O):u32 || O)
A = ACCEPT_APP prefix P || offer_hash || alias:u16 || generation:u32
acceptance_hash = H("PRP-APP-ACCEPT-v1" || length(A):u32 || A)
```

ACCEPT_APP includes A followed by acceptance_hash. All responses repeat the
exact opener, attempt and C of the pending offer. The offer is sent by opener;
acceptance/refusal by the other principal; confirmation by opener. Receivers
MUST check actual authenticated sender roles, commitments and local pending
state. A syntactically valid self-consistent commitment is insufficient.

Alias is 1..65535, allocated by the accepting endpoint. Generation is nonzero,
strictly increasing for new allocations by that endpoint within C. Before
accepting, both endpoints MUST ensure the alias is not active, reserved or
previously used by *any* connection (legacy or bound) in the E2E session.
Once reserved for this variant, an alias MUST NOT be reused until fresh HS2,
even after timeout, refusal or tombstone expiry. The used-alias set is bounded
at 65535 bits. On a collision the opener invalidates this attempt and tears down
the E2E session; it does not confirm or close a possibly unrelated connection
using the conflicting tuple. Concurrent conflicting reservations fail closed.
No reuse is guessed from
generation alone: base data items contain an alias but no connection generation.

REFUSE_APP reasons are 1 POLICY, 2 RESOURCE, 3 UNSUPPORTED, 4 STALE, 5 SHUTDOWN;
no other values, retry hint, or empty success are defined. It refers to a
well-formed authenticated offer whose hash the sender computed. Malformed or
unauthenticated input receives no response. Unknown application triples can
receive UNSUPPORTED; unknown agreement versions/kinds are discarded. No response
allocates application authority. Legacy REFUSE cannot complete this negotiation.

## 4. State, bounds and lifecycle

The opener retains immutable O in OFFER_SENT. The receiver validates O,
authorizes its exact triple, reserves the alias/generation and caches ACCEPT_APP
in ACCEPT_SENT atomically before emission. It has accepted the offer but MUST
NOT admit application records yet. The opener validates acceptance against O
and current context, reserves the returned tuple, then emits CONFIRM_APP. It
may admit records only after confirmation has been handed to authenticated
transport. The receiver becomes ACTIVE only after exact CONFIRM_APP validation.
Confirmation does not acknowledge delivery or durable application persistence.

Until these barriers, inbound application data MUST be discarded without
application callbacks, storage or speculative parsing; outbound data is refused.
Transport reordering may discard early data. Reliable applications retry under
their existing protocol; no data is used as implicit confirmation. Repeating
ACCEPT_APP causes an active opener to resend exact CONFIRM_APP, not reallocate.
Repeating CONFIRM_APP is idempotent while the same incarnation remains active.

At most 64 pending agreements and 4096 distinct attempts per opener per C are
permitted. The 4096 bound includes rejected/cancelled attempts. Each attempt
has a local monotonic deadline of at most 30000 ms from first admission/send,
never extended by duplicates. Implementations may impose lower limits. Retain
exact duplicate replies for at most the offered tombstone_ms after completion,
with no more than 4096 cache entries per opener. Cache eviction is permitted:
high-water and used-alias state remain until E2E teardown, so evicted duplicates
fail stale rather than become new attempts. Rate limiting MUST bound processing
and retransmissions; a duplicate is never an unbounded reply obligation.

Identical duplicates in their valid state reuse cached bytes without repeated
allocation, delivery or policy effects. Divergent bytes for a cached
(C, opener, attempt), including a changed ACCEPT_APP, invalidate that attempt
and any resulting binding. Replay in another C, connection tuple or role fails.
Unknown application/profile/version, refusal, timeout, exhausted limits and
failed authentication never downgrade to legacy OPEN or another application.

Close, replacement, local cancellation or revocation synchronously stops
admission, clears reassembly and invalidates pending/active authority. Existing
CLOSE/CLOSE_ACK applies to the exact allocated alias/generation; a close before
CONFIRM_APP wins and delayed confirmation cannot revive it. Cancellation before
alias allocation is local; a delayed acceptance is rejected and its tuple may
be HARD-closed. Rebinding always uses a new attempt and unused alias; session
replacement requires a new C. Tombstones never authorize restoration.

Any asynchronous delivery/commit MUST revalidate the same private incarnation
and serialize commit versus invalidation, including across independent local
contexts. Persistent payloads or reconstructed public objects cannot recreate
authority. The protocol does not prescribe storage or locking implementations.

## 5. Application separation and compatibility

This registry assigns admission names, not payload prefixes or new POST bytes:

| Application bytes | Profile | Version | Binding meaning |
| --- | --- | --- | --- |
| POST (504f5354) | 1 | 1 | Canonical POST v1 records defined by the social POST contract |
| PRPD (50525044) | 1 | 1 | Existing private demo PRPD v1 record format |

These profile numbers belong exclusively to this application-agreement registry;
they are not social field/profile IDs. Payload semantics remain with each
application's specification. PRPD registration does not promote its private
format into the PRP core. Binding to POST never admits PRPD, or vice versa.
Separate connection incarnations and agreements are required; payload sniffing,
magic-prefix switching and mixed bindings are forbidden. POST records travel
byte-for-byte unchanged; this control adds no per-record envelope or text tunnel.
Application decoders still enforce their own authentication and size limits.

Legacy peers reject the new control kind under wire-v4 Section 16. A caller
requiring agreement MUST report binding unavailable/unsupported on refusal or
deadline, without legacy retries. Legacy APIs remain usable for their original
transport-only purpose, but cannot issue a bound application capability.
There is no capability inference from version strings or ignored parameters.

## 6. Security, conformance and provenance

Domain separation, exact length framing, ordered principal binding and fresh
incarnations prevent cross-application, cross-session and reflection confusion.
Alias nonreuse is necessary because legacy data framing omits its generation.
E2E authentication does not establish social authorship or civil identity.
Application identifiers are visible to endpoints and protected from transit by
the selected E2E suite. Resource limits do not guarantee availability.

Acceptance vectors in vectors/application-agreement-v1 exercise commitments,
exact parsing, lifecycle gates and negative bindings. Synthetic keys/transcript
in these vectors stand for already verified HS2 context; they are not HS2 proof
vectors. Implementers MUST additionally test real authenticated E2E carriage,
nonce/replay behavior and asynchronous invalidation before application enablement.

Decision provenance: request ctl-1788635642-651, report 2e144da3658921f2,
prp/prp-protocol-social#2; base spec 77f84328e10aa4cbc34c1fb3f610cb97f547d1ec;
web-runtime 33e6ea53a993382c147a87cebfb18f624fa3d2da application-record-boundary.md;
orchestrator contracts-post-admission-and-d83-return.md (POST portion only).
No reusable application agreement exists in legacy Section 15.
