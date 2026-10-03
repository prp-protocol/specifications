# Public specifications mirror policy v1

The GitHub repository `prp-protocol/specifications` is a filtered public mirror
of `prp-spec`, not the source repository and not an independent specification
authority.

## Included

The deterministic exporter includes:

1. every tracked Markdown file directly under `specifications/`;
2. every tracked file directly under `registries/`, `vectors/` and `iana/`;
3. the current RFCXML, TXT and HTML forms of:
   * PRP Architecture `-00` and `-01`;
   * PRP SCIR `-00`;
   * PRP URI/QR `-00`; and
   * PRP DNS Reference RR `-00`;
4. `CONTRIBUTING.md` and `DOCUMENT-RIGHTS.md`; and
5. the mirror-specific README, license, policy and generated manifest.

Working Drafts are included when they are intended to become normative. Their
status remains visible and publication is not promotion.

## Excluded

Internal review reports, coordination documents, source maps, work queues,
implementation evidence, test programs, build outputs, private artifacts,
obsolete draft variants and every `working/` path are excluded. In particular,
the superseded identity-only DNS draft is not selected.

## Publication behavior

The exporter reads committed blobs from an exact Git revision and creates a
complete target tree plus `MIRROR-MANIFEST.json`. Publication replaces the
mirror's current tree with that generated tree in a normal non-forced commit;
Git history remains available. A publisher MUST verify the manifest before
and after publication and MUST NOT add unlisted files as current normative
material.

The mirror contains no automatic authority to submit a document to the IETF,
request an IANA assignment, create a release or claim implementation
conformance.
