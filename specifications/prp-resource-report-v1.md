# PRP Protected Resource Reporting

- **Version:** 1
- **Status:** Working Draft
- **Category:** Protocol specification
- **Date:** 2026

## Abstract

This document specifies protected operational resource reporting within an
established Participant Relationship Protocol (PRP) relationship. It defines
the directional resource-report policy carried by continuous relation control,
the wire-v4 `RESOURCE_REPORT` control, report ordering, admission, expiry,
echo correlation, raw observation fields, and the optional correlation-score
profile.

Reports describe only the selected authenticated relationship and direction.
This document does not prescribe collection, persistence, scheduling, storage,
or administrative mechanisms.

## 1. Status and requirements language

This document is a Working Draft. Its wire layouts consolidate independently
consumed version-4 contracts. This revision additionally closes deterministic
policy intersection and originator-relative direction semantics required for
interoperable negotiation.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in this document are to be interpreted as described by BCP 14 when,
and only when, they appear in all capitals.

## 2. Scope

This document defines:

- resource-report policy section type `0x0003`, version `0x01`;
- independent TX and RX policy lineages;
- deterministic policy intersection;
- resource-report control kind `0x09`, version `0x01`;
- canonical report fields and TLVs;
- ordering, duplicate, validity, cadence, rate, and size rules;
- report echo semantics; and
- shaping-resistance and traffic-resistance profile-v1 values.

This document does not define:

- how observations are collected;
- an application, storage, or administrative interface;
- carrier-specific telemetry;
- topology or service reporting;
- a service-level agreement;
- an authorization policy language; or
- a solicitation control for solicited mode.

## 3. Relationship and protection boundary

A resource report is scoped by the authenticated relationship, relation
direction, session scope, and route alias that carry it. It MUST NOT contain or
be interpreted as a participant identity, relationship identity, carrier
locator, topology description, service identity, application credential, or
local implementation handle.

The policy is durable relationship state and is negotiated only through the
authenticated continuous relation control defined by the PRP Relationship
Lifecycle specification. It therefore cannot be negotiated through a `NONE`
session suite.

Wire v4 can structurally carry `RESOURCE_REPORT` through a `NONE` suite.
Because such a suite provides no traffic authentication, integrity,
confidentiality, or replay protection, a receiver MUST NOT admit such a report
as authenticated relationship evidence, update an accepted report, or derive
an authorization decision from it. Structural parsing does not imply semantic
acceptance.

A participant consumes a report locally. It MUST NOT recursively forward the
report as though the observations were its own. A separately authorized
custodian may publish an aggregate or retained observation under another
protocol, but that publication is not a PRP resource report.

## 4. Directional policy

### 4.1 Direction coordinates

Policy direction is expressed from the originator of the relation-control
request:

- `TX=0x01` constrains reports the originator proposes to transmit; and
- `RX=0x02` constrains reports the originator proposes to receive.

The receiver aligns an originator's TX policy with its local RX constraint, and
an originator's RX policy with its local TX constraint. A RESULT retains the
originator's direction byte; it does not rewrite the request into the
receiver's local coordinate system.

TX and RX are independent policy instances. Their section-generation lineages
are keyed by section type, version, request originator, and direction. Updating
one direction does not advance, disable, or replace the other. Reports do not
carry a direction byte; direction is determined by the authenticated
relationship context in which they are received.

### 4.2 Policy body

Relation section type `0x0003`, version `0x01`, is used with PROPOSE or
CONDITION. Its body is exactly 28 bytes:

| Offset | Bytes | Field |
| ---: | ---: | --- |
| 0 | 1 | body version, `0x01` |
| 1 | 1 | direction, TX or RX |
| 2 | 1 | mode |
| 3 | 1 | flags, zero |
| 4 | 8 | permitted field mask |
| 12 | 4 | preferred interval, milliseconds |
| 16 | 4 | minimum interval, milliseconds |
| 20 | 4 | maximum report validity, milliseconds |
| 24 | 2 | maximum reports per rolling minute |
| 26 | 2 | maximum encoded report bytes |

