# PRP Contact Identity Admission and Legacy Migration Contract

- Version: 1
- Status: Consumer integration contract; migration profile not assigned
- Author: Gustavo Junior Alves, GJ LABS, <gjalves@gjalves.com.br>

## 1. Scope

This document defines the bounded admission contract for an application
operation that selects a PRP contact identity from the generic `prp` URI/QR
carrier. It does not redefine the carrier or the PRP Self-Certifying Identifier
Reference (SCIR), alter an identity suite, or grant contact authorization.

The generic carrier remains valid for IDENTITY, ALIAS, MULTICAST and GROUPCAST.
The contact-selection operation defined here accepts only IDENTITY because its
result is an expected principal identity. Rejecting another class for this
operation does not make that class invalid for its own application.

This document also defines the information required before a stored VELOZ
experimental `prp:01:...` value can be considered for migration. It defines no
approved conversion and authorizes no rewrite of VELOZ data.

Uppercase requirement words are interpreted as described by BCP 14 (RFC 2119
and RFC 8174).

## 2. Dependencies

The current generic carrier is defined by the PRP URI/QR carrier specification:

```text
P = reference || application_parameters
text = "prp:" || canonical_unpadded_base64url(P)
```

The complete reference is:

```text
reference_header:u8 || reference_suite:u16be || identifier:L
```

The SCIR specification defines all four reference classes. This operation
admits only the IDENTITY class, its closed identity-suite set,
canonical public-key encodings and Strong-ID formation. The consuming PRP
protocol defines the proof of possession; PRP wire v4 uses HS2.

## 3. Contact carrier admission

A contact importer MUST process a textual input in the following order:

1. enforce a documented finite input bound before base64url decoding or decoded
   allocation;
2. recognize the `prp` scheme according to the generic carrier, reject query,
   fragment, authority, padding, whitespace, ordinary-base64 characters,
   nonzero unused bits and noncanonical payload encoding;
3. decode exactly once and enforce the finite decoded-payload bound;
4. parse the complete reference boundary from the recognized class/suite pair,
   never from the remaining byte count;
5. require `reference_header == 0x00` for this contact operation;
6. interpret the next two octets only as `identity_suite:u16be` and require an
   assigned SCIR suite;
7. retain the exact Strong ID and exact opaque application-parameter remainder
   separately; and
8. produce an unauthenticated contact candidate, not an accepted contact.

Canonical producers emit lowercase `prp:` and unpadded base64url. URI scheme
matching is case-insensitive under the generic carrier; accepting another ASCII
case does not make it canonical output. An importer MUST NOT lowercase or
otherwise normalize the case-sensitive encoded payload.

For this operation, ALIAS returns `CONTACT_CLASS_UNSUPPORTED`, and MULTICAST or
GROUPCAST returns `CONTACT_CLASS_MISMATCH`. These results are scoped operation
failures, not declarations that the generic references are malformed.

Application parameters are not part of the SCIR. They are untrusted input until
the selected application validates and, where necessary, authenticates them.
They MUST NOT select another identity suite, replace the expected SCIR, grant
contact access or authorize personal-data fields.

## 4. Public-key binding and authentication

Given the contact candidate SCIR and a subsequently supplied remote canonical
public key `PK`, the importer or connection layer MUST:

1. obtain `identity_suite` from the SCIR, not from an application profile,
   address profile, key length or algorithm guess;
2. require exactly the suite-defined public-key components and lengths;
3. decode, validate and canonically re-encode each component;
4. compare the re-encoding octet-for-octet with `PK`;
5. recompute:

   ```text
   strong_id = HASH_S(
       ASCII("PRP-STRONG-ID-v4") ||
       identity_suite:u16be ||
       PK
   )
   ```

6. compare the complete reconstructed SCIR, including header and suite, with
   the candidate SCIR; and
7. verify the consuming protocol's proof of possession, including its purpose,
   transcript, roles and freshness. For the current contact connection this is
   the applicable HS2 identity proof.

Syntax success is `CONTACT_CANDIDATE`, not authentication. Successful key
binding without HS2 is `CONTACT_KEY_BOUND`, not an established peer. Only the
successful proof stage yields `CONTACT_IDENTITY_AUTHENTICATED`. Application
consent and field authorization remain later, separate decisions.

A QR code, URI, discovery handle, application parameter, human label, stored
legacy address or matching Strong ID alone MUST NOT authorize a contact, access
to personal data, or session establishment.

