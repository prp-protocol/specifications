# PRP v1 Wire Registry

This registry is the normative v1 numeric identifier registry for the current
`libprp` implementation. Public headers must match these values, and
`tests/wire_registry_test.py` fails the build if they drift. Long names are
diagnostic aliases unless a profile explicitly says otherwise. Unknown values
fail closed for the current v1 profiles.

## Transcript Profiles

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `0x01` | `PRP_PROFILE_DIRECT_ANONYMOUS` | Direct first-contact `NN`. |
| `0x02` | `PRP_PROFILE_ROUTED_BOOTSTRAP` | Temporary routed bootstrap session. |
| `0x03` | `PRP_PROFILE_CONFIGURED_RELATIONSHIP` | Final relationship-selected session. |

## Cryptographic Suites

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `0x00` | `PRP_SUITE_NONE_DEBUG` | Local diagnostic no-crypto mode only; rejected by default unless built with `PRP_ALLOW_NONE_DEBUG=1`; never production. |
| `0x01` | `PRP_SUITE_NN_X25519_CHACHAPOLY_BLAKE3_V1` | Classical `NN`. |
| `0x02` | `PRP_SUITE_NK_X25519_CHACHAPOLY_BLAKE3_V1` | Classical `NK`. |
| `0x03` | `PRP_SUITE_IK_X25519_CHACHAPOLY_BLAKE3_V1` | Classical `IK`. |
| `0x11` | `PRP_SUITE_NN_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` | Hybrid `NN`. |
| `0x12` | `PRP_SUITE_NK_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` | Hybrid `NK`. |
| `0x13` | `PRP_SUITE_IK_X25519_MLKEM768_CHACHAPOLY_BLAKE3_V1` | Hybrid `IK`. |

## Stream Record Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_RECORD_TYPE_DATA` | Stream application bytes. |
| `2` | `PRP_RECORD_TYPE_CLOSE` | Encrypted close status payload. |
| `3` | `PRP_RECORD_TYPE_ROUTE` | Encrypted route id during routed bootstrap. |
| `4` | `PRP_RECORD_TYPE_ERROR` | Encrypted error status payload. |

## Packet Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_PACKET_TYPE_DATA` | Unreliable application message fragment. |
| `2` | `PRP_PACKET_TYPE_ACK` | Optional positive packet feedback. |
| `3` | `PRP_PACKET_TYPE_NACK` | Optional missing-fragment feedback. |
| `4` | `PRP_PACKET_TYPE_STATUS` | Packet status payload. |
| `5` | `PRP_PACKET_TYPE_STREAM` | Reliable stream adapter payload. |

Packet type values `0` and `65535` are reserved. The v1 known flag mask is
`0x001f`.

## Known Flag Masks

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `0x001f` | `PRP_PACKET_FLAGS_KNOWN` | Packet-profile flags valid in v1. |
| `0x0000` | `PRP_MUX_FLAGS_KNOWN` | Mux v1 has no defined flags. |
| `0x0000` | `PRP_RELIABLE_FLAGS_KNOWN` | Reliable v1 has no defined flags. |
| `0x0000` | `PRP_ROUTING_FLAGS_KNOWN` | Routing v1 has no defined flags. |

## Datagram Handshake Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_DATAGRAM_HANDSHAKE_TYPE_MESSAGE1` | Initiator-to-responder Noise message. |
| `2` | `PRP_DATAGRAM_HANDSHAKE_TYPE_MESSAGE2` | Responder-to-initiator Noise message. |

## Fixed Datagram Candidate Envelopes

