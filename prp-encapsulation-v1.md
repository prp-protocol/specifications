# PRP Encapsulation Profiles v1

Status: Draft

This document defines carrier encapsulation profiles below PRP. Encapsulation
profiles exist only to recover complete PRP units when a carrier does not
natively preserve PRP unit boundaries.

A PRP unit is either:

- a handshake unit;
- an encrypted packet unit.

The base PRP protocol consumes complete PRP units. It does not define
byte-stream delimitation, carrier fragmentation, media checksums, physical
retransmission, or resynchronization markers.

## Scope

An encapsulation profile may define:

- local length delimitation;
- carrier-local chunk or fragment numbering;
- carrier-local checksums or integrity checks used only to reconstruct bytes;
- carrier-local duplicate suppression before reassembly;
- carrier-local reassembly timeout and buffer limits;
- carrier-local error handling for incomplete units.

An encapsulation profile must not define:

- PRP relationship semantics;
- peer identity;
- route identity;
- service authorization;
- packet replay rules;
- cryptographic suite negotiation;
- application payload semantics.

Encapsulation headers are carrier metadata. They are not peer identity,
relationship material, route labels, service labels, replay state, or
application object descriptors.

## Registry Model

Encapsulation profiles are identified by canonical names, not by mandatory
wire-visible numeric ids.

Numeric ids should be assigned only when a concrete carrier binding must carry
an encapsulation selector in bytes visible to another implementation. A carrier
that already selects the encapsulation profile out of band, such as by port,
device binding, file format, queue type, characteristic, or local
configuration, does not need a PRP-visible encapsulation id.

Current canonical names:

| Name | Meaning |
| --- | --- |
| `datagram-unit` | One carrier datagram or message carries exactly one complete PRP unit. |
| `length-prefixed-stream` | A byte stream carries PRP units delimited by a local length prefix. |
| `fragmented-carrier` | A small-MTU carrier fragments and reassembles exactly one PRP unit. |
| `packet-file` | A file or indexed chunk set carries one complete PRP unit. |
| `ipc-message` | An IPC message or queue item carries one complete PRP unit. |

## Carrier Binding Requirements

A concrete carrier binding must declare:

- how the carrier selects PRP traffic before PRP decoding starts;
- whether the carrier preserves PRP unit boundaries natively;
- which encapsulation profile is used when boundaries are not preserved;
- the maximum reconstructed PRP unit length accepted locally;
- timeout and memory limits for incomplete carrier-local reassembly;
- whether carrier-local loss, duplication, or reordering is possible.

These declarations are carrier behavior, not PRP relationship semantics. They
must not change peer authentication, route labels, service authorization,
packet replay windows, cryptographic suites, or application payload formats.

## Carrier Binding Record

Carrier bindings should be documented with a small deterministic record. The
record is a specification device, not a PRP header:

```text
binding_name
traffic_selector
unit_boundary
encapsulation_profile
maximum_reconstructed_unit
loss_duplication_reordering_model
reassembly_timeout
local_error_behavior
```

`traffic_selector` names the lower-layer mechanism that identifies PRP traffic
before PRP decoding starts. Examples include an EtherType, IP protocol number,
UDP port, BLE characteristic, IPC endpoint, queue class, file record type, or a
locally configured adapter binding. It is not a peer id, route id, service id,
suite negotiation field, or application payload type.

`unit_boundary` says whether the carrier already preserves one complete PRP
unit or whether an encapsulation profile reconstructs it. The PRP decoder
receives only the completed unit.

`encapsulation_profile` must be one of the canonical names in this document
unless a future version of this document registers a new profile. Carrier
bindings may specialize local limits and timeout policy, but they must not
change the profile's relationship, identity, route, replay, suite, service, or
payload boundaries.

## `datagram-unit`

`datagram-unit` is the direct mapping for carriers that naturally preserve
message boundaries.

Semantics:

- one carrier datagram or message contains one complete PRP unit;
- the carrier binding identifies the content as PRP;
- no encapsulation bytes are required by this profile;
- carrier loss, duplication, or reordering is handled by PRP packet semantics
  or by layers above PRP.