All integers are unsigned and big-endian.

### 4.3 Modes

| Value | Mode | Meaning |
| ---: | --- | --- |
| `0x00` | DISABLED | No report is admitted in this direction |
| `0x01` | PERIODIC | Reports may be originated on a recurring schedule |
| `0x02` | EVENT_ONLY | Reports are originated only in response to a local observation event |
| `0x03` | SOLICITED | Reports are originated only after an authenticated solicitation defined by a governing profile |
| `0x04` | PIGGYBACK | Reports are originated only when they can accompany otherwise permitted protected traffic |

This document defines no solicitation record. Consequently, SOLICITED mode
does not authorize transmission unless another normative profile defines and
authenticates its trigger.

Mode constrains report origination. A receiver can enforce canonical policy,
cadence, rate, and size but is not required to prove that a remote local event
occurred or that a transmission required no additional carrier action.

### 4.4 Canonical policy

A disabled policy has every field after direction equal to zero.

An enabled policy has:

- a nonzero mask containing only fields from Section 6;
- a nonzero minimum interval;
- a nonzero validity not shorter than the minimum interval;
- a nonzero maximum reports per minute; and
- a maximum byte count within the bounds in Section 4.5.

PERIODIC additionally has a nonzero preferred interval no shorter than the
minimum interval and no longer than validity. Every other enabled mode encodes
preferred interval as zero.

### 4.5 Minimum and maximum size

The maximum encoded report bytes is no greater than 65,527. It is no smaller
than the shortest report allowed by the policy mask:

- 73 bytes when either one-byte resistance score is permitted;
- otherwise 80 bytes when any 64-bit counter is permitted;
- otherwise 120 bytes when ECHO is permitted; or
- 68 bytes only for the purpose of evaluating a disabled mask, which cannot be
  used by an enabled policy.

The minimum is based on the shortest permitted subset, because a report need
not contain every field allowed by policy.

### 4.6 Policy intersection

Policy intersection is computed only after aligning complementary local
directions as described in Section 4.1. It is commutative.

If either operand is DISABLED, the effective policy is the canonical disabled
policy in the request originator's direction. Two different enabled modes are
incompatible and produce REJECTED rather than an inferred mode conversion.

For matching enabled modes, the effective policy uses:

- bitwise intersection of the permitted field masks;
- the greater minimum interval;
- the smaller validity;
- the smaller maximum reports per minute;
- the smaller maximum encoded byte count; and
- for PERIODIC, the greater preferred interval.

If the resulting field mask is zero, the effective policy is canonical
DISABLED. If the resulting enabled policy violates Section 4.4 or Section 4.5,
the proposals are incompatible and produce REJECTED. Neither participant may
widen an effective policy after acceptance. Local policy MAY report less
often, include fewer fields, use shorter validity, or use fewer bytes.

## 5. Report header

The report begins with this 68-byte header:

| Offset | Bytes | Field | Rule |
| ---: | ---: | --- | --- |
| 0 | 1 | control kind | `0x09` |
| 1 | 1 | version | `0x01` |
| 2 | 1 | flags | zero |
| 3 | 1 | reserved | zero |
| 4 | 4 | report generation | nonzero |
| 8 | 8 | sequence | unsigned ordering value |
| 16 | 4 | validity | nonzero milliseconds |
| 20 | 4 | sample interval | milliseconds; zero is permitted |
| 24 | 8 | sender monotonic time | nanoseconds |
| 32 | 8 | clock epoch | opaque sender epoch |
| 40 | 8 | field mask | nonzero known fields |
| 48 | 16 | report nonce | nonzero |
| 64 | 2 | TLV count | exact following TLVs |
| 66 | 2 | reserved | zero |

