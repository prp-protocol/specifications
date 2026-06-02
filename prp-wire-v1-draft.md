# PRP Wire v1 Draft

Status: Draft

This document defines the peer-visible PRP v1 wire contracts that are shared
by conforming implementations. The canonical numeric registry for profile ids,
suite ids, frame ids, status ids, routing ids, bootstrap-provider ids, and
service ids is `wire-registry-v1.md` in this repository.

This document does not define implementation APIs, tool behavior, local
operator storage, dataplane backend selectors, application payload semantics,
or carrier-specific framing. Reference implementations may keep implementation
conformance notes in their own repositories.

## Integer Encoding

All integer fields are unsigned and encoded in network byte order unless a
field explicitly says otherwise. Byte offsets are zero-based.

Receivers fail closed for unknown versions, profile ids, suite ids, envelope
kinds, record types, packet types, status codes, status details, non-zero
reserved bytes, unsupported lengths, or locally disabled features.

## PRP Units and Carrier Framing

PRP operates on complete PRP units. A PRP unit is either a handshake unit or an
encrypted packet unit.

PRP expects a lower framing layer to deliver exactly one complete PRP unit to
the PRP decoder and to identify that the delivered bytes are PRP bytes.
Examples include an EtherType, a UDP well-known port, an IP protocol number, a
BLE characteristic, an IPC endpoint, an AF_XDP queue selection, a file record,
or any carrier binding that provides equivalent demultiplexing and length.

The base PRP wire format does not define byte-stream delimitation, carrier
fragmentation, media checksums, physical retransmission, or resynchronization
markers. Those mechanisms belong below PRP.

Carrier bindings that preserve PRP unit boundaries may deliver each unit
directly to the appropriate PRP decoder. Carrier bindings that do not preserve
PRP unit boundaries must use an encapsulation profile that reconstructs exactly
one complete PRP unit before passing it to the PRP decoder.

Encapsulation profiles must not define PRP relationship semantics, peer
identity, route identity, service authorization, packet replay rules,
cryptographic suite negotiation, or application payload semantics.

The base wire format does not require a self-identifying ASCII magic at byte
offset zero. The lower layer already provides unit boundaries and PRP traffic
selection.

## Transcript Context

PRP binds the selected version, profile, and cryptographic suite into the
Noise transcript with this compact binary context:

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 0 | 1 | `uint8` | PRP wire version, currently `1` |
| 1 | 1 | `uint8` | profile id |
| 2 | 1 | `uint8` | suite id |

This 3-byte context is not a negotiation transcript by itself. It is a
deterministic value derived from local configuration, relationship documents,
or a lower-layer carrier selection. Both peers mix the exact same context into
the Noise transcript as prologue before the first handshake message. A mismatch
aborts the handshake. This context is not a PRP packet header, carrier
header, or self-identifying magic. It exists only as Noise prologue input and
is not used by a receiver to discover PRP traffic; the lower carrier binding
already performs that selection.

A peer must not infer a different profile id or suite id after seeing
handshake bytes, and must not retry with another suite as an automatic
fallback.

Profile ids are assigned by `wire-registry-v1.md`. The v1 profile set is:

| Value | Name | Meaning |
| ---: | --- | --- |
| `0x01` | `PRP_PROFILE_DIRECT_ANONYMOUS` | Direct first-contact `NN`. |
| `0x02` | `PRP_PROFILE_ROUTED_BOOTSTRAP` | Temporary routed bootstrap session. |
| `0x03` | `PRP_PROFILE_CONFIGURED_RELATIONSHIP` | Final relationship-selected session. |

## Cryptographic Suites

PRP identifies cryptographic suites on the wire and in transcript context with
a one-byte suite id. Long suite names are diagnostic labels only; they are not
transmitted as suite selectors.

Suite ids are assigned by `wire-registry-v1.md`. The v1 suite set is:

