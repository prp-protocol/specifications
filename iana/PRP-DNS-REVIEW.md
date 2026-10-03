# PRP DNS registration — revision 10 author review

This local package uses the complete explicit typed reference. It has not been
sent to IANA, no RRTYPE has been assigned, and no deployment or interoperability
claim is made.

## Integrated format

```
header: 1 octet | suite: 2 octets, big-endian | identifier: L octets
```

The header contains two class bits and six reserved zero bits. The class map is
IDENTITY=00, ALIAS=01, MULTICAST=10 and GROUPCAST=11, producing complete header
octets 0x00 through 0x03. Each RDATA publishes exactly one such reference. An
RRset can contain multiple distinct references without implied ordering,
preference or fallback. A class cannot be reinterpreted as another class. The
suite is the reference suite, not a session suite. There is no additional
version or length field.

For suites 1, 2, 3 and 5, L=32 and the RDATA length is 35 octets. For suite 4,
L=48 and the RDATA length is 51 octets. The proposed presentation format has
three terms: decimal class 0 through 3, decimal suite and a hexadecimal
identifier. This synthetic GROUPCAST example proves neither creation nor
administrative authority:

```
group.example. 300 IN PRP 3 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

The binary format is the same four-class envelope used by the independent
reference specification and by the generic URI/QR carrier. This DNS encoding
does not broaden class-specific formation, authority or admission rules. The
header does not change Strong ID cryptographic formation.

## Changes and retained boundaries

* Revision 6 used 34/50-octet RDATA and a two-term presentation format. It is
  superseded and preserved in Git history; there is no old/new fallback.
* The association is DNS name to an unordered set of references, not to a
  machine, IP address, location, route or reverse lookup. DNS remains optional.
* Each RDATA contains one reference. Identical duplicates add no reference, and
  DNS ordering selects or prioritizes none of them.
* DNS grants no representation, ALIAS association, MULTICAST control, genesis,
  GROUPCAST administration or GROUPCAST participation.
* A DNS-authoritative binding requires trustworthy validation, including for
  redirections. HS2 alone does not prove the name binding, and independent pins
  are not broadened.
* RRset changes prove no rotation, migration, delegation or equivalence. TTL
  expiry does not revoke sessions or prove the latest publication; cache,
  replay and serve-stale limitations remain applicable.
* Four-class DNS storage does not constitute final admission of class protocols,
  authority codecs or operational wire behavior.
* DNS treats identifiers as opaque octets with suite-determined lengths.
  Formation, authority and lifecycle belong to the PRP specification for each
  class and may evolve without changing this RRTYPE while the common envelope
  remains stable.

## Suggested reading order

1. This review note.
2. [Complete revision 10 specification](../specifications/prp-dns-reference-rr-v1.md).
3. [PRP Self-Certifying Identifier Reference (SCIR)](../specifications/prp-self-certifying-identity-reference-v1.md).
4. [RFC 6895 A–J preparation](PRP-RRTYPE-APPLICATION.txt).
5. [Document rights](../DOCUMENT-RIGHTS.md).

Identity and suite formation follow the current
[PRP Wire Protocol Version 4](../specifications/prp-wire-v4.md) and
[identity suite registry](../registries/identity-suites-v4.tsv). The reference
header belongs to the successor reference envelope; it does not reinterpret
wire-v4.

## Bounded evidence

The source repository checker covers the twenty combinations of four classes
and five suites in `vectors/dns-reference-v1.json`: codec, canonical
presentation, reserved bits, lengths, unordered multi-reference RRsets,
fallback rejection and equality of the complete reference. Trust, caching and
class-authority models remain symbolic. They do not validate DNSSEC, acquire
trusted time, execute PRP class protocols or use a resolver or authoritative
server. Vector checksums bind framing, not signatures.

## Gates before submission

1. Independent DNS protocol, operations, privacy and security review.
2. Final author review of this note, the examples, application and exact
   submission artifacts.
3. A live DNS Parameters registry check and RFC 6895 Expert Review preparation.
4. Completion of the actual submission date and use of the IANA-assigned value;
   no private or placeholder TYPE value is an assignment.
5. Stable representation dependencies without treating DNS publication as HS2
   or class-protocol admission.

DNS and IANA process references were last checked on 2026-09-08 and were not
rechecked merely for this English-language correction. This package promises
neither approval nor submission.