Sender monotonic time, sample interval, and clock epoch are observations. They
do not establish synchronized time and MUST NOT be interpreted as civil time.
A clock epoch change prevents timing samples from different epochs from being
silently combined.

## 6. Field registry

| Mask bit | Name | TLV | Value bytes | Semantics |
| ---: | --- | ---: | ---: | --- |
| 0 | ECHO | `0x0001..0x0004` | 4, 8, 16, 8 | Section 9 tuple |
| 1 | TX_BYTES | `0x0005` | 8 | authenticated transmitted bytes |
| 2 | RX_BYTES | `0x0006` | 8 | authenticated received bytes |
| 3 | TX_ITEMS | `0x0007` | 8 | authenticated transmitted items |
| 4 | RX_ITEMS | `0x0008` | 8 | authenticated received items |
| 5 | TX_DROPS | `0x0009` | 8 | dropped TX items or units under the selected profile |
| 6 | RX_DROPS | `0x000a` | 8 | dropped RX items or units under the selected profile |
| 7 | QUEUE_OCCUPANCY_BYTES | `0x000b` | 8 | occupied bytes in the reported bounded queue |
| 8 | QUEUE_CAPACITY_BYTES | `0x000c` | 8 | capacity of that queue in bytes |
| 9 | RESIDUAL_CAPACITY_BPS | `0x000d` | 8 | estimated residual bits per second |
| 10 | SHAPING_RESISTANCE | `0x000e` | 1 | Section 13 profile-v1 score |
| 11 | TRAFFIC_RESISTANCE | `0x000f` | 1 | Section 13 profile-v1 score |

Counter fields are raw unsigned observations. Except for the two profile-v1
scores, a report MUST NOT encode computed jitter, skew, loss rate, degradation
state, or an SLA verdict in a registered field. Consumers derive such values
from admitted reports and timing evidence.

When both queue fields occur, occupancy MUST NOT exceed capacity. A zero raw
counter is valid and is not the same as an absent field.

## 7. TLV encoding

Each TLV is:

```text
raw_type:u16 || length:u16 || value[length]
```

Bit 15 of `raw_type` is CRITICAL; bits 0 through 14 are the base type. TLVs are
strictly ordered by ascending nonzero base type and a base type occurs at most
once. The TLV count and body length must be exact.

Every set field-mask bit has exactly its registered TLV or, for ECHO, all four
registered echo TLVs. Every known TLV has its mask bit set. Thus the known TLVs
and field mask agree exactly.

An unknown optional TLV is ignored semantically but counts toward TLV count,
ordering, report bytes, and policy size. An unknown critical TLV rejects the
entire report. A receiver MUST NOT assign a meaning to an unknown field from
its length or position.

## 8. Ordering and duplicates

Ordering is lexicographic by report generation and then sequence. A candidate
with a greater generation is NEW even if its sequence is lower. With equal
generation, a greater sequence is NEW and a lower sequence is STALE.

Equal generation and sequence with the same nonce is DUPLICATE. It does not
replace the first admitted canonical report, renew its receipt time, extend its
validity, or consume rate allowance. Equal generation and sequence with a
different nonce is noncanonical and rejected.

A participant that loses the state required to continue a report generation
MUST select a strictly greater nonzero report generation. Report generation is
independent of session generation, relation-section generation, and clock
epoch.

## 9. Echo

ECHO is an indivisible four-TLV tuple:

| TLV | Field |
| ---: | --- |
| `0x0001` | echoed report generation, nonzero u32 |
| `0x0002` | echoed report sequence, u64 |
| `0x0003` | echoed 16-byte nonce, nonzero |
| `0x0004` | receiver-local monotonic receipt time for the echoed report, nonzero u64 nanoseconds |

Generation, sequence, and nonce MUST exactly identify an admitted report in
the opposite relationship direction. The receipt time is the echo sender's
local observation; the peer does not compare it numerically with its own
clock. A partial tuple, mismatched coordinate, or mismatched nonce rejects the
report.

