# PRP Self-Certifying Identifier Reference (SCIR)

- Version: 1
- Status: Working Draft; author-approved name and removal of the obsolete
  `prp:HH:<hex>` syntax; final standards submission pending
- Author: Gustavo Junior Alves, GJ LABS, <gjalves@gjalves.com.br>

## 1. Scope

This document defines the canonical PRP Self-Certifying Identifier Reference
(SCIR). A SCIR names an IDENTITY, ALIAS, MULTICAST or GROUPCAST through a
suite-selected cryptographic commitment to class-specific immutable certifying
material.
After this first expansion, this document uses **SCIR** for both the abstract
reference and its canonical binary encoding when the distinction is immaterial.

A SCIR is an identifier, not a machine address, network location, route,
credential, session or authorization. Self-certification proves correspondence
between the compact identifier and disclosed immutable certifying material. It
does not prove current ownership, membership, administration, control,
participation, application compatibility or civil identity.

The SCIR is the normative typed-reference object reused by binary carriers, the
`prp` URI scheme and PRP DNS Reference RR. Those carriers do not redefine SCIR
formation. Uppercase requirement words are to be interpreted as described by
BCP 14 (RFC 2119 and RFC 8174).

The PRP architecture describes the broader relationship-centric model and the
distinction among identities, services, machines, addresses, sessions and
relationships. It is an informative architectural reference for this document.
This document is self-contained for SCIR syntax, formation and validation and
does not make conformance depend on an architectural publication revision.

## 2. Terminology

- **reference suite:** the hash and output length used for class-specific SCIR
  formation; for IDENTITY it additionally selects canonical public-key material;
- **canonical public key:** the exact suite-defined public verification bytes;
- **Strong ID:** the suite-selected digest committing to the suite and canonical
  public key;
- **SCIR:** the complete reference-class, reference-suite and identifier tuple;
- **certifying material:** the immutable canonical input whose class-specific
  hash produces the identifier;
- **proof of possession:** protocol evidence that an endpoint controls the
  private key material corresponding to an IDENTITY SCIR;
- **carrier:** an encoding or protocol field that transports a SCIR.

“Self-certifying” means that a verifier can recompute the identifier from the
presented canonical certifying material. For IDENTITY, the verifier can then use
the committed public key to verify possession evidence defined by a consuming
protocol. A SCIR is not a certificate and does not sign itself. Recomputing an
identifier does not grant application or mutable-state authority.

## 3. Canonical binary representation

The canonical SCIR is:

```text
offset  size       field
0       1 octet    reference_header
1       2 octets   reference_suite, unsigned big-endian
3       L octets   identifier
```

`reference_header` has a class code in bits1..0 and zero in reserved bits7..2.
The complete admitted values are `0x00` IDENTITY, `0x01` ALIAS, `0x02`
MULTICAST and `0x03` GROUPCAST. A parser MUST reject every other value and MUST
NOT mask reserved bits or coerce one class to another.

`reference_suite` is a full 16-bit suite identifier, not a format version or a
session-protection suite. `L` is determined exclusively by the recognized suite
definition. There is no encoded identifier length and no inner format-version
field. An unknown suite, wrong total length, truncation or trailing byte is
invalid in a field defined to contain exactly one SCIR.

This document defines the following closed initial suite set. These assignments
are part of this specification rather than entries in an external registry.
For non-identity classes the public-key column does not apply; the hash and
identifier length still apply. All classes produce 35-octet or 51-octet
references:

| Suite | Name | Hash | IDENTITY public key | Identifier | SCIR |
| ---: | --- | --- | ---: | ---: | ---: |
| `0x0001` | ED25519-BLAKE3-256 | BLAKE3-256 | 32 | 32 | 35 |
| `0x0002` | ED25519CTX-SHA256-256 | SHA-256 | 32 | 32 | 35 |
| `0x0003` | MLDSA65-SHA256-256 | SHA-256 | 1952 | 32 | 35 |
| `0x0004` | MLDSA87-SHA384-384 | SHA-384 | 2592 | 48 | 51 |
| `0x0005` | ED25519-BLAKE3-MLDSA87-BLAKE3-256 | BLAKE3-256 | 2624 | 32 | 35 |

