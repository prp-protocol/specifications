# DNS publication of PRP references

Status: Working Draft revision10 for author review; NOT submitted or assigned.
The user approved complete explicit references: one-octet header (two class
bits, six reserved zero bits), reference suite:u16be and suite-sized identifier.
This profile accepts the four defined reference classes: IDENTITY, ALIAS,
MULTICAST and GROUPCAST. Each RDATA contains exactly one reference; an RRset can
contain multiple distinct references without DNS ordering or priority semantics.
Final coordinated DNS review and author submission approval remain gates. No
RRTYPE, reference-class interaction or successor HS2 is admitted here. The DNS
carrier is intentionally independent of class-specific identifier formation,
authority and lifecycle semantics.
Author: Gustavo Junior Alves, GJ LABS, gjalves@gjalves.com.br.
Proposed mnemonic: PRP. RRTYPE: TBD by IANA.

Revision6's class-less bytes and two-token presentation are superseded.
Revision7/8 transition notices and historical codec remain in Git history;
the revision6 checker is retained in tests/historical/. No old/new fallback
parser is defined. This revision is a complete current review text, not an
append-only notice over obsolete submission examples.

## 1. Purpose and boundary

This proposal maps a DNS owner name to one or more typed PRP references. A
reference can name an identity, alias, multicast or groupcast. It is not a
machine address. Publication does not imply civil identity, physical location,
reachability, authority to invoke a service, alias membership, multicast control,
groupcast participation or application agreement.

DNS is an optional naming integration. PRP establishment and discovery continue
to operate without DNS or IP. No reverse namespace, PTR convention, IP address,
port, rendezvous locator, route or content ID is defined. The DNS association
does not become authority for the referenced object. Identity representatives,
alias membership claims, multicast controller evidence, groupcast genesis,
administration events and participation proofs are not fields of this RR.

Uppercase requirement words express proposed requirements under BCP 14
(RFC 2119 and RFC 8174); they do not authorize deployment of an unassigned type.
The published architecture Internet-Draft -00 is unchanged.

## 2. Owner, class and RDATA

The record is an ordinary data RR in class IN at the exact service name
requested by the application. No mandatory underscored prefix or special
additional-section processing is defined. Existing DNS owner-name, CNAME,
DNAME, wildcard and RRset rules are not changed. An alias chain used to obtain
a trusted association MUST itself be authenticated, not just its final RRset.
An application MUST NOT infer a PRP binding from an A, AAAA or PTR response.

Authentication covers the complete redirection from the requested name. For
DNAME, validate the signed DNAME and correct CNAME synthesis; a synthesized
CNAME does not require a separate signature (RFC 6672 section 5.3). Wildcard
answers require validation of the expansion, not just the final RRset signature
(RFC 4035 section 5.3.4). Neither rule introduces PRP-specific DNS processing.

RDATA contains exactly:

```
reference_header:u8 || reference_suite:u16be || identifier[L]
```

At offset0, header bits1..0 encode the reference class and bits7..2 are reserved
and MUST be zero. The shared reference map is IDENTITY=00, ALIAS=01,
MULTICAST=10, GROUPCAST=11. The complete valid header octets are consequently
`0x00`, `0x01`, `0x02` and `0x03`. It is a real class header, not an inner
version or absence marker. Every other header MUST be rejected by a PRP-aware
consumer. DNS class IN is a different field from the PRP reference class.

Offsets1..2 carry the full16-bit reference suite in network byte order. The
identifier begins at offset3; L is its assigned reference length. For IDENTITY
this is the Strong ID derived under that identity suite. For ALIAS it commits to
global canonical text. For MULTICAST and GROUPCAST it commits to immutable
canonical genesis under the applicable profile. The reference suite is not a session suite,
signature algorithm inferred for an external statement or format version. There
is no identifier-length field, optional field, TLV or compression pointer. DNS
RDLENGTH supplies the enclosing length but MUST equal 3+L for a known suite. No
domain names occur inside RDATA; canonical DNSSEC processing leaves its octets
unchanged.

