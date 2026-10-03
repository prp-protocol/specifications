# `prp` URI scheme — IANA registration request draft

Status: PRE-SUBMISSION DRAFT. This file prepares, but does not submit, an IANA
request. Contact, change controller and a permanently available specification
reference must be completed before submission. Registry availability must be
checked again immediately before filing.

The IANA URI Schemes registry showed no `prp` entry when checked on 2026-09-16.
That observation is not a reservation of the name.

## RFC 7595 registration template

Scheme name:

> prp

Status:

> Provisional

Applications/protocols that use this scheme name:

> Applications implementing PRP use the `prp` scheme to carry a complete typed
> PRP reference and optional opaque application parameters in textual
> contexts, including QR codes, links, clipboard transfer and inter-application
> dispatch. The URI identifies a protocol reference; it is not a machine or
> network address. IDENTITY, ALIAS, MULTICAST and GROUPCAST are supported.
> Social identity sharing is one application and does not limit the scheme.

Contact:

> Gustavo Junior Alves <gjalves@gjalves.com.br>

Change controller:

> For the Provisional registration: Gustavo Junior Alves
> <gjalves@gjalves.com.br>. Upon Standards Track publication: IETF.

References:

> [TO BE SUPPLIED: permanently available URL for the admitted PRP URI scheme
> specification]
>
> “PRP Self-Certifying Identifier Reference (SCIR)”, the normative four-class
> typed-reference format transported by the scheme.
>
> RFC 3986, “Uniform Resource Identifier (URI): Generic Syntax”.
>
> RFC 4648, “The Base16, Base32, and Base64 Data Encodings”, Section 5.
>
> RFC 7595, “Guidelines and Registration Procedures for URI Schemes”.

## Scheme definition supplied with the request

The authoritative scheme specification is intended to be the admitted successor
of [the current carrier proposal](../docs/qr-identity-sharing-proposal.md). Its compact
syntax is:

```abnf
prp-uri = "prp:" encoded-payload
encoded-payload = 1*( ALPHA / DIGIT / "-" / "_" )
```

`encoded-payload` is the canonical unpadded base64url representation of:

```text
reference || application-parameters:*

reference = reference-header:u8 || reference-suite:u16be || identifier:L
```

The recognized class/suite pair determines `L`; the remainder is preserved as
opaque application parameters. Headers `0x00`, `0x01`, `0x02` and `0x03`
identify IDENTITY, ALIAS, MULTICAST and GROUPCAST. Reserved header bits and
unsupported class/suite pairs fail closed. Canonical producers use lowercase
`prp:`. Scheme
matching is case-insensitive. Padding, query, fragment, authority and relative
forms are not defined.
The discarded experimental `prp:HH:<hex>` form is not a second syntax for this
scheme and is rejected without fallback.

The scheme is a reference carrier rather than a locator. Obtaining a URI does
not select a host. Resolution and authenticated interaction are defined by the
reference class and consuming service protocol. The safe default action
is to parse and present the reference and application context. Invocation alone
does not authorize contact acceptance, network access, publication, session
establishment or another consequential operation.

### Utility and context

The scheme supplies a single interoperable textual form for binary PRP
references in URI-only transports. It avoids treating an application-specific
network endpoint as the peer and permits multiple services to consume the same
reference. The separate binary carrier remains available where raw
octets are supported; it is not itself a URI.

### Encoding and internationalization

The scheme-specific part contains only ASCII base64url characters. It has no
human-language fields and defines no distinct IRI form. Arbitrary parameter
octets are encoded inside the base64url payload; they are not interpreted or
normalized by the URI layer.

### Interoperability considerations

Implementations must not confuse ordinary base64 with base64url, admit padding,
decode through a text character set, guess reference length, or treat trailing
application bytes as a PRP reference extension. Unknown or unadmitted class/suite
combinations are not coerced into another class. A finite common payload limit and
platform-handler behavior must be settled before submission.

### Security and privacy considerations

- A syntactically valid URI does not authenticate its publisher or prove that
  the displayed reference is the one intended by a user.
- Application parameters are opaque and are not authenticated merely because
  they accompany a reference. Consumers must validate them in their own context.
- Scheme invocation must not automatically grant authority or perform unsafe
  actions. Resolution and establishment retain their protocol authentication and
  policy requirements.
- Stable references enable correlation. Applications should avoid unnecessary
  disclosure and must not place private keys or session secrets in the payload.
- Parsers must enforce finite limits before allocation, reject malformed and
  noncanonical encodings, and avoid multiple decoding or character-transcoding
  passes.
- User interfaces should display verified class-appropriate information rather than
  treating untrusted application text as authoritative, and should mitigate QR
  replacement and look-alike presentation attacks.

## Submission checklist

1. Publish a stable scheme Internet-Draft with exact parsing and vectors.
2. Retain the author as controller of the Provisional registration and request
   transition to IETF change control with Permanent registration upon Standards
   Track publication.
3. Recheck the live IANA URI Schemes registry for `prp` and confusing names.
4. Optionally obtain early public review on the IETF `uri-review` list; RFC 7595
   requires that review for Permanent requests, while allowing it for others.
5. Submit the RFC 7595 template and stable specification to `iana@iana.org`,
   initially requesting Provisional status under the First Come First Served
   procedure. A later Permanent request is subject to Expert Review.
6. Incorporate expert feedback without changing already published binary or URI
   semantics in place; version incompatible successors explicitly.

Provisional registration is appropriate while the specification and deployments
stabilize. A later request may transition the entry to Permanent status once the
scheme has a stable, citable specification and meets the permanent-registration
criteria of RFC 7595.
