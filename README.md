# PRP Official Publications

This repository is the public publication channel for stable Participant
Relationship Protocol (PRP) documents referenced by the project website. It is
not the protocol's editorial working tree and does not accept independent
normative development.

The canonical editorial source is the internal `prp-spec` project. Drafts,
registries under development, conformance vectors, tests, source maps, and
cross-project review records remain there. A document appears here only after
an explicit publication decision.

## Published documents

| Document | Publication status | Website reference |
| --- | --- | --- |
| [PRP Architecture v1 RC1](prp-architecture-v1-rc1.md) | Release Candidate 1 | Architecture and PRP core roadmap |
| [PRP Wire v1](prp-wire-v1-draft.md) | Published working draft | PRP core roadmap |
| [PRP Encapsulation Profiles v1](prp-encapsulation-v1.md) | Published working draft | PRP core and carrier roadmap |
| [PRP v1 Wire Registry](wire-registry-v1.md) | Published working registry | PRP core roadmap |

The exact retained publication set and provenance are recorded in
[`PUBLICATION-MANIFEST.tsv`](PUBLICATION-MANIFEST.tsv).

## Authority and change control

Publication here freezes a reviewable public artifact; it does not create a
second editorial authority. Changes originate in `prp-spec`, pass its review
and promotion process, and are then copied here in a dedicated publication
commit. Git history preserves superseded publications.

Implementation behavior, repository examples, and unpublished working material
do not silently revise these documents. Status labels apply independently to
each document.

## Foundational statement

> Communication becomes possible because a relationship already exists.

PRP treats relationships as the primary communication object. Routes,
transports, and carriers are replaceable realization mechanisms rather than
participant identities.

## License

Unless a document states otherwise, publications are licensed under the
[Creative Commons Attribution 4.0 International License](LICENSE).
