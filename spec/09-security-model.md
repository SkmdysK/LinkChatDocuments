# 09. Security Model and Research Properties

## 9.1 Model position

This chapter defines the adversary, observations, and properties used by the protocol
research. It is not an unconditional promise for every deployment. A claim must be cited
with its adversary capabilities, honest-endpoint assumptions, random-source assumptions,
and storage assumptions.

## 9.2 Continuous compromise of one endpoint

For the Alice-compromised case, Alice's controller may choose or observe:

```text
Alice local state, plaintext, randomness, ciphertexts, Tokens
Alice application actions
network, Transport, Mailbox, and DHT actions
replay, delay, drop, modify, and injection behavior
```

Bob remains honest and his Receiver private state, fresh local source, and storage authority
are not directly exposed. The opposite direction is defined symmetrically.

## 9.3 CompleteAdversaryView

The formal view may include:

```text
controlled endpoint state, plaintexts, randomness, ciphertexts, and Tokens
network / Transport / Mailbox / DHT traces
honest endpoint public output
ACK/error and accept/reject observations
event time
logical message length
fixed Envelope length
state-update count
Receiver Package-update count
```

If an experiment exposes only a subset, it must call that object a projection rather than a
complete view.

## 9.4 Structural state isolation

For honest state `s`, local fresh input `rho`, and peer inputs `x1`, `x2`, the strong form is:

```text
accepted(s, x1) = accepted(s, x2)
    implies
next_secret(s, x1, rho) = next_secret(s, x2, rho)
```

An implementation or game binding may require equality of the complete observable
experiment, not only equality of the final private secret.

This property says that a message does not own the honest endpoint's next-state choice. It
does not by itself prove computational indistinguishability of keys.

## 9.5 Future-State challenge

Define two worlds:

```text
Real   uses the honest endpoint's future Receiver State
Random uses an independent random future-state value
```

The worlds must share the same initial state, adaptive strategy, trace schedule, timing and
length metadata, ACK/error interface, accept/reject interface, and challenge message inputs.
Only the challenge material derived from the future state may differ.

## 9.6 Future-Message challenge

The message-level challenge is:

```text
b <- {0, 1}
C* = Seal(K_b, nonce, AD, PayloadFrame(M_b))
```

The distinguisher receives a real-format challenge ciphertext and the experiment's declared
observations. The challenge retains the fixed Envelope length, Header structure, MessageAD,
and metadata. Exposing an internal State value alone is not a message-confidentiality game.

## 9.7 Probability semantics

An experiment must specify:

```text
security parameter lambda
sample space Omega_lambda
random variables
adversary strategy
runtime and query bounds
success predicate
success probability
advantage
```

The Lean finite-uniform model is an explicit checkable instance of these semantics. It does
not automatically represent an operating-system random source or every infinite sample
space.

## 9.8 Reduction shape

The intended composition is:

```text
Adv_SECC(A, lambda)
  <= Adv_state(A, lambda)
  + Adv_X25519(B_X, lambda)
  + Adv_MLKEM(B_PQ, lambda)
  + Adv_HKDF(B_KDF, lambda)
  + Adv_AEAD_conf(B_C, lambda)
  + Adv_AEAD_int(B_I, lambda)
  + Adv_CSPRNG(B_R, lambda)
```

When the Real and State-Isolated experiments are extensionally equal,
`Adv_state = 0`. Otherwise the nonzero state term remains in the bound.

## 9.9 Other games

### Forward secrecy

The adversary obtains a current state and is challenged on a past erased Message Key or
message. The proof must account separately for erasure, key derivation, AEAD, and randomness.

### Post-compromise security

The adversary obtains state at `t0`; the endpoint becomes honest at `t1`, generates a fresh
Package, and is challenged on a message after recovery. Recovery MUST NOT derive new private
state directly from peer input.

### Authentication

The adversary attempts cross-Session, cross-Turn, cross-KeyID, or cross-chain Package
forgeries.

### KCI

Alice-identity compromise and Bob-identity compromise are separate directions. The affected
Session must be invalidated and re-established through the identity procedure.

## 9.10 Explicit non-claims

v1.0 does not claim:

```text
confidentiality after both endpoints are fully compromised
protection for plaintext already present in a compromised endpoint
global traffic-analysis resistance
DHT / Relay availability
absence of external-library defects
absence of operating-system or hardware side channels
automatic rollback resistance from an ordinary filesystem
```

## 9.11 Trust-boundary table

| Object | Protocol requirement | Remaining boundary |
| --- | --- | --- |
| Turn and state transition | precisely specified | Lean structural theorem |
| Canonical codec | precisely specified | implementation conformance |
| KEM/KDF/AEAD calls | precisely specified | primitive security assumption |
| CSPRNG | fresh local source | OS/library assumption |
| Storage commit | crash behavior specified | medium durability/rollback |
| FFI | handle and buffer contract | caller pointer behavior |
| Server/DHT/Relay | abstract interface only | deployment implementation |
