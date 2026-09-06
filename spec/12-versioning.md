# 12. Versioning and Extensions

## 12.1 Version fields

`protocol_version` and `cipher_suite` are authenticated context fields. They participate in
Session, Package, KDF, AAD, and rejection behavior. An unsupported value MUST be rejected;
an implementation MUST NOT guess a compatibility mode.

## 12.2 v1.0 frozen items

```text
independent endpoint-local Receiver Secret State
strict Alice/Bob Application Turn alternation
atomic Receiver Package rotation
Envelope object field order and fixed lengths
ML-KEM-768 profile
HeaderAD excludes encrypted message_id
MessageAD includes authenticated message_id
ACK/SACK do not advance Application state
no generic receive window or concurrent Application commit
```

## 12.3 Extension requirements

An extension MUST define:

```text
extension identifier
canonical encoding
domain separation
negotiation and rejection behavior
whether it enters CompleteAdversaryView
whether it affects state transition
test vectors
```

Possible future extensions include authenticated delivery receipts, fragmentation, new KEM
suites, Transport metadata, and stronger storage capabilities. They must not silently alter
the v1.0 Application fields or state authority.

## 12.4 Changes requiring a new version

The following changes require a new protocol version or a corresponding formal-model change:

```text
introducing sequence/window semantics
accepting arbitrary out-of-order Application Turns
letting ACK/SACK rotate Receiver State
adding concurrent Application commits
changing HeaderAD fields
changing the fixed Envelope budget
deriving Receiver private state from peer input
```

## 12.5 Cipher-suite registry

Each suite entry must register:

```text
suite identifier
identity algorithm
classical wrapper and parameter rules
post-quantum KEM and parameter set
KDF and hash
AEAD
key lengths
nonce lengths
canonical encoding revision
```

## 12.6 Research and deployment profiles

`v1.0-research` identifies the frozen protocol/model/reference-implementation relationship.
A deployment profile must additionally state:

```text
conformance result
CryptoProvider implementation
durable-storage status
rollback-resistance status
cross-process coordination status
transport metadata observation model
```

## 12.7 Change record

Every specification change must record the old and new version, changed fields or state
rules, security-game impact, required Lean changes, required vector changes, and migration
or rejection behavior.

## 12.8 No implicit downgrade

If an implementation cannot determine whether a peer understands an extension, it must use
the v1.0 rejection behavior. It must not use a zero value, old KeyID, default suite, or
silent downgrade.
