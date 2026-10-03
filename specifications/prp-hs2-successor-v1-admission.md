# REF-001 — explicit N08 normative admission

Decision date: 2026-09-12. Request `ctl-1789235862-2525`.
**N08: ADMITTED, within the bilateral successor scope below.**
This is prp-spec's explicit final normative disposition, not inference from
passing tests. No essential user choice or concrete NC01–NC05 blocker remains
for the approved implementation-ready cutoff. It is NOT integrated HS2 delivery,
M2/M3 PASS, release, external registry registration or implementation dispatch.

## 1. Exact authority and precedence

Admitted contracts DELIVERY is **`607bc6e60c3c4057b62d0cc46fc924e9f47e7878`**.
The decision OID is the prp-spec commit introducing THIS file; the return to
root supplies its full OID. A later branch tip is not a substitute for either.
Bind that decision OID once into the existing contracts handoff as documentary
reconciliation, without changing the admitted source or creating another grammar.

This admission supersedes candidate/NOT ADMITTED status labels at that exact
delivery, but does not relabel historical tests or implement their obligations.
The following exact paths in that contracts commit are the admitted set:

1. `proposals/ref-001-final-v1.md`: sole controlling grammar and semantic bindings.
2. `proposals/ref-001-ownership-v1.md` and `proposals/ref-001-close-v1.md`:
   role-specific operation/ownership barriers and precise ordered close refinement.
3. `proposals/ref-001-safety-v1.md` sections1–3,6–7: safety policy, immutable
   limits, complete assembly/accounting and corrected sample/epoch barriers.
4. `proposals/ref-001-admission-package-v1.md`: exact evidence/precedence index
   and closed-scope provider-first handoff, not another grammar.

CODEC/crypto/safety/close companions and executable sources are evidence, not
authority to replace verification with their dictionaries, booleans or fixture
defaults. All executable helpers/corpora remain pinned contracts
`4bed8b1db90262efff4265fd70569894f2e6792d` and are unchanged at DELIVERY.

## 2. Explicit admitted technical dispositions

| Subject | Admitted scope/assignment |
| --- | --- |
| Recognition | Exact ASCII `PRPREF01` (no NUL), new-format Frame kinds1 declaration,2 M1,3 M2,4 protected record,5 fragment; outer11/body<=65524/complete<=65535. Unknown/truncated/trailing/reserved bytes refuse; no heuristic legacy or earlier candidate fallback. |
| Reference | Header low2 classes0 IDENTITY,1 ALIAS,2 MULTICAST,3 GROUPCAST; high6 zero; full u16 suite and derived identifier length. This bilateral profile admits ONLY0/1 before suite-width lookup. Classes2/3 retain separate future semantics, not implementation support here. Representation is a proven role within IDENTITY, not a fifth reference class. |
| Packed roles | Count bits0..2,1..6; INIT role bit3, bits4..7 zero. M2 RESP role bit3 and exact INIT echo bit4, bits5..7 zero. These belong to the new format, never legacy flags or reserved Ref bits. |
| Purpose/signature | F=`PRP-REF001-1`, Q=B16(F)\|\|B16(purpose)\|\|B32(X), C=F; local purposes `declaration`, `M1`, `M2`. X excludes own proof and enclosing prefix. Suite1 Ed25519(BLAKE3(Q));2 Ed25519ctx(Q,C),phflag0;3/4 pure MLDSA65/87(Q,C);5 ordered Ed25519(BLAKE3(Q)) AND MLDSA87(Q,C). Full key/ID equality mandatory. |
| Commitment | SHA512(B16(F)\|\|B16(p)\|\|B32(X)),64 bytes, p=`query`,`result`,`vectors`. Binding under hash assumptions, not origin/membership authentication or a claim of512-bit collision/automatic quantum strength. |
| Negotiation | Five identity suites/six protected suites; same identity universe for both endpoints and each principal/representative. Independent per-offer shares; ordered unique acceptable lists; first INIT preference accepted by RESP wins. No common suite means refusal/cleanup, not fallback. NONE excluded from successor; legacy unchanged. |
| Flights | Full signed M1 committed by selected H in M2; T hashes length-delimited format and ordered full signed M1/M2. Freeze identities, role/echo, selection/pins, context/generation/nonces and current policies. No early application authority. |
| Ownership/ALIAS | INIT retains authenticated-source Q/R plus frozen row/index/S; RESP validates public signed M1/S against explicit local expectations without private R or query keys. Exact requested suite and selected logical principal bind to HS2, with valid same-suite delegation and fresh representative proof. Membership legitimacy/permissions remain service concerns. |
| Confirmation/records | Direction0 I→R,1 R→I; sequence u64, nonce directional prefix4\|\|sequence8; exact FINAL AAD and transcript binding. Types0 CONTROL,1 DATA,2 CLOSE,3 CLOSE_ACK. Only eligible CONTROL/DATA confirm before payload/replay effects; CLOSE/ACK never confirm. |
| Close/recovery | One outstanding local close per direction, exact session/opposite direction correlation, exact-next-sequence admission and bounded gap retention; no ACK-of-ACK. Stop bars new DATA, admitted work drains. Passive completion proves local completion only. Current policy/time/epoch required; original-proof closing revalidation preserves history; CLOSED terminal. |
| Fragments | Unauthenticated correlation envelope with37-byte overhead, nonempty bounded chunks, exact coverage and full internal verification before OWN admission. Effective capacity after carrier wrappers; complete65535/bare65532/legacy16384 distinct. Exact duplicates idempotent, other overlap/mixed totals refuse; attempt discontinuity aborts pending assemblies while preserving caller-held output charges. |