The fixed datagram profile is a candidate migration profile for carriers and
dataplanes that prefer a constant 48-byte low-level header. It is not the
current v1 `PRPH`/`PRPP` wire format. Classical and hybrid suites still use
the cryptographic suite ids registered above.

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_DATAGRAM_FIXED_ENVELOPE_HANDSHAKE` | Fixed-envelope Noise handshake payload. |
| `2` | `PRP_DATAGRAM_FIXED_ENVELOPE_PACKET` | Fixed-envelope encrypted packet payload. |
| `3` | `PRP_DATAGRAM_FIXED_ENVELOPE_PACKET_FEEDBACK` | Native encrypted packet feedback envelope. |
| `4` | `PRP_DATAGRAM_FIXED_ENVELOPE_STATUS` | Native encrypted status envelope. |

The fixed profile uses version `1`, fixed header length `48`, maximum payload
length `65535`, and currently defines no flags. Fixed session helpers select
the same one-byte `suite_id` values as the active v1 cryptographic registry:
`0x01`..`0x03` for classical `NN`/`NK`/`IK` and `0x11`..`0x13` for the
corresponding ML-KEM-768 hybrid profiles.

Native fixed feedback uses the fixed header as AEAD associated data and does
not carry an inner `PRPP` packet. `PACKET_FEEDBACK` with empty encrypted
plaintext is an ACK for `message_id`; `PACKET_FEEDBACK` with a 4-byte encrypted
plaintext is a NACK whose plaintext is the missing `fragment_index` in network
byte order. `STATUS` carries a 4-byte encrypted plaintext: 16-bit status code
followed by 16-bit status detail.

Native fixed `PACKET` also uses the fixed header as AEAD associated data and
does not carry an inner `PRPP` packet. Its encrypted plaintext starts with
`uint16 type`, `uint16 flags`, and `uint32 plaintext_offset`, followed by the
application packet plaintext. The fixed header carries session id, sequence,
message id, fragment index/count, and total plaintext length.

## Mux Frame Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_MUX_FRAME_DATA` | Service data frame. |
| `2` | `PRP_MUX_FRAME_OPEN` | Service open frame. |
| `3` | `PRP_MUX_FRAME_CLOSE` | Service close frame. |
| `4` | `PRP_MUX_FRAME_RESET` | Service reset frame. |

Mux v1 has no defined flags; the known flag mask is `0x0000`.

## Mux Reset Codes

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_MUX_RESET_SERVICE_DENIED` | Service is not authorized. |
| `2` | `PRP_MUX_RESET_UNKNOWN_SERVICE` | Service id is unknown. |
| `3` | `PRP_MUX_RESET_PROTOCOL` | Service mux protocol error. |

## Reliable Frame Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_RELIABLE_FRAME_DATA` | Reliable stream data. |
| `2` | `PRP_RELIABLE_FRAME_ACK` | Cumulative/SACK feedback. |
| `3` | `PRP_RELIABLE_FRAME_NACK` | Missing range feedback. |
| `4` | `PRP_RELIABLE_FRAME_OPEN` | Stream open. |
| `5` | `PRP_RELIABLE_FRAME_FIN` | Stream finish. |
| `6` | `PRP_RELIABLE_FRAME_RESET` | Stream reset. |
| `7` | `PRP_RELIABLE_FRAME_WINDOW` | Flow-control window. |

Reliable v1 has no defined flags; the known flag mask is `0x0000`.

## Routing Frame Types

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_ROUTING_FRAME_DATA` | Opaque routed body with label rewrite. |

Routing v1 has no defined flags; the known flag mask is `0x0000`.

## Routing Shaping Modes

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `0` | `PRP_ROUTING_PADDING_NONE` | No padding. |
| `1` | `PRP_ROUTING_PADDING_FIXED` | Fixed-size padding cell. |
| `2` | `PRP_ROUTING_PADDING_RANGE` | Range-based padding. |
| `0` | `PRP_ROUTING_EMISSION_IMMEDIATE` | Immediate emission. |
| `1` | `PRP_ROUTING_EMISSION_WINDOWED` | Windowed emission. |

## Bootstrap Provider Frames

| Value | Symbol | Meaning |
| ---: | --- | --- |
| `1` | `PRP_BOOTSTRAP_PROVIDER_FRAME_ATTEMPT` | Provider-local transport attempt. |
| `2` | `PRP_BOOTSTRAP_PROVIDER_FRAME_NOT_TRANSPORTED` | Fast denial: provider does not transport. |

## Reserved Service Names

| Service id | Name | Meaning |
| ---: | --- | --- |
| `1001` | `prp.telemetry` | Telemetry payloads for examples/tools. |
| `2001` | `prp.control` | Control payloads for examples/tools. |
| `3001` | `prp.routing.egress` | Routing egress envelope over adjacent PRP sessions. |

Service ids are dispatch labels inside an established relationship. They are
not global capabilities, identities, or carrier selectors.