Suite `0x0005` encodes its canonical public key as
`Ed25519[32] || ML-DSA-87[2592]`. Both components are committed.

For suites `0x0001` and `0x0002`, the canonical public key is the exact
32-octet Ed25519 public-key encoding defined by RFC 8032. Suite `0x0001` uses
unkeyed BLAKE3 `hash` mode, beginning at output offset zero, and takes the first
32 output octets for Strong-ID formation; suite `0x0002` uses SHA-256.

For suites `0x0003` and `0x0004`, the canonical public keys are the exact
ML-DSA-65 and ML-DSA-87 public-key encodings defined by FIPS 204. Suite `0x0003`
uses SHA-256 and suite `0x0004` uses SHA-384 for Strong-ID formation. Suite
`0x0005` uses the same unkeyed BLAKE3 output rule as suite `0x0001` over its
ordered two-component public key.

The numeric values and names above are preserved from the existing PRP identity-
suite assignments. This specification projects those assignments only onto
SCIR formation: canonical public-key bytes, commitment hash and Strong-ID
length. It neither defines nor changes a possession-proof operation or proof
length. A consuming protocol that uses the same numeric value MUST define the
complete proof operation, inputs, component ordering, purpose, context,
transcript, freshness, role binding, canonical verification and rejection
behavior. Existing PRP wire profiles retain their separately specified proof
bytes and semantics; they are not imported into SCIR by the suite mnemonic.

All other 16-bit suite values are unassigned in this version and MUST be
rejected. There is no private-use range, implementation-defined allocation or
negotiation-by-observed-length. A future specification can revise the closed
set, but existing values MUST NOT be reassigned or given new semantics.

## 4. IDENTITY formation

Let `PK` be the exact canonical public key selected by suite `S`. The Strong ID
is:

```text
strong_id = HASH_S(
    ASCII("PRP-STRONG-ID-v4") ||
    S:u16be ||
    PK
)
```

`HASH_S` and the output length are fixed by the assigned suite. No NUL byte,
length prefix, textual suite spelling or carrier field is included. The SCIR is:

```text
scir = 0x00 || S:u16be || strong_id
```

The class octet is not part of the historical Strong-ID preimage. It is part of
SCIR equality and carrier binding. Prefixing it therefore does not change the
identity commitment, but substituting a different class never produces an
equivalent SCIR. This exception preserves the established IDENTITY formation;
the other classes include their class in their class-specific preimages.

Suites sharing a hash or output length remain distinct because `S` participates
in the preimage and in the SCIR. Public-key encodings MUST be canonical before
hashing; a parser MUST NOT normalize, decompress, reorder or reinterpret them.

For BLAKE3 suites, `HASH_S` is the unkeyed `hash` mode specified by C2SP BLAKE3
v1.0.0. The Strong ID is the first 32 output octets starting at XOF output
offset zero. `keyed_hash`, `derive_key`, a nonzero seek offset, or any other
output selection MUST NOT be used.

For an operational IDENTITY SCIR, each public-key component MUST successfully pass the
decoding and public-key validity checks required by its referenced algorithm.
The implementation MUST re-encode the decoded key using that algorithm's
canonical encoding and compare it octet-for-octet with `PK` before hashing.
The two components of suite `0x0005` are checked independently and retained in
their specified order. An encoding that fails decoding, validation, canonical
re-encoding, or exact comparison MUST be rejected rather than normalized.

## 5. IDENTITY verification and proof of possession

Given an expected SCIR and presented public key, a verifier MUST:

1. parse the exact SCIR and require header `0x00`;
2. resolve its reference suite and exact IDENTITY lengths;
3. validate the suite's canonical public-key encoding;
4. recompute the Strong ID using Section 4;
5. compare the complete recomputed SCIR with the expected SCIR; and
6. verify the proof of possession defined by the consuming protocol, using the
   validated public key and that protocol's exact operation and domain.

