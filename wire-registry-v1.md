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

## Dataplane Registry

Dataplane ids are local control-plane selectors for accelerated targets, not
base wire bytes. They are listed here so software, AF_XDP, kernel, and FPGA
targets converge on one vocabulary.

Backends:

| Value | Symbol |
| ---: | --- |
| `0` | `PRP_DATAPLANE_BACKEND_SOFTWARE_SYNC` |
| `1` | `PRP_DATAPLANE_BACKEND_SOFTWARE_RING` |
| `2` | `PRP_DATAPLANE_BACKEND_AF_XDP` |
| `3` | `PRP_DATAPLANE_BACKEND_KERNEL` |
| `4` | `PRP_DATAPLANE_BACKEND_FPGA` |

Domains:

| Value | Symbol |
| ---: | --- |
| `0` | `PRP_DATAPLANE_DOMAIN_PACKET` |
| `1` | `PRP_DATAPLANE_DOMAIN_ROUTING` |
| `2` | `PRP_DATAPLANE_DOMAIN_MUX` |
| `3` | `PRP_DATAPLANE_DOMAIN_RELIABLE` |
| `4` | `PRP_DATAPLANE_DOMAIN_ROUTE_CONTROL` |
| `5` | `PRP_DATAPLANE_DOMAIN_COVER` |

Operations:

| Value | Symbol |
| ---: | --- |
| `0` | `PRP_DATAPLANE_OP_DECRYPT` |
| `1` | `PRP_DATAPLANE_OP_ENCRYPT` |

Drop reasons:

| Value | Symbol |
| ---: | --- |
| `0` | `PRP_DATAPLANE_DROP_NONE` |
| `1` | `PRP_DATAPLANE_DROP_MALFORMED` |
| `2` | `PRP_DATAPLANE_DROP_DECRYPT_FAILED` |
| `3` | `PRP_DATAPLANE_DROP_REPLAY` |
| `4` | `PRP_DATAPLANE_DROP_UNKNOWN_SESSION` |
| `5` | `PRP_DATAPLANE_DROP_UNKNOWN_ROUTE` |
| `6` | `PRP_DATAPLANE_DROP_HOP_LIMIT` |
| `7` | `PRP_DATAPLANE_DROP_EGRESS_UNAVAILABLE` |
| `8` | `PRP_DATAPLANE_DROP_QUEUE_FULL` |
| `9` | `PRP_DATAPLANE_DROP_KEY_SLOT_MISS` |
| `10` | `PRP_DATAPLANE_DROP_KEY_GENERATION_MISMATCH` |
| `11` | `PRP_DATAPLANE_DROP_UNSUPPORTED_LIMIT` |
| `12` | `PRP_DATAPLANE_DROP_BACKEND_RESET` |
