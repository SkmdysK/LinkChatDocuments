# 02. Architecture and State Ownership

## 2.1 Layers

```text
Identity / Session Layer
        ↓
Receiver State Layer
        ↓
KEM / KDF / Message Layer
        ↓
Transport / Mailbox / Path Layer
```

Each layer may expose an interface to the next layer, but a lower-layer event MUST NOT
silently authorize an unrelated Application state transition.

## 2.2 Production endpoint state

A production endpoint owns exactly its local private state and verified peer public state:

```text
EndpointState = {
    immutable SessionContext,
    local Ed25519 identity secret,
    verified peer Ed25519 public key,
    current private Receiver Package,
    current verified peer public Package,
    expected Application Turn,
    consumed MessageID set,
    package-chain checkpoint
}
```

It MUST NOT own the peer's X25519 or ML-KEM private keys.

## 2.3 State-changing entry points

The core Application API has two preparation operations:

```text
prepare_send(application_plaintext) -> PreparedSend
prepare_receive(wire_envelope)      -> PreparedReceive
```

Each preparation result contains a complete candidate state. Preparation alone does not
publish state. Publication requires the storage `commit` operation.

The public API MUST NOT provide independent setters for a Receiver private key, public
key, Token, KeyID, generation, peer Package, or expected Turn.

## 2.4 Authority model

An Application message MAY:

```text
carry authenticated encrypted application data
carry the sender's authenticated public return Package
request one valid state-machine transition
```

It MUST NOT be interpreted as any of:

```text
SET_RECEIVER_PRIVATE_KEY
SET_RECEIVER_TOKEN
SET_RECEIVER_GENERATION
ROLLBACK_RECEIVER_PACKAGE
GRANT_STATE_OWNERSHIP
```

## 2.5 Application and Transport separation

Only `Leader(expected_turn)` may create a new Application Turn. The following Transport
events MUST NOT advance Application state:

```text
ACK / SACK
retry, drop, delay, duplicate, reorder
Mailbox replication or deletion
route failure
```

If a future version adds windows, concurrency, or generic out-of-order processing, it must
also update the formal state model and receive semantics.

## 2.6 Success and failure boundary

One accepted Application Turn atomically combines:

```text
consume the message
accept the authenticated peer return Package
generate a local fresh Receiver Package
advance the expected Turn
publish the complete pending state
```

Any parse, context, Package, KEM, AEAD, payload, replay, or storage failure MUST leave the
current Application state unchanged and MUST NOT deliver the application plaintext.

## 2.7 Untrusted dependencies

Transport, Mailbox, DHT, Relay, and ordinary filesystems may return malformed, old,
duplicated, delayed, missing, or modified data. They are input sources, not state authority.