The six admitted agreement/record mappings and exact EP/KP/KC/H widths are those
in FINAL §3:0101 X25519/ChaCha;0102 X25519/AES256GCM;0201 X25519+MLKEM768/ChaCha;
0202 X25519+MLKEM1024/ChaCha;0301 MLKEM768/AES256GCM;0302 MLKEM1024/AES256GCM.
0101/0102/0201/0202 use BLAKE3-32 T and derive-key context
`PRP-REF001-1 traffic`, input B16(Z)||info,36-byte output.0301/0302 use
SHA256/SHA384 T and HKDF Extract(salt=T,IKM=Z),Expand(info,36).
Z and info retain exact length/direction/suite framing in the controller;
key first32/prefix last4. Failed/zero agreement is not a usable secret.

The following conservative crypto-use ceilings are expressly admitted, NOT
fixture resource quotas or measured deployment profiles:

| Cipher | Seal records | Open records | Bytes per operation kind | Auth16 blocks per operation kind | Failed/unknown opens |
| --- | ---: | ---: | ---: | ---: | ---: |
| AES256GCM | 2^16 | 2^20 | 2^28 | 2^24 | 1024 |
| ChaCha20Poly1305 | 2^20 | 2^20 | 2^34 | 2^30 | 1024 |

Counting is exactly SAFE: n+AAD bytes, ceil(n/16)+ceil(AAD/16)+1 auth blocks,
full128-bit tags, independent seal/open columns and all failure attempts.
Immutable stricter local limits are allowed; pre-crypto checked reservation and
unique sequence burn apply. Shared key aliases cannot reset budgets; unknown
seal outcome retires key; restart with lost continuity discards live authority.
No persistent live resumption, replica nonce sharing or rekey fallback is admitted.
Numeric native memory/rate/queue profile requires later measurement/approval;
fixture1MiB/32slots/two retries/deadline100 are NOT assigned by this decision.

## 3. Evidence and final compatibility review

The exact NC01–NC05 review OIDs and frozen hashes in package §2 are incorporated
with their limits. In particular structural CODEC evidence, finite final crypto,
corrected SAFE/OWN models and ten encrypted CLOSE schedules are not all-product,
native ownership or concurrent implementation evidence. Their acceptance-status
booleans remain false for such claims; this N08 disposition is separate.

This turn inspected the full five-file DELIVERY delta: only Markdown changed.
`git diff --exit-code 4bed8b1... 607bc6e... -- tests` passed; the full name-only
diff lists no corpus/helper change. Rehashed all six corpora at DELIVERY;
all match package §2. `git show --check 607bc6e...` passed. No test campaign was
repeated. Prior executions remain spec/owner-attributed as recorded, not reruns.

Scoped committed spec wire/discovery search finds no admitted `PRPREF01` or
`PRP-REF001-1` collision. Existing wire-v4 HS2 prefix is version:u8=04;
new complete recognition starts50 (ASCII P) and differs in full framing. This
is not a global collision proof or permission to put new kinds in a legacy
opcode table. Recognition must occur in an explicitly supported successor path;
failure cannot reinterpret bytes as v4/earlier proposals. Existing suite IDs
retain registry meanings, while new-format purpose adaptation does not mutate
legacy wrappers. DNSrev9 remains independently scoped and unmodified.

Role-specific commit wording and pre-crypto versus final admission barriers are
consistent: private source evidence is only INIT's prerequisite; RESP uses public
proof and explicit expectations. A later fault still gates admission even after
valid AEAD; usage accounting burns without replay/payload effects. Diagnostics
refine typed local results, not new peer-controlled error opcodes. No new user
semantic choice is hidden in these technical assignments.

## 4. Closed-scope handoff and later acceptance

Package §4 is accepted as the SAME implementation handoff, bound to DELIVERY
and this decision's commit OID. Root/contracts must insert that exact decision
OID once; no new byte set or competing contract is needed. This bookkeeping
does not authorize dispatch by spec. NC01–NC06 normative dispositions are
satisfied for the approved cutoff; external coordination/publication remains
separate from local normative delivery.

Dependency order is provider libprp → provider review → runtime via the existing
graph → contracts integrated evidence → independent spec M2/M3 review. Provider
owns keys/entropy/signatures/KEX/records/assembly/installed evidence; runtime
owns carrier FDs/entrants/queues and verified-data exposure. Exact primitive pin
f28a26c79073e44a3feb4614e72581e303fe2904 and assessment/P1 pins remain as listed
in the package; provenance-only31d363 is not a new crypto baseline.

Future finite manifest:5 universes×6 selected protections×4 mode pairs×2 request
classes=240 coordinates; two fresh contexts each=480 actual sessions. This
derives from approved support plus current same-universe semantics, superseding
historical5400/10800 and deterministic-map counts, not silently narrowing support.
Every coordinate uses actual client/service APIs in two independent native
processes over AF_UNIX SOCK_SEQPACKET, with verified data and orderly close.
CONTROL-first (RESP delivered first) and eligible DATA-first (INIT first) remain
distinct schedules. Named negative-equivalence and runtime U3-1..8 branches in
the package are mandatory; no parser-only substitute or all625 inference.

Native key custody, trusted interval/continuity acquisition, dual budgets,
heap/workspace/stack/FD accounting, reachable concurrency, cancellation/join,
erasure and measured numeric profile remain essential M2/M3 obligations, not
optional hardening or delivered implementation. Shared clock faults require
independent original-proof recovery per owner. No invented external time service,
hardware/VM/browser/kernel deployment prerequisite or service-membership issuer.

Local normative milestone M1 is admitted; integrated M2/M3 are NOT complete.
No provider/runtime implementation, activation, issue, grant/install, ABI/legacy/
DNS mutation, release/push/IANA action or remote-publication assertion accompanies
this decision. The original owner request has no recovered correlated final
response: this review relies on exact committed source, not an invented return.
