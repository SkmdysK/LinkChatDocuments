# LinkChat Documents

[中文](README.zh-CN.md) | **English**

LinkChat Documents is the language-independent research companion for
LinkChat v1.0. It collects the protocol specification, cryptographic profile,
public conformance vectors, design rationale, and the rendered research paper
needed to study, review, or independently implement the protocol without
depending on a specific Rust API or memory layout.

It is written for cryptographers, protocol designers, formal-methods
researchers, implementers, and reviewers. The executable Rust kernel and Lean
artifacts live in the companion
[`LinkChat`](https://github.com/SkmdysK/LinkChat) repository.

## Research Question

LinkChat studies a strictly alternating end-to-end protocol where a peer can
submit an authenticated input but cannot choose the honest endpoint's next
private Receiver State. Each endpoint owns a private Receiver Package and
publishes a cryptographically bound public projection. Valid inputs advance a
single application turn; malformed, replayed, stale, future-turn, or
contextually invalid inputs do not advance state.

The documents treat the following as distinct, auditable boundaries:

- endpoint-local private Receiver State and public Receiver Packages;
- strict Alice/Bob application turns and replay behavior;
- identity, session, cipher-suite, token, turn, generation, and package-hash
  binding;
- canonical encoding, re-encoding checks, and fixed 4096-byte Envelopes;
- X25519 plus ML-KEM-768 hybrid establishment;
- domain-separated HKDF-SHA-256 and authenticated-header/message contexts;
- prepare, durable commit, recovery, and truthful rollback-capability claims;
- transport and mailbox behavior that must not advance application state;
- formal correspondence, bounded conformance, and security-game obligations.

## Reading Paths

### Protocol and implementation review

1. [`SUMMARY.md`](SUMMARY.md)
2. [`spec/00-overview.md`](spec/00-overview.md)
3. [`spec/04-cryptographic-profile.md`](spec/04-cryptographic-profile.md)
4. [`spec/05-wire-format.md`](spec/05-wire-format.md)
5. [`spec/06-state-machine.md`](spec/06-state-machine.md)
6. [`spec/08-storage-and-recovery.md`](spec/08-storage-and-recovery.md)
7. [`spec/11-conformance.md`](spec/11-conformance.md)

### Research and formal-methods review

1. [`spec/09-security-model.md`](spec/09-security-model.md)
2. [`spec/10-formal-verification.md`](spec/10-formal-verification.md)
3. [`paper/LinkChat-Protocol-Research.pdf`](paper/LinkChat-Protocol-Research.pdf)
4. The `formalization/` directory in the companion implementation repository

### Independent implementation

Read the terminology, cryptographic profile, wire format, state machine,
storage/recovery rules, and conformance requirements in that order. Do not
derive a wire format from host-language object layouts or a single vector.
Use the public vectors as cross-checks after implementing the normative
requirements.

## Cryptographic Profile

| Boundary | Standard construction |
| --- | --- |
| Identity authentication | Ed25519 |
| Classical key agreement/KEM wrapper | X25519 |
| Post-quantum key encapsulation | ML-KEM-768 |
| Key derivation | Domain-separated HKDF-SHA-256 |
| Package/transcript digest | SHA-256 over canonical encoding |
| Header and payload protection | ChaCha20-Poly1305 |
| Optional MAC boundary | HMAC-SHA-256 |
| Fresh randomness | OS CSPRNG |

The specification defines composition, binding, encoding, and failure
semantics. It does not replace the primitive-level analysis, implementation
review, or parameter guidance for these standard constructions.

## Repository Map

| Path | Purpose |
| --- | --- |
| `SUMMARY.md` | Research overview and reading guide |
| `spec/` | Normative and explanatory protocol chapters |
| `vectors/` | Public JSON conformance vectors and manifest |
| `paper/LinkChat-Protocol-Research.tex` | Research paper source |
| `paper/LinkChat-Protocol-Research.pdf` | Rendered research paper |
| `CHANGELOG.md` | Documented changes to the research material |

The specification chapters cover terminology, architecture, identity/session
binding, cryptography, wire format, state transitions, transport/mailbox
contracts, storage/recovery, the security model, formal verification,
conformance, versioning, and an end-to-end example.

## Evidence and Claim Boundary

The project distinguishes four kinds of evidence:

1. **Normative requirements.** The specification defines required fields,
   encodings, context binding, state rules, storage ordering, and integration
   obligations.
2. **Machine-checked structural results.** The companion Lean model checks
   selected state-machine, package-rotation, non-interference, game-interface,
   and reduction-bookkeeping properties.
3. **Executable conformance evidence.** The Rust workspace supplies vectors,
   unit/property tests, fuzz smoke cases, crash matrices, and bounded
   differential checks.
4. **External assumptions.** Standard primitive security, operating-system
   randomness, secure erasure, side-channel behavior, rollback resistance, and
   deployment architecture remain external assumptions.

Neither finite test coverage nor the included symbolic/formal models alone
proves security against every probabilistic polynomial-time adversary in every
implementation or deployment. The documents deliberately state these limits
so an adopter can evaluate the protocol within the correct threat model.

## Scope and Non-Goals

This repository does not define or operate a public server, DHT, relay,
production mailbox service, account system, GUI, chat client, traffic-analysis
defense, or key-transparency infrastructure. Those are separate deployment and
integration problems and must not be inferred from the protocol core.

The documents describe a frozen research core, not an unconditional claim of
production readiness or complete cryptographic security.

## License

Released under the MIT License. Refer to the companion implementation
repository for executable-code licensing and contribution guidance.
