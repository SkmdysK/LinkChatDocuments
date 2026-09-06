# 06. Application State Machine

## 6.1 Turn ownership

```text
Leader(t) = Alice, if t mod 2 = 0
Leader(t) = Bob,   if t mod 2 = 1
```

Only the Leader for `expected_turn` may create an Application Envelope. v1.0 does not
define a receive window, generic out-of-order processing, or concurrent Application commits.

## 6.2 Endpoint state

```text
EndpointState = {
    SessionContext,
    local_identity_secret,
    verified_peer_identity_public_key,
    current_private_receiver_package,
    current_verified_peer_public_package,
    expected_turn,
    consumed_message_ids,
    package_chain_checkpoint
}
```

`current_private_receiver_package` contains private key material and its public projection.
The peer Package is public and verified; it never contains the peer's private material.

## 6.3 Send preparation

The send operation is:

```text
1. Check that the local endpoint is Leader(expected_turn).
2. Select the current verified peer public Package.
3. Generate ephemeral X25519 material.
4. Encapsulate with X25519 and ML-KEM-768.
5. Derive Hybrid PRK, Header Key, Message Key, and nonces.
6. Generate the sender's next local public Receiver Package.
7. Construct the Header and return_receiver_package.
8. Seal the canonical Header with Header AEAD.
9. Encode the fixed PayloadFrame.
10. Seal the PayloadFrame with Message AEAD.
11. Return the Envelope and a complete pending send state.
```

The next local Package is generated locally. Peer input does not select its private key,
Token, KeyID, or secret bytes.

## 6.4 Receive processing order

The receiver MUST use this order:

```text
bounded decode of the 4096-byte Envelope
→ outer Token and context checks
→ X25519 and ML-KEM decapsulation
→ Hybrid KDF
→ Header AEAD Open with HeaderAD
→ canonical Header decode
→ Session, Turn, Direction, MessageID, and KeyID checks
→ receiver package hash check
→ return Package signature and chain checks
→ Message KDF
→ Message AEAD Open with MessageAD
→ PayloadFrame length and zero-padding checks
→ duplicate and consumed-message checks
→ local fresh Receiver Package generation
→ complete pending-state construction
→ durable prepare and atomic commit
→ application delivery
```

Header authentication failure MUST stop before Message AEAD. Message authentication failure
MUST stop before Package generation and state commit.

## 6.5 Package validation

The receiver MUST validate:

```text
protocol version and cipher suite
SessionID
owner and direction
Turn and generation
KeyID
expiration
previous_package_hash
recomputed package_hash
Ed25519 package_auth
```

The Header's `sender_package_hash` MUST equal the hash of its
`return_receiver_package`. The `receiver_package_hash` MUST equal the hash of the receiver's
current local public Package.

## 6.6 Successful receive transition

After every check succeeds, the receiver creates:

```text
(new_xsk, new_xpk)   = X25519.KeyGen(CSPRNG)
(new_pqsk, new_pqpk) = ML-KEM-768.KeyGen(CSPRNG)
new_token            = CSPRNG(32 bytes)
new_key_id           = locally unique identifier
new_generation       = old_generation + 1
new_previous_hash    = Hash(current_public_package)
```

The new private material, public projection, KeyID, Token, generation, expected Turn,
verified peer Package, and consumed MessageID set MUST be committed as one logical state.

## 6.7 Failure transition

The following results MUST be no-state-change:

```text
malformed or non-canonical input
unsupported version or suite
Session or outer-context mismatch
Token or KeyID mismatch
past Turn / replay
future Turn
wrong Direction
Package hash, signature, or chain failure
KEM failure
Header AEAD failure
Message AEAD failure
invalid PayloadFrame length or padding
duplicate message
storage prepare or commit failure
```

Failure MUST NOT change the current Package, peer Package, expected Turn, consumed IDs,
generation, or retained Message Key state.

## 6.8 Replay and duplicate behavior

```text
turn < expected_turn       → REPLAY
turn > expected_turn       → FUTURE_TURN
message_id already used    → DUPLICATE
```

These results may be observed by Transport, but they MUST NOT rotate a Receiver Package.

## 6.9 State non-interference

With the honest endpoint state and local fresh schedule fixed, peer-controlled input may
affect acceptance, rejection, application delivery, and Transport observations. It MUST
NOT choose the honest endpoint's next X25519 private key, ML-KEM private key, Token, KeyID,
or Package secret.

## 6.10 State lifecycle

```text
ACTIVE → PROCESSING → PENDING → COMMITTED → OLD → ERASED
```

`OLD` and `ERASED` describe local lifecycle decisions. They do not prove that every copy of
historical bytes has been physically removed from a platform.

## 6.11 No independent mutation API

The core API MUST NOT expose independent setters for:

```text
receiver secret
receiver public key
receiver Token
generation
expected Turn
unverified peer Package
```

State is produced only by protocol processing, storage commit, or validated recovery.
