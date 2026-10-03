# PRP registries

This directory contains the machine-readable assignments incorporated by PRP
specifications. A registry row is normative only under the specification and
document status that define it.

The requirements-language policy in [`CONTRIBUTING.md`](../CONTRIBUTING.md)
applies to this document.

An implementation, fixture, or experimental deployment cannot allocate a PRP
value by use. New assignments require a specification that defines:

- the requested value and registry;
- complete interoperable semantics;
- canonical encoding and bounds;
- unknown-value and negotiation behavior;
- security and privacy consequences; and
- deterministic positive and rejection vectors where bytes are involved.

Assignments MUST NOT change meaning after publication. A materially different
meaning requires a new value. Names are descriptive; numeric values and the
referenced specification determine wire semantics.

`manifest.sha256` fixes the exact bytes of the current registry revision. The
repository validation verifies that every registered identity suite has a
Strong-ID vector and every registered session suite has a traffic-key vector.

The `discovery-policy-v1` directory records the index, query, and composite
discovery profile assignments used by *PRP Directional Discovery Policy*,
version 1. Its own manifest fixes that registry revision independently of the wire-v4
suite registries. It also records the lookup-token profiles referenced by
discovery profiles and Bloom coordinates.

`relation-sections-v1.tsv` records assigned continuous relation-control
section types, versions, permitted request operations and state models.
Transaction-scoped sections do not inherit persistent-policy result semantics.

`protected-discovery-payload-kinds-v1.tsv` records the exhaustive outer kinds
admitted on a bound discovery lane. It incorporates existing canonical inner
formats without allocating identifiers to incomplete terminal, introduction,
referral or private-query messages.

`discovery-target-kinds-v1.tsv` records the four canonical target classes and
their result and proof semantics. `content-suites-v1.tsv` is the normative
content projection of the identity-suite registry: its numeric values are not
a second allocation space and reuse only each suite's hash and digest length.