Strong-ID equality without a valid proof of possession is insufficient for
authenticated interaction. A valid proof does not authenticate a DNS name,
application parameter, alias membership, representative authorization or civil
identity unless the applicable protocol separately binds and validates it.

For represented service, the principal SCIR remains the expected identity. The
representative proves its own identity and supplies separately authenticated
principal authorization. The representative's key or identity MUST NOT replace
the principal Strong ID in the expected SCIR.

## 6. ALIAS formation

An ALIAS is a global, application-independent rendezvous identifier derived
from canonical text. Applications written by different authors obtain the same
ALIAS when they intentionally use the same canonical text and reference suite.
The ALIAS does not encode an application profile, creator, member, role,
authority or creation nonce.

The input `alias_text` MUST be a nonempty Unicode scalar-value sequence. It MUST
be encoded as UTF-8 (RFC 3629) after Unicode Normalization Form C (NFC) as
specified by Unicode Standard Annex #15, Revision 58. The canonical
UTF-8 encoding MUST contain between 1 and 1024 octets inclusive and MUST NOT
contain U+0000 or a C0/C1 control character. Case, punctuation and whitespace
are otherwise significant. Application-specific case folding or user-interface
mapping, if any, occurs before SCIR formation and is not performed by PRP.

Let `T` be those exact canonical UTF-8 octets and `S` the reference suite:

```text
alias_identifier = HASH_S(
    ASCII("PRP-SCIR-ALIAS-v1") ||
    0x01 ||
    S:u16be ||
    len(T):u32be ||
    T
)

alias_scir = 0x01 || S:u16be || alias_identifier
```

`HASH_S` uses the hash and exact output length assigned by the closed suite
table. The length prefix is the number of UTF-8 octets, not Unicode scalar
values. A verifier presented with alias text MUST canonicalize it as above,
recompute the complete ALIAS SCIR and compare every octet.

An ALIAS SCIR certifies only the committed text. It does not certify a member,
publisher, application, role or protocol. Membership and selected-candidate
claims require separate authenticated evidence. Human-readable, low-entropy
aliases are enumerable by dictionary attack; parties needing an unguessable
rendezvous value SHOULD share high-entropy alias text.

## 7. MULTICAST and GROUPCAST formation

MULTICAST and GROUPCAST are SCIR classes whose identifiers commit to their
immutable canonical genesis material. The common formation uses a distinct
domain, the exact class, the reference suite and the canonical genesis. Mutable
controllers, administrators, participants, incorporated groups, sessions,
locators and application state are not identifier fields. Their exact genesis
encoding and authority evidence are specified by the managed-reference profile;
until that profile is admitted, an implementation MUST NOT invent local
certifying material and claim an interoperable MULTICAST or GROUPCAST SCIR.

## 8. Equality, canonicalization and replacement

Two SCIRs are equal only when their complete octet strings are equal. Equality
therefore includes class, suite and identifier. Implementations MUST NOT compare
only digest bytes or infer a class or suite from length.

A change of suite or certifying material produces a different SCIR. Identity
key rotation is consequently reference replacement. Continuity, migration,
revocation and authorization across two SCIRs require a separate authenticated
application or protocol statement.

The binary SCIR has exactly one encoding. Human display MAY abbreviate it for
diagnostics, but an abbreviation is not a reference and MUST NOT be accepted as
an interoperable input.

## 9. Textual URI representation

Where a textual URI is required, the complete SCIR is carried by the `prp` URI
scheme as canonical unpadded base64url:

```text
prp:<base64url(scir)>
```

The URI carrier specification may append opaque application parameters after
the exact SCIR boundary before base64url encoding. Those bytes are not part of
the SCIR and gain no authenticity from proximity to it.

The historical experimental spelling `prp:HH:<hex>` is not part of this
specification, has no compatibility status and MUST NOT be emitted or accepted.
In particular, `prp:01:<hex>` is not an alternative encoding of suite1. A
colon inside the scheme-specific payload is invalid under the canonical URI
grammar. Implementations containing an experimental parser for that spelling
SHOULD remove it rather than provide fallback or format guessing.

