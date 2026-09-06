# 13. End-to-End Reproducible Example

This chapter is a conformance walkthrough for one Alice-to-Bob Application Turn. It is
written so that an independent implementation can reproduce the same object boundaries,
lengths, and state transitions without implementing a server, DHT, Relay, Mailbox
service, GUI, or chat client.

The cryptographic inputs in this chapter are test-only. They are not credentials and MUST
NOT be used in a deployed session.

## 13.1 Fixture

Use the following fixed context:

```text
protocol_version     = 1.0
canonical_format     = 1
cipher_suite         = 0x0001 (V1HybridMlKem768)
session_id           = 42
initial_turn         = 0
leader               = Alice
direction            = Alice (0x00)
message_id           = 7
receiver_key_id      = 99
expiration           = 1700000000
application_payload  = UTF-8 "hello"
```

For structural vectors, use these test-only byte patterns:

```text
Alice X25519 public key       = 32 bytes of 0x11
Alice ML-KEM public key       = 1184 bytes of 0x22
Alice Mailbox Token           = 32 bytes of 0x33
Alice previous package hash   = 32 bytes of 0x44
Alice package authentication  = 64 bytes of 0x55
```

The package patterns are intentionally not valid public-key material for a production
cryptographic backend. They are used only to verify canonical field order and lengths.
The complete structural package is defined by
[`../vectors/package-structural.json`](../vectors/package-structural.json).

## 13.2 Deterministic test randomness

An implementation that wants deterministic cryptographic artifacts MAY use this test-only
source:

```text
R(seed, counter) = SHA-256(seed || u64_be(counter))
```

where `seed` is exactly 32 bytes and `counter` starts at zero. Concatenate successive
32-byte blocks and consume bytes in the order required by the operation. The reference
Rust test backend uses this schedule for test-only key generation and encapsulation. A
production backend MUST use an OS CSPRNG or an equivalent vetted source instead.

The deterministic source is not a security assumption. It exists to make a test run
repeatable and to expose disagreements about byte consumption, domain framing, or field
ordering.

## 13.3 Initial state

Before Turn 0, both endpoints have independently created and verified their own private
Receiver Package. Alice stores her private X25519 and ML-KEM material and publishes only
her public projection. Bob stores the analogous private material and publishes Bob's
public projection. Both endpoints pin the peer identity public key and the SessionID.

The public package used by Alice as the receiver is Bob's current package:

```text
receiver_package_hash = SHA-256(
    CanonicalContext("LinkChat/package-hash/v1", PackageBody_Bob)
)
receiver_key_id        = 99
receiver_mailbox_token = Token_Bob
```

Alice's current package hash is placed in the Header as `sender_package_hash`. At Turn 0,
the returned package in the Header is Alice's current package, not a newly generated
package. Bob generates a new local package only after the authenticated message has been
fully accepted.

## 13.4 Alice prepares the message

Alice performs the following operations without publishing a state change:

```text
1. Confirm Endpoint::leader(turn=0) = Alice.
2. Confirm Bob's package context, expiration, KeyID, and Mailbox Token.
3. Encode PayloadFrame("hello"):
       0005 || 68656c6c6f || 1284 zero bytes
   Result length = 1291 bytes.
4. Encapsulate to Bob's X25519 public key and ML-KEM-768 public key.
5. Build HybridInput and derive PRK_hybrid with HKDF-SHA-256.
6. Derive Header Key and Header nonce.
7. Construct the logical Header.
8. Seal the Header with HeaderAD.
9. Derive Message Key and Message nonce.
10. Seal the PayloadFrame with MessageAD.
11. Construct the fixed-size Envelope.
12. Return a pending send transition.
```

The logical Header fields are:

```text
cipher_suite            = 0x0001
session_id              = 42
turn                    = 0
direction               = Alice (0x00)
message_id              = 7
receiver_key_id         = 99
message_type            = Application (0x00)
sender_package_hash     = Hash(current Alice public package)
receiver_package_hash   = Hash(current Bob public package)
return_receiver_package = current Alice public package
```

The canonical logical Header is 1588 bytes. Header AEAD adds a 16-byte tag, so the
`encrypted_header` field is 1604 bytes.

The message ciphertext is:

```text
ChaCha20-Poly1305.Seal(
    MessageKey,
    message_nonce,
    MessageAD,
    PayloadFrame("hello")
)
```