For every admitted class, the RDATA is exactly one
[`PRP Self-Certifying Identifier Reference (SCIR)`](prp-self-certifying-identity-reference-v1.md).
Its envelope follows the shared canonical layout in
`../docs/reference-canonical-layout-proposal.md` C1.2; it is not a DNS-specific codec.
The suite/identifier substructure retains PRP wire v4 section9 identity formation;
the prefixed reference header is a successor structure, not wire-v4 bytes.
The common envelope is the only class-independent dependency of this DNS format.
ALIAS, MULTICAST and GROUPCAST have distinct certifying material and do not
inherit IDENTITY proof-of-possession semantics. Their current semantic model is defined by
[`PRP Multicast and Groupcast Reference Model`](prp-multicast-groupcast-v1.md)
and the alias sections of `prp-reference-representation-v2.md`. Operational DNS
use of a reference depends on the consumer supporting that class's identifier
formation and authority rules. Those rules are not required to store, forward,
compare or parse the opaque DNS RDATA and can mature without changing this
RRTYPE while the common envelope and suite-selected length stay stable. DNS
storage does not supply the missing semantics.

For the closed initial PRP reference-suite set:

| Suite  | Assigned name                     | Identifier bytes | RDLENGTH |
| :----- | :-------------------------------- | ---------------: | -------: |
| 0x0001 | ED25519-BLAKE3-256                |               32 |       35 |
| 0x0002 | ED25519CTX-SHA256-256             |               32 |       35 |
| 0x0003 | MLDSA65-SHA256-256                |               32 |       35 |
| 0x0004 | MLDSA87-SHA384-384                |               48 |       51 |
| 0x0005 | ED25519-BLAKE3-MLDSA87-BLAKE3-256 |               32 |       35 |

Suites 0x0001 and 0x0005 both use BLAKE3-256 for the identity commitment, but
their key and proof formats differ: the latter binds both classical and
post-quantum components. As defined by PRP wire v4 section 9:

```
strong_id = HASH("PRP-STRONG-ID-v4" || identity_suite_id:u16 || public_key)
```

The hash input includes the suite identifier and that suite's exact canonical
public key (both key components for 0x0005). Sharing a hash algorithm or output
length therefore does not make suites or identities interchangeable. The same
distinction applies to suites 0x0002 and 0x0003, which share SHA-256. Public keys
and proofs are verified through PRP; they are not additional DNS RDATA fields.
The obsolete experimental textual form `prp:HH:<hex>` is neither DNS
presentation nor a compatible SCIR encoding and MUST NOT be accepted here.

For IDENTITY, the suite selects the SCIR public-key and commitment semantics.
For another class, the same numeric suite selects that class's admitted
identifier-formation profile and the common output length; it does not turn the
identifier into a key or import the identity proof algorithm. New suite or
class/suite definitions require explicit assignment; length alone never
identifies either one. An incompatible RDATA structure requires a separately
reviewed extension mechanism or another RRTYPE, not heuristic parsing. No new
IANA subregistry is requested. Document revision numbers and the filename's v1
label are editorial identifiers, not transmitted fields.

## 3. Presentation and examples

The proposed presentation has three whitespace-separated tokens: decimal
reference class, decimal reference suite, and an even-length hexadecimal
identifier. The accepted class tokens are `0` (IDENTITY), `1` (ALIAS), `2`
(MULTICAST) and `3` (GROUPCAST), each encoding the corresponding zero-reserved
header; no extra flags or reserved values are expressible. Hex input is
case-insensitive; canonical output is lowercase. The identifier is one token
without quotes, separators, 0x prefix or address-scheme prefix.
Decimal tokens contain ASCII digits only, without signs or leading zeros;
the single-digit class tokens are permitted. Suite numbers are1..65535 and must
be supported to produce a usable reference. This is proposed DNS presentation,
not a QR textual alphabet or URI scheme.
Generic unknown-type representation remains available under RFC 3597.

Illustrative only, using synthetic identifier bytes and no authority proof:

```
identity.example.  300 IN PRP 0 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
alias.example.     300 IN PRP 1 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
stream.example.    300 IN PRP 2 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
group.example.     300 IN PRP 3 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

Their RDATA values are 35 bytes. The GROUPCAST example is hexadecimal
(header03, suite0005, identifier00..1f):

```
030005000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

PRP is only a proposed mnemonic. No numeric TYPE example or private-use code
is assigned here. A real RFC 3597 zone entry must use the actual assigned code.

## 4. Parsing, lookup and selection