| Value | Pattern | Name |
| ---: | --- | --- |
| `0x00` | none | `PRP_SUITE_NONE_DEBUG` |
| `0x01` | `NN` | `PRP_SUITE_NN_X25519_CHACHAPOLY_BLAKE3_V1` |
| `0x02` | `NK` | `PRP_SUITE_NK_X25519_CHACHAPOLY_BLAKE3_V1` |
| `0x03` | `IK` | `PRP_SUITE_IK_X25519_CHACHAPOLY_BLAKE3_V1` |
| `0x11` | `NN` | `PRP_SUITE_NN_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` |
| `0x12` | `NK` | `PRP_SUITE_NK_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` |
| `0x13` | `IK` | `PRP_SUITE_IK_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` |

`0x00` is `NONE_DEBUG`. It performs no Noise handshake, key derivation, AEAD,
authentication, confidentiality, integrity protection, or replay protection. It
must not be selected by fallback. It must be rejected unless explicitly enabled
by local debug/test policy. Production relationships, routed bootstrap,
cartorial flows, aggregator dataplane, and public transports must reject it.

Unknown suite ids are unsupported. A peer must not silently map an unknown
value to a known suite, and must not treat a classical suite id as compatible
with a hybrid suite id.

The Noise initialization name is derived deterministically from suite id, not
from carrier text. Diagnostic names are display and implementation labels only;
they are not transmitted as suite selectors.

## PRP BLAKE3 KDF

The DH output length, public key length, hash length, and transport key length
are all 32 bytes. `HASH(data)` is the first 32 bytes of BLAKE3 over `data`.
`MixHash(h, data)` is BLAKE3 over `h || data`.

`MixKey(ck, input)` uses this PRP BLAKE3 KDF:

| Step | Output |
| ---: | --- |
| 1 | `temp_key = BLAKE3-keyed(key = ck, input)` |
| 2 | `out1 = BLAKE3-keyed(key = temp_key, 0x01)` |
| 3 | `out2 = BLAKE3-keyed(key = temp_key, out1 || 0x02)` |

The new chaining key is `out1`; the handshake AEAD key is `out2`. Session
splitting uses the same KDF with empty input and maps `out1` and `out2` to
initiator-to-responder and responder-to-initiator transport keys respectively.

Profiles that derive additional keys from a completed session must use
domain-specific KDF labels or an equivalent suite-defined key schedule.
Routing, application data, route control, ACK/NACK, padding, and future
cover-traffic domains must not rely on ambiguous reuse of one undifferentiated
key.

## Noise Message Payloads

All public keys in the classical suites are X25519 public keys. Encrypted
fields are ChaCha20-Poly1305 ciphertext plus 16-byte tag.

Classical `NN` message 1:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | initiator ephemeral public key |

Classical `NN` message 2:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | responder ephemeral public key |
| 32 | 16 | encrypted empty authenticator |

Classical `NK` has the same message byte shape as classical `NN`; the
responder static public key is known from relationship state and mixed into the
transcript according to the selected suite.

Classical `IK` message 1:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | initiator ephemeral public key |
| 32 | 48 | encrypted initiator static public key |

Classical `IK` message 2:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | responder ephemeral public key |
| 32 | 16 | encrypted empty authenticator |

## Hybrid ML-KEM-768 Noise Payloads

The hybrid suites use X25519 plus one ML-KEM-768 shared secret. They use the
same AEAD, BLAKE3 hash, and PRP KDF as the classical suites. Every successful
session secret depends on both the X25519 DH result and the ML-KEM-768 shared
secret.

Hybrid `NN` message 1:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | initiator X25519 ephemeral public key |
| 32 | 1184 | initiator ML-KEM-768 ephemeral public key |

Hybrid `NN` message 2:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | responder X25519 ephemeral public key |
| 32 | 1088 | ML-KEM-768 ciphertext encapsulated to message-1 public key |
| 1120 | 16 | encrypted empty authenticator |

Hybrid `NK` and `IK` use the responder ML-KEM-768 public key from the
relationship document. The initiator encapsulates to that static KEM public key
in message 1. The responder must abort if ML-KEM decapsulation fails.

Hybrid `NK` message 1:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | initiator X25519 ephemeral public key |
| 32 | 1088 | ML-KEM-768 ciphertext to responder static KEM public key |

Hybrid `NK` message 2 is identical in shape to classical `NK` message 2.

Hybrid `IK` message 1:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 32 | initiator X25519 ephemeral public key |
| 32 | 1088 | ML-KEM-768 ciphertext to responder static KEM public key |
| 1120 | 48 | encrypted initiator static X25519 public key |

