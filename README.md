# LinkChat Documents

## Cryptographic Protocol Specification, Test Vectors, and Research Paper

This repository contains the research documentation for Link Chat v1.0. It is
the normative and explanatory companion to the `LinkChat` repository, which
contains the Rust reference kernel and Lean formalization.

The repository is intended for cryptographers, protocol designers,
formal-methods researchers, implementers, and reviewers who need a stable
description of the protocol independent of any particular programming language.

## 中文简介

本仓库是 Link Chat v1.0 的密码学协议研究资料库，包含协议规范、canonical
wire 测试向量、实现一致性材料和研究论文。它与 `LinkChat` 仓库配套，后者包含
Rust 参考内核和 Lean 形式化模型。

本仓库的目标是让密码学研究者、协议设计者、形式化验证研究者、实现者和审阅
者能够在不依赖特定编程语言或内存布局的前提下，独立理解、复现和审查协议。

## Research Question

Link Chat studies a strictly alternating end-to-end protocol in which
peer-controlled input participates in authenticated message processing without
giving the peer authority to select the honest endpoint's next private Receiver
State.

The specification treats the following as separate, auditable objects:

- endpoint-local private Receiver State and public Receiver Packages;
- strict Alice/Bob Application Turns;
- identity, session, key-id, token, turn, and package-hash binding;
- canonical encoding and fixed-size 4096-byte Envelopes;
- X25519 plus ML-KEM-768 hybrid key establishment;
- domain-separated HKDF-SHA-256 derivation and AEAD context construction;
- prepare, durable commit, recovery, and rollback-capability declarations;
- transport and mailbox behavior that cannot advance application state;
- formal correspondence, conformance, and security-game obligations.

## 中文研究范围

协议研究重点包括：

1. **端点本地私有状态：**私有 Receiver Package 与公开 projection 必须绑定，
   作为一个原子逻辑单位生成和轮换。
2. **严格应用轮次：**Alice 与 Bob 严格交替；非法、重放、过去轮次、未来轮次
   或错误上下文输入不得推进协议状态。
3. **固定长度 Envelope：**Header 和 payload 使用规范化域分离、AAD 和固定
   4096-byte Envelope 进行认证保护。
4. **持久化事务：**Current、Pending、Committed、durability barrier、提交和
   恢复语义必须分开描述，不能把普通文件系统误称为具备回滚防护。
5. **分层边界：**网络、Mailbox、重试、去重和 ACK/SACK 属于外层集成，不得
   改变纯协议状态机的语义。

## Repository Layout

```text
SUMMARY.md                         research overview and reading guide
spec/00-overview.md                protocol scope and design goals
spec/01-terminology.md             normative terminology
spec/02-architecture.md            endpoint and integration architecture
spec/03-identity-session.md        identity and session binding
spec/04-cryptographic-profile.md   primitive profile and derivation boundaries
spec/05-wire-format.md             canonical wire contract
spec/06-state-machine.md           turns, replay, and state transitions
spec/07-transport-mailbox.md       outer transport and mailbox contracts
spec/08-storage-and-recovery.md    prepare, commit, durability, and recovery
spec/09-security-model.md          threat model and security claims
spec/10-formal-verification.md     Lean/Rust correspondence obligations
spec/11-conformance.md             independent implementation requirements
spec/12-versioning.md               version and compatibility rules
spec/13-end-to-end-example.md      complete protocol walkthrough
vectors/                            public canonical implementation vectors
paper/                              LaTeX source and rendered research paper
```

## Reading Order

For a short introduction, start with [`SUMMARY.md`](SUMMARY.md). For an
implementation, read the wire and state-machine contracts first:

1. [`spec/00-overview.md`](spec/00-overview.md)
2. [`spec/04-cryptographic-profile.md`](spec/04-cryptographic-profile.md)
3. [`spec/05-wire-format.md`](spec/05-wire-format.md)
4. [`spec/06-state-machine.md`](spec/06-state-machine.md)
5. [`spec/08-storage-and-recovery.md`](spec/08-storage-and-recovery.md)
6. [`spec/11-conformance.md`](spec/11-conformance.md)

The complete paper is available as
[`paper/LinkChat-Protocol-Research.pdf`](paper/LinkChat-Protocol-Research.pdf),
with its source in
[`paper/LinkChat-Protocol-Research.tex`](paper/LinkChat-Protocol-Research.tex).

## Cryptographic Profile

| Protocol boundary | Standard choice |
| --- | --- |
| Identity authentication | Ed25519 |
| Classical key agreement/KEM wrapper | X25519 |
| Post-quantum KEM | ML-KEM-768 |
| Domain-separated derivation | HKDF-SHA-256 |
| Package and transcript hash | SHA-256 over canonical encoding |
| Header and message authentication/confidentiality | ChaCha20-Poly1305 |
| Optional MAC boundary | HMAC-SHA-256 |
| Fresh key and token generation | OS CSPRNG |

The protocol specification defines how these primitives are composed and what
inputs are authenticated. It does not replace the security analyses of the
primitives themselves.

## Claim Boundary

The project distinguishes four layers of evidence:

- **Normative protocol requirements:** fields, encodings, contexts, state rules,
  and integration obligations specified here.
- **Machine-checked structural properties:** Lean proofs of state-machine,
  package-rotation, non-interference, game-interface, and reduction-bookkeeping
  statements.
- **Executable conformance evidence:** Rust tests, vectors, fuzz smoke checks,
  crash matrices, and bounded differential checks.
- **External assumptions:** computational security of standard primitives, OS
  randomness, secure erasure, filesystem rollback resistance, side-channel
  behavior, and deployment architecture.

The formalization and tests do not by themselves prove a complete computational
security theorem for every implementation or deployment. In particular,
bounded testing cannot replace an arbitrary-adversary proof, and symbolic Lean
encodings cannot replace a vetted cryptographic library or a concrete
probabilistic security reduction.

## Normative Language

`MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY` are normative terms. An
implementation must follow the specification rather than infer a wire format
from Rust memory layout or from a single test vector.

## Reproducibility

The companion `LinkChat` repository provides the executable Rust and Lean
artifacts. Public vectors in this repository are intentionally free of private
keys, long-lived credentials, personal identifiers, and machine-specific paths.
Generated compiler and LaTeX build products are excluded by `.gitignore` and
should remain outside version control.

## Scope and Non-Goals

This repository does not define or operate a public server, DHT deployment,
Relay network, production Mailbox service, account system, GUI, chat client, or
global traffic-analysis defense. Those are deployment and integration concerns
outside the protocol-kernel research artifact.

## Status and License

The documents describe a frozen research core and its implementation contracts;
they are not a claim of production readiness or unconditional security.

Released under the MIT License. See the companion implementation repository for
the executable license and contribution policy.
