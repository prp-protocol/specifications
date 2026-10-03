# Participant Relationship Protocol specifications

This repository is the public, filtered mirror of the normative and
proposed-normative documents maintained by the Participant Relationship
Protocol (PRP) specification project.

PRP is a relationship-centric, carrier-independent communication architecture.
Its foundational statement is:

> Communication becomes possible because a relationship already exists.

## Status

Documents in this mirror have different maturity levels. Unless a document
states otherwise, it is a **Working Draft**: publication here invites review
and implementation feedback, but does not mean IETF publication, IANA
registration, protocol assignment, deployment approval or implementation
conformance.

The status declared inside each document is authoritative for that document.
The generated `MIRROR-MANIFEST.json` binds every mirrored path to an exact
source revision, size and SHA-256 digest.

## Current document families

* `specifications/` — architecture, wire protocol and focused normative
  specifications;
* `internet-drafts/` — current RFCXML sources and rendered Internet-Draft
  candidates, including Architecture, SCIR, URI/QR and DNS;
* `registries/` — closed or versioned protocol registries;
* `vectors/` — public conformance vectors;
* `iana/` — draft registration and review material; and
* `DOCUMENT-RIGHTS.md` — contribution, reuse and IPR boundaries.

The PRP Self-Certifying Identifier Reference (SCIR) envelope has four classes:
`IDENTITY`, `ALIAS`, `MULTICAST` and `GROUPCAST`. The generic textual carrier
uses lowercase `prp:` plus canonical unpadded base64url; contact admission is a
separate, narrower operation that requires an `IDENTITY` SCIR and cryptographic
proof. Older `prp:01:<hex>` examples are not the current generic carrier.

## Mirror boundary

This mirror intentionally excludes internal coordination records, working
directories, private evidence, implementation artifacts and repository-local
review reports. Its history is preserved as normal Git history, but the
current tree is generated from the exact source revision named in the manifest.

Project overview: <https://prp-protocol.org/>

## License

Author-owned material selected by the mirror policy is available under the
Creative Commons Attribution 4.0 International license, subject to the precise
scope and third-party boundaries in `DOCUMENT-RIGHTS.md`.