Its length is 1291 + 16 = 1307 bytes.

## 13.5 HeaderAD and MessageAD ordering

Alice constructs HeaderAD before Header decryption can occur at Bob:

```text
HeaderAD = CanonicalContext(
    "LinkChat/header-ad/v1",
    protocol_major=1,
    protocol_minor=0,
    cipher_suite=0x0001,
    session_id=42,
    turn=0,
    direction=0,
    receiver_key_id=99
)
```

`message_id` is deliberately absent because it is inside the encrypted Header.

After Bob opens and decodes the authenticated Header, Bob constructs:

```text
MessageAD = CanonicalContext(
    "LinkChat/message-ad/v1",
    protocol_major=1,
    protocol_minor=0,
    cipher_suite=0x0001,
    session_id=42,
    turn=0,
    direction=0,
    message_id=7,
    receiver_key_id=99
)
```

The exact bytes for this context pair are in
[`../vectors/context-aad.json`](../vectors/context-aad.json).

## 13.6 Envelope layout

The final Envelope is exactly 4096 bytes:

```text
offset  length  field
0       9       object header
9       36      mailbox_token field prefix + value
45      36      X25519 ciphertext field prefix + value
81      1092    ML-KEM ciphertext field prefix + value
1173    1608    encrypted_header field prefix + value
2781    1311    ciphertext field prefix + value
4092    4       zero-length padding field
```

The outer Mailbox Token is Bob's current Token. The outer object contains no plaintext
message ID, plaintext payload, private key, or secret state.

The layout and a patterned 4096-byte construction are specified in
[`../vectors/envelope-layout.json`](../vectors/envelope-layout.json).

## 13.7 Bob prepares and validates the message

Bob receives opaque Envelope bytes and performs the following order:

```text
1. Require exactly 4096 bytes and canonical Envelope encoding.
2. Match the outer Mailbox Token to Bob's current local package.
3. Decapsulate X25519 and ML-KEM-768.
4. Derive the same Hybrid PRK.
5. Derive Header Key and Header nonce.
6. Open the encrypted Header using HeaderAD.
7. Canonically decode the Header.
8. Validate SessionID, Turn, Direction, MessageID, MessageType, and KeyID.
9. Validate receiver_package_hash against Bob's current public package.
10. Verify Alice's returned package signature and package-chain relation.
11. Derive Message Key and Message nonce.
12. Open the ciphertext using MessageAD.
13. Validate PayloadFrame length and zero padding.
14. Reject a duplicate MessageID if it has already been consumed.
15. Generate Bob's fresh Receiver Package locally.
16. Build a complete pending receive transition.
17. Prepare and durably commit the endpoint state.
18. Deliver "hello" only after commit succeeds.
```

Any failure through step 13 leaves Bob's published state unchanged. A duplicate check
also leaves state unchanged. A storage failure after preparation leaves the published
state unchanged and MUST NOT deliver the payload as committed.

## 13.8 State transition

On successful commit:

```text
Bob current package       -> freshly generated Bob package
Bob package generation    -> generation + 1
Bob expected turn         -> 1
Bob consumed message IDs  -> previous IDs plus 7
Bob peer package          -> authenticated Alice return package
Bob public snapshot       -> updated atomically
```

Alice's send commit advances Alice's expected turn from 0 to 1 and records MessageID 7 as
used. A dropped `PreparedSend` does not advance Alice. Bob's `PreparedReceive` likewise
does not advance Bob until its explicit commit succeeds.

## 13.9 Reverse Turn

Turn 1 is symmetric:

```text
leader               = Bob
direction            = Bob
receiver             = Alice
expected_turn_after  = 2
```

The same construction is reused with Bob as sender, Alice's current public package as the
receiver package, a new MessageID, and fresh encapsulation material. No ACK, SACK, retry,
timestamp, or mailbox delivery event can substitute for the Application commit.

## 13.10 Conformance result

An implementation conforms to this example when it agrees on:

```text
object tags and header fields
field order and big-endian lengths
PayloadFrame length and zero padding
HeaderAD / MessageAD construction
Envelope component lengths and offsets
prepare/commit state visibility
failure no-state-change behavior
Turn 0 -> Turn 1 transition
```

The example is a reproducibility and interoperability artifact. It is not, by itself, a
security proof for the underlying cryptographic primitives or for a deployment.
