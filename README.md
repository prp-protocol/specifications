# PRP Specifications

This repository contains the normative architecture, whitepaper, protocol specifications, service-layer models, and related technical documents for the **Participant Relationship Protocol (PRP)**.

PRP is a relationship-centric communication architecture in which communication emerges from pre-existing relationships rather than globally reachable locations.

## Status

The current foundational architecture document is:

```text
prp-architecture-v1-rc1.md
```

Status:

```text
Release Candidate 1
```

This document defines the architectural foundations of PRP, including its relationship-centric model, terminology, architectural principles, layers, threat assumptions, design goals, non-goals, and conformance rules.

## Foundational Documents

* [`prp-architecture-v1-rc1.md`](prp-architecture-v1-rc1.md)
  Normative foundational architecture specification for PRP.

Future foundational documents may include:

```text
whitepaper/
protocol/
services/
registries/
```

## Architectural Position

PRP is not an address-centric, identity-centric, routing-centric, or governance-centric communication architecture.

Instead, PRP treats relationships as the primary architectural object from which communication emerges.

The foundational architectural statement is:

> Communication becomes possible because a relationship already exists.

In this model:

* relationships are primary architectural objects;
* continuity is more fundamental than identity;
* routing, transport, and carrier selection are realization mechanisms;
* governance is optional rather than foundational;
* services extend the architecture but do not define it.

## Repository Purpose

This repository is the normative source for PRP architectural and protocol documentation.

It is intended to answer:

```text
What is PRP?
What belongs to the PRP core?
What belongs above PRP as a service?
What architectural principles should implementations preserve?
```

It is not intended to host implementation code.

Reference implementations should live in separate repositories, such as:

```text
prp-protocol/libprp
```

Academic papers and preprints should live in:

```text
prp-protocol/papers
```

## Conceptual Model

The current PRP architecture is organized around:

```text
Participant
    ↓
Relationship
    ↓
Continuity
```

Communication infrastructure exists to realize relationship continuity.

The core architectural layers are:

```text
Relationship Layer
Routing Layer
Transport Layer
Carrier Layer
```

The Service Layer exists above the core architecture and may include:

```text
Discovery
Governance
Evidence
Benchmarking
Economics
Custody
Advanced Routing
Applications
```

## Core Rule

A component belongs to the PRP core only if:

1. it is required by every valid deployment;
2. it cannot be implemented as a service;
3. its absence would invalidate the relationship-centric model itself.

Everything else belongs above the core as a service, extension, implementation detail, or deployment policy.

## Related Repositories

* Papers and preprints:
  `https://github.com/prp-protocol/papers`

* Reference implementation:
  `https://github.com/prp-protocol/libprp`

* Main project organization:
  `https://github.com/prp-protocol`

## Related Paper

The architecture defined here is the normative foundation for the paper:

**Relationship-Centric Communication: The Participant Relationship Protocol (PRP)**
Gustavo Junior Alves

Zenodo DOI:

```text
https://doi.org/10.5281/zenodo.20482932
```

## Suggested Citation

Until a formal specification DOI is available, cite the architecture document as:

```bibtex
@misc{alves2026prparchitecture,
  author = {Alves, Gustavo Junior},
  title = {PRP Architecture v1: Participant Relationship Protocol},
  year = {2026},
  version = {Release Candidate 1},
  url = {https://github.com/prp-protocol/specifications}
}
```

## License

Unless otherwise stated, documents in this repository are licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

You are free to share and adapt the material, including for commercial purposes, provided appropriate credit is given.

See [`LICENSE`](LICENSE) for details.