A format-aware consumer MUST reject truncated headers and invalid lengths for
known suites. Trailing octets are invalid, not extensions. Headers `0x00` through
`0x03` are accepted; every other header is rejected. No class-less revision6,
version-prefixed or legacy-target fallback is defined. Reserved bits cannot be
masked off; lengths cannot select another grammar. This proposed RRTYPE defines
its grammar; no deployed earlier revision of that unassigned type is claimed.
Unknown suites MUST NOT yield a candidate or be reinterpreted as a known suite.
An authenticated RRset containing malformed known-suite records MUST
fail this binding lookup rather than silently yield a partial association.
Opaque DNS storage/forwarding under RFC 3597 does not require PRP validation.

An RRset MAY contain multiple distinct PRP references. Equality uses the complete
class || reference_suite || identifier, not the identifier bytes alone.
Byte-identical duplicates do not add another reference. RRset ordering is not
significant and MUST NOT express priority, preference, fallback or rotation
order. An empty answer supplies no binding; an unsupported reference supplies no
usable candidate for an implementation that does not support it.

The application selects only among references whose class it requested and
whose suite and class semantics it supports. Selection policy is outside DNS and
MUST NOT use DNS wire order. Failure to authenticate or authorize a selected
reference MUST NOT silently fall back to another class, a weaker suite or an
unverified record. Applications needing preference or weight require a separate
authenticated application mechanism. Consumers impose bounded RRset size,
alias-chain traversal, record processing and connection attempts; exceeding a
local bound fails the lookup rather than accepting an arbitrary truncated subset.

DNS TTL and authenticated validity constrain reuse for new lookups. Negative
answers do not authorize fallback to unauthenticated naming. Timeout, SERVFAIL
or truncation is not an authenticated absence. Ordinary DNS truncation/retry
handling applies; a partial RRset MUST NOT be used as a complete binding.

DNS authenticates only the owner-name association. Class-specific validation is
mandatory after selection:

- IDENTITY requires its class-specific SCIR formation and protocol proof of possession, including
  representation evidence when applicable;
- ALIAS requires recomputation from canonical text plus the consuming discovery
  and selected-candidate binding rules;
- MULTICAST requires controller and current transmission authority evidence; and
- GROUPCAST requires the applicable genesis, administration, participation and
  current-state evidence.

DNS TTL is not the validity period of any of those proofs. An implementation
lacking the admitted semantics for a class MUST treat that reference as
unsupported, not reinterpret it as IDENTITY or opaque application authority.
DNS cannot stand in for HS2, service authorization, group administration or
Application Agreement.

## 5. Trust, rotation and privacy

When DNS is the authority for the name-to-reference association, the consumer
MUST obtain DNSSEC validation with a trusted validation path, including any
alias chain. A bare AD flag from an untrusted resolver is not such evidence.
Bogus or indeterminate validation fails; insecure DNS is not authenticated
name binding. Independently trusted reference or identity pins can instead
restrict DNS to untrusted hints, but DNS MUST NOT replace or broaden those pins.
Class-specific authentication by itself
does not authenticate the association to the originally requested DNS name.
DNSSEC authenticates the publisher's assertion, not civil identity or benevolence.
These are consumer requirements; they do not change DNS server processing.

Publishers can add or remove references in an RRset. Coexistence does not prove
migration, delegation, equivalence or an ordered transition among them. Different
caches may temporarily expose different valid RRset versions. An application
MUST NOT merge versions as though they were one simultaneously authenticated
set. A cached RRset MAY remain usable for a new lookup while its accepted DNS
TTL and authenticated validity remain unexpired and local policy permits.
Publication of a replacement does not extend the previous binding's validity.
Expired DNS evidence MUST NOT be used as a current authoritative name binding;
independently trusted pins retain their separate authority.

DNSSEC validation does not establish that a binding is the newest publication.
An older signed RRset may still validate when replayed before signature expiry;
TTL alone is not a global withdrawal deadline. Cache lifetime is constrained
by the validated TTL and signature bounds in RFC 4035 section 5.3.3, not a
fresh full TTL restarted by an application's own cache. A TTL-zero answer may
be used for the transaction in progress if otherwise valid, but not cached for
later use. This does not make expired signatures acceptable.

RFC 8767 permits resolvers to serve stale data with a positive response TTL.
Such a TTL alone cannot demonstrate compliance with this profile's no-expired-
binding policy. The consumer's trusted validation path must enforce that policy
or supply sufficient trusted freshness information; a response known to be stale
is not current authoritative binding evidence. If the required validity cannot
be established, fail the DNS-authoritative lookup rather than infer freshness
from TTL or AD alone. This is a consumer trust condition, not a new DNS field,
server configuration prescription or proof of globally latest state.