Examples include UDP datagrams, raw IP/PRP payloads, Unix datagram sockets,
message queues, and AF_XDP descriptors when the descriptor spans one complete
PRP unit.

## `length-prefixed-stream`

`length-prefixed-stream` is for byte streams that do not preserve PRP unit
boundaries.

Encoding:

```text
uint32 unit_len
uint8  unit[unit_len]
```

Rules:

- `unit_len` is unsigned and encoded in network byte order;
- `unit_len` is the exact number of bytes in the following PRP unit;
- receivers must buffer until exactly `unit_len` bytes are available;
- receivers must reject zero-length units unless a concrete carrier binding
  explicitly reserves them for a local keepalive outside PRP;
- receivers must reject `unit_len` values above the local configured maximum;
- after reconstruction, only `unit` is passed to the PRP decoder.

The length prefix is not AEAD associated data for the PRP packet. It is only a
local carrier delimiter. PRP authentication begins at the reconstructed unit.

## `fragmented-carrier`

`fragmented-carrier` is for media whose physical message size is smaller than
the PRP unit size.

Minimal metadata:

```text
fragment_set_id
fragment_index
fragment_count
unit_len
fragment_offset
fragment_len
fragment_bytes
```

Rules:

- all fragments in one set reconstruct exactly one PRP unit;
- `fragment_index` is zero-based;
- `fragment_count` must be greater than zero;
- `unit_len` is the complete PRP unit length after reassembly;
- `fragment_offset` is the zero-based byte offset inside the reconstructed PRP
  unit;
- `fragment_offset + fragment_len` must not exceed `unit_len`;
- duplicate fragments may be ignored before PRP authentication;
- conflicting duplicate fragments must fail the reassembly set;
- incomplete sets expire according to carrier-local policy;
- after reassembly, only the complete PRP unit is passed to the PRP decoder.

The fragment set id is a carrier-local reassembly key. It must not be a peer
id, route id, service id, packet sequence, or application object id.

## `packet-file`

`packet-file` is for store-and-forward media where complete PRP units are
written to files or indexed chunk sets.

Valid shapes:

- one file contains exactly one complete PRP unit;
- one manifest entry names a bounded set of chunks that reconstruct exactly one
  complete PRP unit.

Rules:

- file paths, chunk names, and manifest indexes are local carrier metadata;
- file names must not be interpreted as peer identity, route identity, service
  authorization, replay state, or application payload semantics;
- the reconstructed PRP unit must be authenticated by PRP before plaintext is
  exposed;
- incomplete or inconsistent chunk sets fail closed.

## `ipc-message`

`ipc-message` is for local IPC mechanisms that preserve message boundaries.

Semantics:

- one IPC message carries exactly one complete PRP unit;
- queue names, socket paths, VM socket endpoints, shared-memory indexes, or
  guest/host handles are local carrier locators;
- queue backpressure is carrier policy;
- IPC metadata must not select PRP peers, routes, services, suites, replay
  windows, or payload formats.

If the IPC mechanism is stream-like rather than message-like, use
`length-prefixed-stream`.

## Failure Behavior

Encapsulation failures are carrier failures. They happen before PRP
authentication.

Receivers should fail closed for:

- malformed length prefixes;
- unit lengths above local limits;
- fragment indexes outside `fragment_count`;
- fragment count or total length mismatch;
- conflicting duplicate fragments;
- expired incomplete reassembly sets;
- trailing bytes in a supposedly complete unit, unless the concrete carrier
  binding explicitly defines a stream cursor that consumes only one unit at a
  time.

Encapsulation failure must not produce PRP plaintext, route diagnostics,
relationship diagnostics, or peer-visible fine-grained PRP errors. Tools may
log local carrier diagnostics for operators.

## Relationship To PRP

After an encapsulation profile reconstructs one complete PRP unit, PRP owns:

- profile and suite validation;
- Noise transcript binding;
- AEAD authentication;
- replay rules;
- route labels;
- service multiplexing;
- reliable adapter state;
- payload delivery.

Before that point, the carrier owns only the mechanics needed to recover bytes.
