# 11. Conformance and Test Vectors

## 11.1 Conformance levels

| Level | Required evidence |
| --- | --- |
| Codec | canonical encoding, lengths, field order, round-trip |
| Crypto | primitive calls, domains, nonce, AAD, AEAD results |
| State | Turn, Package, replay, failure, atomic commit |
| Boundary | Storage, Transport, Mailbox, and FFI capability declarations |

Passing Codec tests does not establish State conformance. Passing State tests does not
establish external primitive security.

## 11.2 Required test categories

An implementation SHOULD test:

```text
canonical encode/decode round-trip
non-canonical encoding rejection
truncated and oversized input rejection
fixed 4096-byte Envelope
PayloadFrame logical length and zero padding
HeaderAD without message_id
MessageAD with authenticated message_id
Package signature and chain validation
wrong Session / Turn / Direction / KeyID
past-turn replay and future-turn rejection
duplicate message idempotence
KEM, Header AEAD, and Message AEAD tampering
every reject path no-state-change
fresh Package generation after successful receive
atomic prepare / commit / recover
crash matrix at every storage step
ACK/SACK non-interference
bounded Rust/Lean differential checks
```

## 11.3 Required vector fields

Every published vector SHOULD include:

```text
vector_id
protocol_version
cipher_suite
canonical_format_version
input bytes
expected output bytes
expected accept/reject result
expected public-state projection
test-only key-material marker
```

Cryptographic vectors must also identify whether randomness is deterministic test input or
an actual CSPRNG. Test keys MUST NOT be used in production.

The maintained v1.0 vector set is in [`../vectors/`](../vectors/). It includes exact
canonical bytes for a small ApplicationMessage and context inputs, compact patterned
constructions for the fixed Package and Envelope layouts, and explicit correctness or
failure expectations for bootstrap, KEM, HKDF, AEAD, replay, and recovery. A patterned
construction is a vector: the implementation must expand the repeated-byte fields before
encoding and digesting the result.

## 11.4 Minimum vector set

```text
PackageBody round-trip and exact length
Package Hash and Ed25519 Package Auth
S0 bootstrap derivation
X25519 wrapper and ML-KEM-768 encapsulation
Hybrid PRK and Message/Header derivation
HeaderAD without message_id
MessageAD with message_id
PayloadFrame and zero padding
complete 4096-byte Envelope
each authentication and payload failure path
replay, future-turn, and duplicate behavior
storage crash recovery projections
```

## 11.5 CryptoProvider contract

A backend must expose typed operations equivalent to:

```text
ed25519_keygen / sign / verify
x25519_keygen / encap / decap
mlkem768_keygen / encap / decap
hkdf_extract / hkdf_expand
sha256
chacha20poly1305_seal / open
random_bytes
```

It must document parameter sets, input validation, stable failure behavior, cost/resource
bounds, constant-time claims, memory handling, and zeroization behavior.

## 11.6 Endpoint API contract

The core API should expose:

```text
prepare_send(plaintext) -> PreparedSend
prepare_receive(envelope) -> PreparedReceive
commit(prepared) -> Result
recover() -> Result<CurrentState>
```

It must not expose independent mutation of Secret, Public Key, Token, KeyID, generation,
peer Package, or expected Turn.

## 11.7 Stable failure categories

Remote-visible errors SHOULD be categorized as:

```text
Malformed
UnsupportedVersion
UnsupportedSuite
ContextMismatch
Replay
FutureTurn
Duplicate
PackageInvalid
KemFailure
AuthenticationFailure
PayloadInvalid
StorageFailure
Unavailable
```

Internal diagnostics may be more detailed, but must not expose private keys, Message Keys,
complete plaintexts, or CSPRNG state.

## 11.8 Release report

A conforming implementation should publish its compiler and dependency versions, crypto
backend, parameter set, vector revision, test command, passed/skipped categories, and known
external trust boundaries. Large adversarial simulations provide coverage evidence; they do
not replace a Lean theorem, a PPT reduction, or a primitive security assumption.

## 11.9 Interoperability declaration

An implementation may claim `v1.0 interoperable` only when it matches:

```text
canonical encoding and object tags
field order, offsets, and fixed lengths
cryptographic profile and wrapper semantics
HeaderAD / MessageAD construction order
Package and chain validation
state transitions and failure behavior
Transport boundary and ACK/SACK non-interference
```

The declaration should include the vector revision and the exact commands used to run the
codec, crypto-boundary, state, and storage tests. A successful vector run is evidence of
implementation agreement for those supplied inputs; it is not a universal theorem and
does not replace the external primitive-security certificates described in Section 10.
