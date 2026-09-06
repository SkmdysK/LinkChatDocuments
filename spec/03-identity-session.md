# 03. Identity, Bootstrap, and Session Initialization

## 3.1 Identity keys

Each endpoint has a long-term Ed25519 identity key pair:

```text
IK_priv
IK_pub = Ed25519.PublicKey(IK_priv)
```

Identity keys are used for offline binding, Receiver Package authentication, identity-change
confirmation, and re-pairing. Ordinary Application messages do not require a long-term
identity signature.

## 3.2 Out-of-band exchange

The initial out-of-band procedure MUST confirm at least:

```text
protocol_version
cipher_suite
IdentityPublicKey_A
IdentityPublicKey_B
SessionID
S0
InitialReceiverPublicPackage_A
InitialReceiverPublicPackage_B
PackageChainRoot_A
PackageChainRoot_B
```

The participants MUST verify identity fingerprints through a physical or equivalent
authenticated channel. A network-delivered first identity claim is not sufficient.

## 3.3 Identity binding

```text
binding = CanonicalEncode(
    "LinkChat/identity-binding/v1",
    protocol_version,
    cipher_suite,
    session_id,
    identity_public_key_A,
    identity_public_key_B
)
```

```text
sig_A = Ed25519.Sign(IK_priv_A, binding)
sig_B = Ed25519.Sign(IK_priv_B, binding)
```

The binding covers the selected protocol and Session context. It does not grant permission
to alter the peer's private Receiver State.

## 3.4 SessionID

Every Session MUST use a fresh `SessionID`. It is included in Package Hash, Package Auth,
Hybrid KDF, Message/Header KDF, nonces, and both AEAD associated-data values.

An endpoint MUST NOT silently reuse Receiver private state, Mailbox Token, or Path State
from another Session.

## 3.5 S0 bootstrap

```text
PRK_bootstrap = HKDF-Extract(32 zero bytes, S0)

bootstrap_key = HKDF-Expand(
    PRK_bootstrap,
    CanonicalEncode(
        "LinkChat/bootstrap/v1",
        protocol_version,
        cipher_suite,
        session_id
    ),
    32
)
```

`S0` is used for bootstrap authentication, Session binding, and initial state confirmation.
It is not the direct root of later Receiver private states. After bootstrap, `S0`,
`PRK_bootstrap`, and `bootstrap_key` MUST enter the implementation's erase procedure.

## 3.6 Initial Receiver State

Alice and Bob independently generate:

```text
(RS_X_0, RP_X_0)   = X25519.KeyGen(OS-CSPRNG)
(RS_PQ_0, RP_PQ_0) = ML-KEM.KeyGen(OS-CSPRNG)
Token_0            = OS-CSPRNG(32 bytes)
KeyID_0            = locally unique identifier
```

Only public keys, KeyID, Token, and the public Package are exchanged out of band.

## 3.7 Package chain roots

```text
PackageChainRoot(owner) = SHA-256(CanonicalEncode(
    "LinkChat/package-chain-root/v1",
    protocol_version,
    cipher_suite,
    session_id,
    identity_public_key_A,
    identity_public_key_B,
    owner
))
```

The initial Package's `previous_package_hash` MUST equal the owner's chain root. Later
Packages MUST link to the previous confirmed public Package owned by the same endpoint.

## 3.8 Identity changes and compromise boundary

If the peer identity public key changes, the endpoint MUST stop the Session, invalidate it,
reject the new Package, and require a new out-of-band verification.

If an identity secret is compromised, an attacker may forge Packages relying on that
identity. The affected Session MUST be replaced with a new SessionID, S0, Receiver State,
Package chain, and Mailbox Token set.
