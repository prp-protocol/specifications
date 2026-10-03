# Contributing to the PRP specifications

PRP specifications describe externally observable semantics and interoperable
behavior. They do not prescribe a programming language, operating system,
library, process architecture, storage engine, package format, deployment
method, or administrative tool.

## Normative language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** in PRP specifications are to be interpreted as described by BCP 14
when, and only when, they appear in all capitals.

Lowercase words such as "must" and "should" have their ordinary English
meaning and do not create BCP 14 requirements.

## Sources and authority

Sibling projects provide evidence and technical input. They may reveal:

- interoperable formats already in use;
- invariants enforced independently by multiple implementations;
- ambiguities, defects, security constraints, and operational limits; and
- candidate extensions supported by tests or experimental results.

No implementation becomes normative merely because it is the first, most
complete, or most widely deployed implementation. During consolidation, every
material rule must be classified as one of:

- an architectural invariant;
- an interoperable protocol requirement;
- a registered assignment;
- a profile-specific requirement;
- local policy; or
- an implementation choice.

Only the first four classes belong in normative protocol text. Local policy may
be described as a permitted decision boundary. Implementation choices belong
in non-normative notes only when they materially aid understanding.

## Document rights and external publication

The architecture and its Internet-Draft publication snapshots are licensed as
described in [`DOCUMENT-RIGHTS.md`](DOCUMENT-RIGHTS.md). A contribution intended
for an Internet-Draft also carries the incoming-rights, attribution and IPR
obligations applicable to the selected RFC stream. Contributors must disclose
third-party material and known implementation-relevant IPR; repository
acceptance does not imply that the contributor owned undisclosed material.

An RFC publication snapshot is immutable. Normative architectural changes
after publication require an explicitly versioned successor; the working
repository must not silently rewrite the meaning of the published version.

## Change requirements

A normative change should include:

1. the affected scope and document status;
2. the interoperability or architectural reason for the change;
3. security and privacy consequences;
4. compatibility and version-negotiation consequences;
5. updated registries or deterministic vectors when bytes change; and
6. provenance sufficient to audit the decision.

Wire meanings are immutable within an assigned protocol version. A change to a
field width, byte order, required field, cryptographic transcript, or existing
numeric meaning requires a new version or an explicitly specified compatible
extension point. Parsers MUST NOT guess a protocol version from malformed or
ambiguous input.

## Document maturity

Documents use the following maturity labels:

- **Working Draft**: incomplete and subject to incompatible change;
- **Draft Standard**: coherent candidate for implementation and review;
- **Standard**: approved normative authority for its declared scope;
- **Historic**: retained for reference and no longer recommended; and
- **Experimental**: specified for bounded experimentation without general
  interoperability status.

Maturity and protocol version are independent. A fourth version of a wire
format can still be a Working Draft; a version-one architecture can be a
Standard.

## Editorial rules

Readiness and handoff documents must identify their review date, exact source
revisions, evidence type (executed here, owner-reported, or inspection only),
remaining gates and successor when superseded. Correct obsolete blocker
explanations without turning a specification assignment or provider test into
consumer activation. Keep these implementation observations outside normative
specifications. Preserve published snapshots and historical test outcomes;
annotate them with a dated successor instead of rewriting their history.

- Define a term once and use it consistently.
- State requirements in observable terms.
- Separate architecture, base protocol, profiles, and bindings.
- Specify byte order, bounds, error handling, and unknown-value behavior for
  every wire structure.
- Distinguish identity, relationship, session, route, and carrier scopes.
- Identify whether extensibility is fail-closed, ignorable, or negotiable.
- Use diagrams and examples as non-normative aids unless stated otherwise.
- Never copy names of implementation handles into the protocol vocabulary
  solely because a current implementation exposes them.