Hybrid `IK` message 2 is identical in shape to classical `IK` message 2.

The hybrid transcript order is deterministic:

1. initialize `ck` and `h` with the suite-id-derived diagnostic name;
2. for `NK` and `IK`, mix the responder static X25519 public key and then the
   responder static ML-KEM-768 public key into `h`;
3. mix the 3-byte PRP transcript context into `h` as prologue;
4. mix the initiator X25519 ephemeral public key into `h`;
5. mix the ML-KEM public key or ciphertext bytes into `h` at the byte offset
   shown above;
6. mix the applicable X25519 DH output with `MixKey`;
7. mix the ML-KEM shared secret with a second `MixKey`;
8. continue with the classical encrypted static-key and responder-reply steps.

## Fixed Datagram Unit Header

The fixed datagram profile is the packet-first v1 target for PRP units. It
uses a fixed-size cleartext datagram header for both handshake and encrypted
packet units. The header does not carry a PRP magic value. The lower carrier
binding identifies that the next bytes are a PRP unit.

The fixed header length is 48 bytes. The encrypted or cleartext payload begins
at byte offset 48 for all envelope kinds. This value is a profile constant and
is not carried on the wire. Unused envelope-specific fields are encoded as
zero.

The first 8 bytes are the common prefix:

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 0 | 1 | `uint8` | datagram profile version, currently `1` |
| 1 | 1 | `uint8` | envelope kind |
| 2 | 1 | `uint8` | flags |
| 3 | 1 | `uint8` | handshake or relationship profile id |
| 4 | 2 | `uint16` | payload length |
| 6 | 2 | `uint16` | reserved, encoded as zero |

The fixed header context after the common prefix is:

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 8 | 1 | `uint8` | suite id for handshake envelopes; zero when selected by session state |
| 9 | 1 | `uint8` | handshake message type for handshake envelopes; zero otherwise |
| 10 | 6 | bytes | reserved, encoded as zero |
| 16 | 8 | bytes | session id for encrypted packet envelopes; zero otherwise |
| 24 | 8 | `uint64` | packet sequence for encrypted packet envelopes; zero otherwise |
| 32 | 8 | `uint64` | packet message id for fragmented message envelopes; zero otherwise |
| 40 | 2 | `uint16` | fragment index; zero otherwise |
| 42 | 2 | `uint16` | fragment count; zero otherwise |
| 44 | 2 | `uint16` | total plaintext message length when known; zero otherwise |
| 46 | 2 | `uint16` | reserved, encoded as zero |

`payload length` is 16-bit by design. A PRP datagram profile can address
payloads up to 65535 bytes while allowing a carrier or deployment profile to
impose a lower MTU. Larger objects are handled by fragmentation and reassembly
above the datagram payload boundary.

The fixed datagram enum space is assigned by `wire-registry-v1.md`. The v1
envelope kinds are:

| Value | Meaning |
| ---: | --- |
| `0x01` | handshake message |
| `0x02` | encrypted packet |
| `0x03` | encrypted packet feedback |
| `0x04` | encrypted status |

For encrypted packet, feedback, and status envelopes, the full 48-byte header
is AEAD associated data. The header is clear so low-level carriers, relays,
AF_XDP queues, kernel dataplanes, and FPGA dataplanes can size the unit, select
a session/key slot, apply replay policy, and enqueue the opaque body without
understanding service or application payload semantics.

For native fixed encrypted packet envelopes, `type`, `flags`, and
`plaintext_offset` are encrypted in the body, not clear fixed-header fields.
Service dispatch, fragment placement, and packet semantics are endpoint
decisions.

## Stream Handshake Envelope

In the stream profile, every Noise handshake message is carried as:

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 0 | 4 | `uint32` | handshake message length |
| 4 | length | bytes | Noise handshake message |

The length must be greater than zero and no larger than 1216 bytes. Classical
handshake messages remain at or below 96 bytes.

## Stream Record Header