Echo can support round-trip, offset, and skew estimation when combined with
local receipt times and stable clock epochs. This document does not prescribe
an estimator and does not assert synchronized clocks.

## 10. Admission against policy

A structurally canonical report is admitted only when:

- it arrives in the authenticated relationship and direction selected by the
  effective policy;
- the policy is enabled;
- its known field mask is a nonempty subset of the permitted mask;
- its encoded length does not exceed the policy byte limit;
- its validity does not exceed the policy validity;
- its ordering result is NEW, or it is an exact DUPLICATE handled as described
  in Section 8;
- NEW reports are separated from the previous admitted NEW report by at least
  the minimum interval; and
- admitting it would not exceed the rolling-minute rate.

For a candidate received at local monotonic time `t`, the rolling rate counts
previously admitted NEW reports in the same relationship direction whose
receipt time is greater than or equal to `t - 60,000 ms`, with arithmetic
saturated at zero. If that count is already the policy maximum, the candidate
is rejected. Duplicate and rejected reports do not enter the count.

The preferred interval guides PERIODIC origination; it does not authorize a
receiver to relax minimum interval or rate enforcement. Sample interval is an
observation field and is not the report cadence.

## 11. Validity and current observations

Validity begins at the receiver's local authenticated receipt time. A report
is current while:

```text
now - authenticated_receipt <= validity_ms * 1,000,000
```

It is expired when the difference is greater. A local monotonic regression
invalidates the comparison and MUST NOT make the report current. Sender time,
duplicates, echoes, administrative reads, and session rekey do not renew
validity.

Expiry removes the report from current operational evidence. It need not erase
an independently authorized audit record, but an expired record MUST NOT be
presented as current.

## 12. Counter interpretation

Counters are monotonic only within the scope declared by their observation
profile and clock epoch. A consumer MUST NOT calculate a delta across report
generation, clock epoch, relationship epoch, field absence, or a counter
decrease without an explicit reset rule.

TX and RX are always interpreted from the report sender's viewpoint. Queue
occupancy and capacity refer to the same queue and sampling instant. Residual
capacity is an estimate, not a reservation or transmission promise.

Drop counters require a profile that identifies the counted unit. In the
absence of such a profile, they are useful only for changes within one stable
reporting context and MUST NOT be compared as universal item-loss counts.

## 13. Correlation metric profile v1

### 13.1 Meaning and admission

SHAPING_RESISTANCE and TRAFFIC_RESISTANCE describe bounded evidence for one
authorized relationship direction. They do not describe an entire carrier,
aggregator, participant, service, or trajectory and do not directly assign a
security class.

Only a logical PRP unit admitted after reconstruction, authentication, replay
validation, and relationship binding is an observation. Carrier fragments,
retransmissions, coding symbols, polling records, and local queue entries are
not observations. Invalid input changes no accumulator.

### 13.2 Fixed time and bucket profile

Profile v1 uses 10 ms bins, 4,096 bins per nonoverlapping window, and therefore
a 40.96 second window. Fixed directional lags are one and two bins. At least
eight observations are required in each compared class; full confidence is
reached at 32.

Size buckets have inclusive upper bounds 256, 512, 1,024, 1,500, 4,096,
16,384, and 60,000 bytes, followed by a greater-than-60,000 bucket. Gap buckets
have inclusive upper bounds 1, 2, 4, 8, 16, 32, and 64 bins, followed by a
greater-than-64 bucket.

Bins align to a locally selected monotonic epoch. A monotonic discontinuity or
epoch change discards the active window. A completed window atomically replaces
the previous completed window. Before the first completion the score is absent;
after two complete window durations without replacement it becomes absent.

### 13.3 Shaping resistance