## 5. Versioned legacy migration proposal

The experimental VELOZ spelling `prp:01:...` is not accepted by the canonical
`prp` parser. A migration tool MUST recognize it only through an explicitly
selected migration contract, never through fallback in the current parser.

The proposal identifier for the information contract in this section is:

```text
VELOZ-PRP-01-MIGRATION-INPUT-v1
```

This identifier describes required evidence only. It is not an approved
conversion profile and assigns no meaning to `01`.

For each legacy record, all of the following source evidence is required:

- exact original stored value as bytes and, if originally textual, its exact
  text encoding without normalization;
- source product and component name;
- exact source release/version, build identifier and source commit when known;
- exact legacy grammar/decoder revision and an immutable hash or archival
  reference for that decoder specification;
- raw `address_profile` value and its source-version definition;
- raw legacy identifier/digest bytes and the exact historical preimage,
  algorithm, domain separator, component order and output selection used to
  derive them;
- exact key algorithm and key-format identifiers;
- complete original public verification key bytes, including every component,
  their order and original encoding;
- provenance binding that key to the stored record and evidence that the record
  was not assembled from unrelated fields;
- any historical possession-proof operation and available proof evidence; and
- stable record identity, storage generation and collision/duplicate history so
  a migration cannot silently merge or overwrite two stored identities.

`address_profile` MUST NOT be interpreted as `identity_suite`. Numeric equality
with a current suite value is insufficient. Key length, digest length, the
legacy token `01`, or a familiar algorithm name is likewise insufficient.

An eventual conversion would additionally require a separately reviewed and
versioned mapping profile that states which complete legacy metadata tuple maps
to which assigned PRP identity suite, how the canonical public key is produced,
and how continuity is authenticated. The converter would then recompute the
SCIR from the full canonical public key and require current proof of possession.
It would store the new SCIR as a distinct identity plus an authenticated
continuity record; it would not mutate the old identity in place.

Until that mapping profile is approved, even a complete input record returns
`MIGRATION_PROFILE_REQUIRED`. Missing or ambiguous source/key metadata returns
`LEGACY_METADATA_INCOMPLETE`. A mismatching historical derivation, public key or
provenance returns `LEGACY_EVIDENCE_MISMATCH`. None is a canonical contact.

## 6. Acceptance criteria

### 6.1 Carrier admission

PASS requires all of the following:

- canonical unpadded base64url payload;
- a complete recognized reference boundary;
- class IDENTITY;
- an assigned identity suite and suite-sized Strong ID;
- exact preservation of trailing application parameters; and
- finite pre-decode and decoded bounds.

This PASS produces only `CONTACT_CANDIDATE`.

### 6.2 Identity authentication

PASS requires canonical validation of the complete suite public key, exact
Strong-ID recomputation and a valid current HS2 proof bound to the expected
SCIR. This PASS produces `CONTACT_IDENTITY_AUTHENTICATED`, not application
consent or field authorization.

### 6.3 Legacy migration

There is no PASS mapping in version 1. A record can become
`MIGRATION_REVIEWABLE` only when every source-evidence field in Section 5 is
present and internally consistent. It remains `MIGRATION_PROFILE_REQUIRED`
until a future approved mapping profile and current possession proof exist.

## 7. Validation boundaries

The finite vectors accompanying this contract exercise carrier syntax, class
admission, exact parameter separation, actual Strong-ID recomputation for one
known public key, malformed inputs and migration-evidence classification.

They do not execute HS2, establish a network session, approve a contact, grant
personal-data access, validate Android dispatch/scanning, reproduce VELOZ's
historical decoder, or establish an empirical QR optical capacity. The checker's
3072-octet decoded bound is a local validation bound, not a protocol-wide or QR
capacity assignment.

## 8. Security and privacy considerations

A contact QR can be replaced, replayed, photographed or correlated. Verified
identity information must be displayed only after key binding and possession
proof. Application parameters and labels remain untrusted presentation data.

Migration is especially sensitive to identity substitution. Guessing a suite,
normalizing a legacy value, accepting a partial public key, joining records by
display name or overwriting the stored identity destroys the evidence needed to
detect substitution and collision. Ambiguity therefore fails closed.

Private keys and session secrets are neither carrier nor migration fields and
MUST NOT be requested, exported or stored by this contract.