Encrypted stream records are PRP frames. The frame header is authenticated as
AEAD associated data. Stream records use implicit transport nonces and require
reliable ordered delivery.

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 0 | 4 | bytes | ASCII `PRP1` |
| 4 | 2 | `uint16` | PRP version, currently `1` |
| 6 | 2 | `uint16` | record type |
| 8 | 2 | `uint16` | flags |
| 10 | 8 | `uint64` | request id |
| 18 | 4 | `uint32` | encrypted payload length |
| 22 | payload length | bytes | ciphertext plus AEAD tag |

`payload length` includes the 16-byte ChaCha20-Poly1305 tag. The decrypted
payload length is therefore `payload length - 16`.

Stream AEAD nonces are 12 bytes. The first four bytes are zero. The final
eight bytes are the local 64-bit send or receive nonce encoded little-endian.
The nonce is not carried on the wire; each side starts at zero and increments
its local counter after successful stream record encryption or authentication.

## Stream Record Types

Record type values are assigned by `wire-registry-v1.md`. The v1 record set is:

| Value | Name | Decrypted payload |
| ---: | --- | --- |
| `0x0001` | `PRP_RECORD_TYPE_DATA` | application bytes |
| `0x0002` | `PRP_RECORD_TYPE_CLOSE` | status payload |
| `0x0003` | `PRP_RECORD_TYPE_ROUTE` | route id |
| `0x0004` | `PRP_RECORD_TYPE_ERROR` | status payload |

## Routed Bootstrap Transcript

Routed `NK` and `IK` sessions use the same bootstrap transcript. The bootstrap
is a temporary stream-profile `Noise_NK` session whose responder static key is
the listener key from the acceptor relationship definition.

The bootstrap transcript on the wire is:

| Step | Sender | Bytes |
| ---: | --- | --- |
| 1 | initiator | stream handshake envelope containing bootstrap `Noise_NK` message 1 |
| 2 | acceptor | stream handshake envelope containing bootstrap `Noise_NK` message 2 |
| 3 | initiator | encrypted stream PRP frame with `type = PRP_RECORD_TYPE_ROUTE` |

The route record decrypted payload is exactly 16 bytes:

| Offset | Size | Meaning |
| ---: | ---: | --- |
| 0 | 16 | opaque local route id sent by the initiator |

The acceptor must authenticate and decrypt the route record before attempting
peer selection. If authentication fails, the record type is not
`PRP_RECORD_TYPE_ROUTE`, the decrypted payload length is not 16, or no
relationship matches the decrypted route id, the acceptor aborts before the
final Noise handshake.

Bootstrap keys are discarded immediately after route selection. The final
configured `Noise_NK` or `Noise_IK` handshake starts a separate transcript and
produces separate transport keys. The route record never authorizes application
traffic by itself.

## Status Payload

`PRP_RECORD_TYPE_CLOSE`, `PRP_RECORD_TYPE_ERROR`, and fixed datagram status
bodies use this encrypted payload:

| Offset | Size | Type | Meaning |
| ---: | ---: | --- | --- |
| 0 | 2 | `uint16` | status code |
| 2 | 2 | `uint16` | status detail |

Status code and detail values are assigned by `wire-registry-v1.md`. Status
payloads are exactly 4 bytes. Unknown status codes or details fail closed.
Human-readable reason strings are local diagnostics and are not carried on the
wire.

## Flow Summary

Direct `NN` is first-contact anonymous encryption. It has no relationship
file, route id, or static keys. Both peers select the direct anonymous profile
and suite id by local policy, mix the transcript context, complete `NN`, and
then exchange encrypted PRP units.

Routed `NK` starts with the routed bootstrap transcript, sends encrypted route
id, selects the peer definition, runs final configured `Noise_NK`, and only
then accepts encrypted application or service data.

Routed `IK` follows the same bootstrap and selection step, then runs final
mutually authenticated `Noise_IK` using pre-provisioned peer public keys.

## Limits

| Limit | Value |
| --- | ---: |
| Transcript context length | 3 bytes |
| Fixed datagram header length | 48 bytes |
| Fixed datagram max payload length | 65535 bytes |
| Stream record header length | 22 bytes |
| AEAD tag length | 16 bytes |
| Route id length | 16 bytes |
| Max stream encrypted payload | 16 MiB |
| Max classical handshake message | 96 bytes |
| Max hybrid handshake message | 1216 bytes |
| Status payload length | 4 bytes |
