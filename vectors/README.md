# PRP conformance vectors

Vectors contain deterministic examples and rejection cases for normative PRP
specifications. They do not define semantics independently of the specification
that references them.

Every vector set identifies its protocol version in its path or filename.
Changing a normative preimage, encoding, or expected result requires a new
reviewed vector revision. A conforming implementation MUST NOT alter input bytes
to make a vector pass.

The wire-v4 directory contains canonical structural, cryptographic,
key-derivation, ML-KEM, public-unit, and protected-fragment vectors. Its
`manifest.sha256` fixes the byte-exact revision. Run `make check` from the
repository root to validate integrity, TSV structure, result vocabulary, and
Strong-ID derivation. This validation command is repository tooling and does
not prescribe an implementation architecture.

The relationship-lifecycle-v1 directory contains semantic vectors for section
generation, key-policy intersection, rekey scheduling, generation overlap, and
drain transitions. Its checker derives the expected outcomes directly from the
normative rules and does not encode implementation-local state names.

The resource-report-v1 directory contains canonical policy and report bodies,
directional policy intersections, admission decisions, timing boundaries,
echo correlation, unknown-field behavior, and correlation-profile outputs.

The carrier-attachment-binding-v1 directory fixes published section `0x0005`,
its canonical declaration and receiver-local direct-evidence admission rules.

The discovery-policy-v1 directory fixes section `0x0006`, its exact
directional tuple and deterministic ordinary, crossed and restart transitions.
The protected-discovery-lane-v1 directory fixes transaction section `0x0101`
and exact session, policy, connection and receiver-alias admission boundaries.
The protected-discovery-payload-v1 directory composes all twelve assigned
outer kinds with canonical inner fixtures and fixes dispatch, generation,
correlation, replay and one-E2E-item rejection boundaries.

The sharded-bloom-v4 directory contains coordinates from P16 through P63,
sparse-tree commitments and proofs, canonical manifest and shard prefixes, a
signed relationship-bound view, generation transitions, and a coherent
MICROSHARD_V2 exchange. Its checker derives the microshard request, root and
returned membership bits from the same canonical discovery target key.

The content-identity-v1 directory fixes suite-bound content IDs, PRCD
descriptors, MERKLE_64K_BINARY trees, binary proofs, u64 lengths, metadata
separation and session-suite independence. The discovery-target-v1 directory
fixes the four target kinds, target keys, P24 coordinates and the common
weak-alias/content-ID multi-result policy.

The identity-publication-v1 directory contains canonical PRPU publication and
withdrawal records plus authorization, replay, generation, conflict and
tombstone state transitions.

The exact-discovery-v1 directory contains canonical-target lookup v2, RFC 9497
VOPRF, PRP VOPRF framing, exact-query, destination-challenge, proof and
lifecycle vectors. Its checker independently derives hashes and Strong IDs and
verifies the destination signature. Version-two target derivation is a
single-derivation cutover and has no version-one fallback.

The directional-carrier-v1 directory contains semantic admission and
composition vectors for local direction, capability, intent, readiness,
member evidence, exact member generations, HS2 promotion, asymmetric paths,
reverse control and handover.

The bootstrap-reachability-v1 directory contains semantic vectors for
physical and protected domains, opaque candidate sources, leases and
generations, cancellation, split-horizon projections and HS2 promotion. It
also proves that unbound referral and unregistered PIR sources remain
unavailable.

The member-route-intent-v1 directory contains canonical section `0x0100`
request and result records and conditional semantic vectors for reciprocal
intent, bilateral activation, replay, alias invalidation and recursive HS2.
Activation remains open independently of codec conformance.

The relationship-retention-v1 directory contains semantic vectors for
directional policy intersection, bounded item and byte admission, cumulative
selective E2E feedback, release conditions, rekey, handover, drain, restart and
replica failure. Storage placement is intentionally absent. Policy activation
remains open independently of semantic conformance.
