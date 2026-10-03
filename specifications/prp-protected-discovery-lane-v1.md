# PRP Protected Discovery Lane Version 1

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protected discovery transport binding
- **Date:** 2026

## 1. Scope

This document binds one runtime-internal discovery lane to an authenticated
E2E session. It assigns no service identifier. Payload is admitted only under
the version, kind, context and bounds assigned by *PRP Protected Discovery
Payload Envelope*, version 1.

## 2. Prerequisite connection

The direction sender first opens an ordinary E2E connection under wire-v4.
The connection MUST be accepted with a nonzero receiver-allocated alias and
the exact flags `ORDERED | RELIABLE | MESSAGE_BOUNDARIES` (`0x0007`). The
connection lifecycle, flow control, queue bounds and tombstones remain those
of wire-v4. A service alias, application endpoint or implementation-local
handle MUST NOT identify the lane.

## 3. Binding section

`DISCOVERY_LANE_BIND` has relation-section type `0x0101`, version `0x01`, no
section flags, operation `PROPOSE`, and transaction-scoped state. Its body is
exactly twenty octets:

| Offset | Width | Field |
| ---: | ---: | --- |
| 0 | 1 | canonical discovery direction (`0x01` or `0x02`) |
| 1 | 1 | reserved, zero |
| 2 | 2 | connection flags, exactly `0x0007` |
| 4 | 4 | accepted discovery-policy generation |
| 8 | 4 | authenticated E2E session generation |
| 12 | 4 | connection generation |
| 16 | 2 | receiver-allocated connection alias, nonzero |
| 18 | 2 | reserved, zero |

The section `generation` equals the body policy generation and is nonzero;
`base_generation` is zero. Request and result contain the same complete body.

## 4. Admission and activation

The authenticated Relation Control context binds the exact self-certifying E2E
endpoint identity already selected by the host session, relationship identity
and epoch plus route; the binding creates no additional identity. The
body additionally binds direction, accepted policy generation, session
generation, connection generation, flags and receiver alias. The receiver
MUST verify all fields against the connection it allocated and accepted.

Binding is rejected when identity, relationship, direction, session, policy
generation, connection or alias differs; while policy is absent, pending or
disabled; or under either registered `NONE` suite. The receiver durably binds
the lane before emitting `ACCEPTED`. The sender activates only after
authenticating that result. Payload before bilateral activation is rejected.
Replay in another relationship, session, direction or policy generation is
therefore invalid. An alias collision cannot replace an existing binding.

## 5. Lifecycle

At most one lane is active for a direction and policy generation. Rekey uses
make-before-break: open a new-session connection, bind it while the old lane
remains active, switch only after acceptance, then close the old connection.
Aliases and sequence numbers are never carried between sessions. Restart
invalidates session and connection state and requires OPEN, ACCEPT and binding
again; durable discovery policy alone does not restore a lane.

Policy disablement stops new discovery admission and closes the lane. Session
or connection terminal loss also closes it. Already authenticated durable
objects follow their own retention rules; no queued ephemeral request obtains
authority after closure.

## 6. Bounded records

Every assigned discovery envelope MUST fit one E2E item. No generic
continuation or fragmentation protocol exists. Bloom transfer is shard-native
and each shard is independently authenticated and bounded by its specification.
An unassigned envelope version or payload kind is rejected without inner
dispatch.

## 7. Conformance

Conformance MUST cover exact activation, payload-before-bind, both `NONE`
suites, identity and relationship mismatch, wrong direction, stale policy,
wrong session or connection generation, non-receiver alias, flag mismatch,
replay, alias collision, rekey overlap, restart, disablement and terminal
loss. Canonical evidence is fixed by
`vectors/protected-discovery-lane-v1/manifest.sha256`.