The profile compares size histograms for inactive versus active protected
demand and, when independently sufficiently populated, real versus locally
generated cover. It also compares the joint TX/RX size distribution in bins
containing both directions with the product of its marginals. Each comparison
uses total variation distance. Shaping distinguishability is the greatest
applicable distance.

### 13.4 Traffic resistance

Demand distinguishability is the greater of the inactive/active emission-rate
difference and the applicable gap-histogram total variation distance.
Directional dependence is the greatest capped L1 distance between an observed
binary joint distribution and the product of its marginals, evaluated for the
same bin and separately for TX-to-RX and RX-to-TX at lags one and two. Traffic
distinguishability is the greater of demand distinguishability and directional
dependence.

### 13.5 Confidence and quantization

For class counts `a` and `b`, let `balanced = min(a,b)`. Evidence is absent
when balanced is below 8. Confidence is `balanced/32` for values 8 through 31
and one at 32 or greater. The main inactive/active comparison determines score
presence; optional comparisons participate only when independently sufficient.

The raw resistance is one minus the greatest applicable distinguishability.
The transmitted score is the nearest integer to
`clamp(raw_resistance * confidence, 0, 1) * 254`, with exact half values rounded
up. Values 0 through 254 are measured evidence. Absence means insufficient or
expired evidence and is represented by omitting the field.

Value 255 is reserved for a physical override explicitly authorized by local
policy. Receiving 255 does not grant or prove that authorization and does not
change the measured profile-v1 result.

Aggregation receives no direct score bonus. Bilateral or multi-path consumers
MAY conservatively select the minimum available score, but missing evidence
remains missing and the operation does not create an S1-through-S5 class.

## 14. Failure and restart

Malformed, unauthenticated, stale, conflicting, over-policy, over-rate, and
expired reports fail independently of relationship continuity. They MUST NOT
silently widen policy or reset ordering state.

Loss of volatile receipt time makes a retained report unusable as current
evidence unless an authenticated retained checkpoint preserves the required
monotonic relationship. Loss of report ordering state requires a newer report
generation; it does not require a new relationship epoch or session generation.

Rekey and path replacement do not reset accepted policy, report generation,
ordering, or current validity. A report remains scoped to the authenticated
relationship direction, not the path that carried it.

## 15. Security and privacy considerations

Resource reports expose activity, capacity, queue pressure, loss, timing, and
correlation evidence. Relationships SHOULD negotiate only fields required for
an authorized purpose, use bounded validity, and avoid reporting more often or
with more precision than necessary.

An attacker can falsify operational decisions if reports are admitted without
traffic authentication. An attacker can also exhaust parsing or retained state
with large reports, optional TLVs, high rates, or generation churn. Receivers
MUST enforce negotiated size, cadence, rate, ordering, and bounded state before
performing expensive analysis.

Raw counters and resistance scores are observations, not authorization. They
MUST NOT independently grant forwarding, identity, service access, or physical
security status. Score 255 in particular is a local-policy assertion rather
than measured cryptographic evidence.

Unknown optional fields enable extension but can create inconsistent views.
Security-critical semantics therefore require a critical registered field or
a new version, not an optional field whose absence changes authorization.

## 16. Conformance

Enclosing relation-control and RRL framing uses the applicable wire-v4 codec
vectors. Policy and report encoding and semantic conformance use every
applicable case in `vectors/resource-report-v1`, including policy intersection,
ordering, expiry, echo, size, unknown fields, policy admission, and correlation
profile outputs.

Vector outcome names are conformance vocabulary and do not add wire status
values. A claim that excludes correlation profile v1 MUST state that exclusion;
such an implementation cannot emit measured values 0 through 254 for the two
profile-v1 score fields.

## 17. IANA considerations

This document currently requests no IANA action.

## 18. References

### 18.1 Normative references

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- *PRP Architecture*, version 1.
- *PRP Wire Protocol Version 4*.
- *PRP Relationship Lifecycle and Continuous Control*, version 1.