Even a DNSSEC-validated replacement MUST NOT replace or broaden an independently
trusted pin. A different pinned reference requires a separately authorized pin
update; DNS change alone is insufficient. A pin mismatch fails
the binding for that policy, not an invitation to retry without the pin.

A DNS replacement, removal or TTL expiration MUST NOT by itself terminate,
reidentify or transfer an established identity, multicast, groupcast,
relationship or session. Existing class-specific authority and application
lifecycle rules continue to apply. DNS rotation is neither delegation nor a
session migration.

Publication reveals an association and may enable cross-name correlation when
references are reused. Publishing GROUPCAST relationships or MULTICAST names can
expose social structure and communication purpose even without embedded
authority evidence. DNSSEC does not provide query confidentiality. Operators
should publish only references intended to be public. Compromise of DNS
publishing or validation authority can redirect first contact; class-specific
proof validates the substituted object, not that the substitution was wanted.

## 6. Why a distinct RR

A/AAAA contain network addresses rather than PRP references. TXT could carry an
application convention, but a dedicated type provides typed queries and a
single binary class/suite/identifier schema. URI (RFC 7553) is a viable alternative for
URI discovery; this proposal instead publishes reference bytes without locator,
priority or weight semantics, and does not depend on registration of a PRP URI
scheme. This is a design tradeoff, not a claim that TXT/URI cannot carry data.

HIP (RFC 8005) is the closest identity-oriented precedent, but carries HIP
identifiers, public keys and optional rendezvous names with HIP-specific
processing. Reusing its algorithm fields for PRP suites would conflate distinct
protocols. PRP RR carries one typed PRP reference per RDATA. The application
must justify this distinct semantic need to the designated DNS expert.

## 7. IANA and review gates

Request one data RRTYPE, mnemonic PRP (subject to availability and expert
approval), numeric value TBD, through RFC 6895 Appendix A. No existing value
is repurposed. No new DNS class, opcode, label, reverse tree or underscored-node
registration is requested. Unknown-type transport is compatible with RFC 3597;
presentation and consuming applications need type-aware support, not special
resolver behavior. This document makes no deployment or interoperation claim.

Before submission: final author approval of the complete submission (the
four-class scope, multiple-reference RRset semantics and complete-reference
design are approved, not submission); final coordinated canonical-reference
review and stable public specification of the envelope, class codes and closed
suite-sized identifier lengths;
confirmation of rights in any additional material included in the submission;
DNS/security review of the proposed trust and selection rules. An architecture
draft alone does not fix the binary reference formats. These are open submission
gates, not IANA assignments. See `../iana/PRP-RRTYPE-APPLICATION.txt`.
Reference the exact canonical grammar/code-map OID and closed suite definitions
before sending. Class-specific formation and authority can remain in separate
PRP specifications because the DNS carrier treats identifiers as opaque; this
does not waive class-specific admission before an application acts on a record.

The author has authorized CC BY 4.0 for this DNS specification and its author-owned
preparatory material; see `../DOCUMENT-RIGHTS.md`. Final submission approval
remains separate. REG-001 concerns PRP suite governance, not a prerequisite to
DNS storage or a new IANA subregistry requested by this RR. DNS transports the
opaque RDATA; PRP consumers interpret its explicitly referenced class format.

## References

- PRP wire v4 section 9 and identity-suites-v4.tsv, pinned above.
- [RFC 6895, allocation process](https://www.rfc-editor.org/rfc/rfc6895.html).
- [RFC 3597, unknown types](https://www.rfc-editor.org/rfc/rfc3597.html).
- [RFC 4033, DNSSEC model](https://www.rfc-editor.org/rfc/rfc4033.html).
- [RFC 4035, DNSSEC validation and cache bounds](https://www.rfc-editor.org/rfc/rfc4035.html).
- [RFC 6672, authenticated DNAME synthesis](https://www.rfc-editor.org/rfc/rfc6672.html).
- [RFC 8767, stale answers and TTL](https://www.rfc-editor.org/rfc/rfc8767.html).
- [RFC 8005, HIP DNS extension](https://datatracker.ietf.org/doc/html/rfc8005).
- [RFC 7553, URI RR](https://www.rfc-editor.org/rfc/rfc7553.html).
- [IANA DNS registry](https://www.iana.org/assignments/dns-parameters).

DNS validation/allocation sources rechecked 2026-09-08. This is a local review proposal, not an uploaded
Internet-Draft or an approved standards-body publication.
