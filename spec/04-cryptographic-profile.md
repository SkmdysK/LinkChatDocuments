# 04. Cryptographic Profile

## 4.1 Suite identifier

The v1.0 suite is:

```text
cipher_suite = 0x0001
name         = V1HybridMlKem768
```

It consists of Ed25519 identity authentication, an X25519 ephemeral-DH wrapper,
ML-KEM-768, HKDF-SHA-256, SHA-256, and ChaCha20-Poly1305. A Session MUST NOT silently
switch any of these choices.

| Function | Primitive |
| --- | --- |
| Identity signature | Ed25519 |
| Classical wrapper | X25519 ephemeral DH, 32-byte wrapper ciphertext |
| Post-quantum KEM | ML-KEM-768 |
| KDF | HKDF-SHA-256 |
| Hash | SHA-256 |
| AEAD | ChaCha20-Poly1305 with 12-byte nonce and 16-byte tag |
| Optional MAC | HMAC-SHA-256 |
| Fresh material | OS CSPRNG or an equivalent vetted library source |

## 4.2 Fixed sizes

```text
Ed25519 public key       = 32 bytes
Ed25519 secret seed      = 32 bytes
Ed25519 signature        = 64 bytes
X25519 public/secret     = 32 bytes
ML-KEM-768 public key    = 1184 bytes
ML-KEM-768 ciphertext    = 1088 bytes
KEM shared secret        = 32 bytes
HKDF output key          = 32 bytes
ChaCha20-Poly1305 nonce  = 12 bytes
ChaCha20-Poly1305 tag    = 16 bytes
SHA-256 digest           = 32 bytes
Mailbox Token            = 32 bytes
```

## 4.3 Ed25519

```text
IdentityPublicKey = Ed25519.PublicKey(IK_priv)
Signature         = Ed25519.Sign(IK_priv, canonical_message)
valid             = Ed25519.Verify(IK_pub, canonical_message, Signature)
```

All signed inputs MUST include the protocol domain and use the canonical encoding in
Section 05. Long-term identity signatures are used for identity binding and Receiver
Package authentication, not automatically appended to every ordinary message.

## 4.4 X25519 wrapper

The wrapper produces a 32-byte ephemeral public value as its ciphertext:

```text
(esk_X, epk_X) = X25519.KeyGen(CSPRNG)
ct_X           = epk_X
ss_X           = X25519(esk_X, receiver_x25519_public_key)
```

The receiver computes:

```text
ss_X = X25519(receiver_x25519_private_key, ct_X)
```

The implementation MUST use the standard X25519 scalar decoding and clamping behavior.
Low-order inputs and an all-zero shared result MUST be rejected and mapped to a stable
cryptographic failure. The wrapper's ciphertext is exactly 32 bytes.

## 4.5 ML-KEM-768

```text
(pk_PQ, sk_PQ) = ML-KEM-768.KeyGen(CSPRNG)
(ct_PQ, ss_PQ) = ML-KEM-768.Encapsulate(pk_PQ)
ss_PQ          = ML-KEM-768.Decapsulate(sk_PQ, ct_PQ)
```

The v1.0 wire representation uses a 1184-byte public key and a 1088-byte ciphertext.
The receiver secret representation is implementation-specific but MUST reconstruct the
same public projection. Invalid ciphertexts MUST NOT expose library-internal failure
details through a remote response.

## 4.6 Hybrid input

The two shared secrets are combined with a domain-separated canonical input:

```text
HybridInput = CanonicalContext(
    "LinkChat/hybrid/v1",
    protocol_version,
    cipher_suite,
    session_id,
    turn,
    receiver_key_id,
    ct_X,
    ct_PQ,
    ss_X,
    ss_PQ
)

PRK_hybrid = HKDF-Extract(zero_salt_32, HybridInput)
```

An implementation MUST NOT use either KEM shared secret directly as an AEAD key.

## 4.7 Derived keys

```text
MessageInfo = CanonicalContext(
    "LinkChat/message/v1",
    protocol_version, cipher_suite, session_id, turn,
    direction, message_id, receiver_key_id
)
MK = HKDF-Expand(PRK_hybrid, MessageInfo, 32)
```

```text
HeaderInfo = CanonicalContext(
    "LinkChat/header/v1",
    protocol_version, cipher_suite, session_id, turn,
    direction, receiver_key_id
)
HK = HKDF-Expand(PRK_hybrid, HeaderInfo, 32)
```

`MK` and `HK` MUST be distinct. A Message Key is used for one Application message only.

## 4.8 Nonces

```text
NonceInfo = CanonicalContext(
    "LinkChat/nonce/v1",
    session_id, turn, direction, message_id, receiver_key_id
)

message_nonce = HKDF-Expand(
    HKDF-Extract(zero_salt_32, MK), NonceInfo, 12
)
```

Header nonce derivation uses the same context shape with the independent domain
`LinkChat/header-nonce/v1`. Nonce uniqueness relies on unique MessageID, correct Turn,
Direction and KeyID binding, and fresh message keys.

## 4.9 Package hash and authentication

```text
package_hash = SHA-256(
    CanonicalContext("LinkChat/package-hash/v1", PackageBody)
)

package_auth = Ed25519.Sign(
    IK_priv_owner,
    CanonicalContext("LinkChat/receiver-package-auth/v1", PackageBody)
)
```

`package_auth` is not part of `PackageBody`. The verifier MUST recompute the package hash.

## 4.10 AEAD and associated data

```text
encrypted_header = ChaCha20-Poly1305.Seal(
    HK, header_nonce, HeaderAD, Canonical(Header)
)

ciphertext = ChaCha20-Poly1305.Seal(
    MK, message_nonce, MessageAD, PayloadFrame
)
```

`Open` is a combined decrypt-and-authenticate operation. Failure MUST stop processing;
it MUST NOT update the consumed-message set or create a new Receiver Package.

Because `message_id` is inside the encrypted Header, it cannot be an input to HeaderAD.
The exact values are:

```text
HeaderAD = CanonicalContext(
    "LinkChat/header-ad/v1",
    protocol_version,
    cipher_suite,
    session_id,
    turn,
    direction,
    receiver_key_id
)
```

```text
MessageAD = CanonicalContext(
    "LinkChat/message-ad/v1",
    protocol_version,
    cipher_suite,
    session_id,
    turn,
    direction,
    message_id,
    receiver_key_id
)
```

Header AEAD authenticates the complete Header, including `message_id`. After successful
Header open and validation, the receiver constructs MessageAD using that authenticated ID.

## 4.11 Randomness and erasure

The following values MUST come from a CSPRNG:

```text
S0, Receiver key pairs, ephemeral X25519 key,
Mailbox Token, Path Seed, and any randomized KeyID material
```

Protocol non-use is not proof of physical erasure. Swap, crash dumps, snapshots, backups,
allocator copies, and hardware remnants remain implementation or platform concerns.

## 4.12 Computational assumptions

The formal model may receive certificates for:

```text
X25519 wrapper security
ML-KEM-768 security
HKDF-SHA-256 pseudorandomness
ChaCha20-Poly1305 confidentiality and integrity
Ed25519 unforgeability
CSPRNG unpredictability and recovery behavior
```

These are external assumptions attached to named games. They are not derived from the
state-machine theorems alone.
