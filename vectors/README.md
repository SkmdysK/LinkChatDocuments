# Link Chat v1.0 Test Vector Directory

This directory contains byte-level interoperability vectors. Structural vectors are
fully specified here; cryptographic vectors use test-only inputs and must never be
copied into production configuration. Recommended files are:

```text
identity-binding.json
bootstrap.json
package-structural.json
application-message.json
context-aad.json
payload-frame.json
envelope-layout.json
hkdf-domain-framing.json
aead-roundtrip.json
kem-boundaries.json
replay-and-recovery.json
```

Every vector must identify the protocol version, cipher suite, canonical format version,
input bytes, expected output bytes, expected result, and whether all key material is
test-only.

The minimum set is:

```text
1. PackageBody canonical round-trip and exact length
2. Package Hash and Ed25519 Package Auth
3. S0 bootstrap derivation
4. X25519 wrapper and ML-KEM-768 encapsulation
5. Hybrid PRK and Message/Header derivation
6. HeaderAD without message_id
7. MessageAD with message_id
8. fixed PayloadFrame and zero padding
9. complete 4096-byte Envelope
10. each authentication and payload failure path
11. replay, future-turn, and duplicate behavior
12. storage crash recovery projections

The JSON files deliberately distinguish exact bytes from a compact byte-pattern
construction. A consumer MUST expand the construction, encode the object, and compare
the stated length and digest before accepting a vector.

## Reproduction commands

From the Rust reference kernel:

```text
cd <rust-kernel-repository>
cargo fmt --all -- --check
cargo check --workspace
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo run -p linkchat-cli -- test-vector
```

The final command checks the frozen manifest categories. It does not claim that the
external cryptographic primitives have been formally proved secure.
```

Vectors prove behavior for supplied inputs. They do not replace formal universal theorems
or external primitive security assumptions.
