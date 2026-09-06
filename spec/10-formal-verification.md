# 10. Formal Verification and Trust Boundaries

## 10.1 Lean project

The Link Chat Lean project is maintained in the companion
`LinkChatFormalization` repository:

```text
LinkChat/formalization/
```

Relevant modules are:

```text
LinkChat.lean
SecurityGames.lean
RefinedProtocol.lean
StandardCrypto.lean
ComputationalGames.lean
ParameterizedConcreteSecurity.lean
ConcreteSECCReduction.lean
ConcreteSecurityReductions.lean
ImplementationSemantics.lean
TrustBoundaryAudit.lean
AuditHardening.lean
```

## 10.2 Structural theorems

Under their explicit model premises, the files check:

```text
strict alternation and unique Leader
invalid-input no-state-change
Replay, future-turn, and duplicate handling
atomic Receiver Package rotation
Package binding and state-injection resistance
bidirectional State Non-Interference
continuous adaptive attack traces
abstract KEM and key-derivation correctness
```

## 10.3 Computational game layer

The formalization also contains:

```text
finite and parameterized probability semantics
PPT resource certificates
CompleteAdversaryView
Future-State and Future-Message challenges
FS, PCS, Authentication, and KCI game interfaces
named primitive wrapper games
hybrid advantage addition
finite-term negligible closure
```

## 10.4 Audited concrete-instance endpoint

The recommended endpoints are:

```text
audited_final_standard_primitive_secc
audited_final_standard_primitive_secc_with_refinement
```

They require:

```text
one parameterized probability model
exact named wrapper-game identities
PPT witnesses
the declared adversary correspondence
complete-view protocol-game binding
Real = StateIsolated when claiming a zero state term
an explicit hybrid reduction bound
primitive negligible certificates
```

The result is a conditional theorem for a supplied concrete game instance and witnesses. It
is not an unconditional theorem that every external implementation is secure.

## 10.5 Certificate identity

A field named `x25519` is not sufficient to establish an X25519 certificate. The audit
boundary requires:

```text
certificate.game = designated X25519 wrapper game
certificate.model = reduction probability model
certificate adversary = declared PPT witness
certificate index = the same security parameter n
```

The same requirement applies to ML-KEM, HKDF, AEAD, and CSPRNG. Each hybrid hop must also
document its simulator and resource bound.

## 10.6 What Lean does not prove here

The current project does not derive the computational security of X25519, ML-KEM, HKDF,
ChaCha20-Poly1305, Ed25519, or an OS CSPRNG from their low-level algorithms and probability
distributions. Those properties enter as named external certificates or standard assumptions.

This is an explicit layering decision: protocol proof, game composition, and primitive proof
are separate artifacts.

## 10.7 Rust correspondence

```text
Lean step/refinedStep
    -> Rust pure transition and EndpointKernel

State Non-Interference
    -> fixed-fresh-schedule semantic binding and tests

rollback theorem
    -> storage transaction, recovery contract, and fault matrix

Future-Message Game
    -> research/verification harness, not a business API

KEM correctness
    -> CryptoProvider backend conformance, not a trait name alone
```

## 10.8 Recommended claim

> Under the specified parameterized probability model, PPT adversary witness, named wrapper
> game certificates, and game-hop reduction bound, the Link Chat protocol game satisfies the
> conditional Computational SECC theorem.

The following claim is not supported by this project:

> Lean independently proved the computational security of the external X25519, ML-KEM, or
> AEAD implementation.