## 10. DNS use

One RDATA value of the PRP DNS Reference RR can transport exactly one binary
SCIR of any admitted class. The same RRset can contain multiple independently
typed SCIRs.
DNSSEC can authenticate the association from a DNS owner name to those bytes; it does not
replace class-specific validation or, for IDENTITY, proof of possession.
Conversely, class-specific SCIR validation does not prove the DNS-name
association. A relying application that needs both
properties MUST validate both layers.

The DNS presentation form is owned by the RR specification and is not a second
general textual form for SCIRs. Generic RFC 3597 presentation remains available
for unknown DNS implementations.

## 11. Extensibility and parsing

New reference suites require a future normative revision covering their numeric
value, hash, identifier length, applicable canonical material, security
properties and deterministic vectors. A consuming protocol independently
specifies any possession profile it associates with IDENTITY. Length alone MUST
NOT identify a suite. Unknown and unassigned suites fail closed.

All four headers are SCIR classes, but they have distinct certifying material
and verification rules. They do not inherit IDENTITY key-possession semantics
merely because they share an envelope. A future incompatible SCIR grammar
requires an explicitly recognizable successor; decoders MUST NOT guess from
total length or reserved bits.

Parsers MUST validate fixed bounds before allocation and MUST distinguish an
exact-SCIR field from an enclosing carrier that deliberately permits trailing
application parameters.

## 12. Security and privacy considerations

- The security strength of a SCIR is bounded by its suite, hash and the entropy
  and authenticity of its class-specific certifying material.
- A stable SCIR enables correlation across publications and applications.
- Publishing a SCIR in DNS or a QR Code makes that stable pseudonym observable
  to parties that can read the publication or carrier.
- Replacing a SCIR does not ensure unlinkability when the same public key, an
  authenticated migration statement, or the same application account exposes
  continuity.
- Suite choice can fingerprint implementation capability or policy.
- A SCIR contains no revocation or freshness signal; carriers and caches can
  retain an obsolete reference after replacement.
- A syntactically valid or correctly formed SCIR does not prove who supplied it.
- Proof of key possession, genesis validity or alias-text equality does not
  grant service, membership, controller or administration authority.
- Downgrade or cross-suite fallback is prohibited; equal lengths are not
  compatibility.
- A verifier must bind possession evidence to the consuming protocol's exact
  purpose, transcript, roles and freshness rules.
- Private keys, session secrets and private application data are never fields of
  a SCIR.
- Display software should present verified identity information and avoid
  treating abbreviated or application-supplied labels as authoritative.

## 13. IANA considerations

This document requests no IANA action. The reference-suite namespace is closed
to the assignments in Section 3; this version neither creates an IANA registry
nor delegates suite allocation to another registry. It is intended to be the
normative identifier-format dependency cited by the separate applications for a
`prp` URI scheme and a PRP DNS RRTYPE. A future standards action can create a
registry or revise the closed set without changing the meaning of an existing
assignment.

## 14. References and vectors

The machine-readable suite table in `../registries/identity-suites-v4.tsv` and
the current PRP proof profile in `prp-wire-v4.md` mirror the assignments in this
document for repository validation; they are not external registries on which
the published SCIR format depends. Exact all-zero-public-key Strong-ID vectors
are in `../vectors/wire-v4/identity-strong-id-v4.tsv`; complete IDENTITY SCIR and
URI carrier vectors are in
`../vectors/self-certifying-identity-reference-v1.json`. Deterministic global
ALIAS vectors are in `../vectors/self-certifying-alias-reference-v1.json`.

Relevant external specifications include RFC 2119, RFC 3629, RFC 3986, RFC 4648,
RFC 8174 and RFC 8032, Unicode Standard Annex #15 Revision 58, FIPS 180-4,
FIPS 204, and C2SP BLAKE3 v1.0.0. *The
Participant Relationship Protocol (PRP): A Relationship-Centric Network
Architecture* is an informative reference for the conceptual model.
