# 00. Overview and Research Position

## 0.1 Purpose

Link Chat v1.0 is specified as a small protocol core for studying the interaction between
standard cryptographic primitives and endpoint-local state evolution. The specification is
intended to be sufficient for independent implementations to reproduce the same protocol
objects and state decisions.

The protocol's distinctive construction is not a new cipher. It is the combination of:

```text
independent Receiver Secret State at each endpoint
strict alternating Application Turns
authenticated public Receiver Packages
atomic local package rotation after accepted input
an explicit separation between Application state and Transport state
```

## 0.2 Fixed architecture

```text
out-of-band identity confirmation
        ↓
Ed25519 identity binding
        ↓
S0 bootstrap and Session Context
        ↓
endpoint-local Receiver Secret State
        ↓
authenticated public Receiver Package
        ↓
strict alternating Application Turn
        ↓
X25519 wrapper + ML-KEM-768
        ↓
HKDF-SHA-256 domain-separated keys
        ↓
Header AEAD + Message AEAD
        ↓
Transport / Mailbox capability
```

Alice and Bob never exchange their Receiver private keys. Each endpoint uses the peer's
public Package to construct a message and uses its own private Package to process a message
addressed to it.

## 0.3 Research questions

The specification makes the following questions executable:

1. Does a malicious peer input change the honest endpoint's next private state, or only the
   accept/reject result and application delivery result?
2. Do replay, future-turn, malformed, authenticated-failure, and storage-failure paths
   leave the current Application state unchanged?
3. Can a future-state challenge be exposed through a real fixed-format message ciphertext?
4. Can the protocol game be reduced to named X25519, ML-KEM, HKDF, AEAD, signature, and
   CSPRNG games without silently assuming the target conclusion?

## 0.4 Non-goals

This version does not specify:

- Matrix-style federation, server discovery, or server administration;
- a particular DHT algorithm or Relay topology;
- account, contact, registration, login, or user-interface semantics;
- generic out-of-order windows or concurrent Application commits;
- a complete proof of external cryptographic libraries or operating systems;
- a guarantee against a global passive traffic analyst.

## 0.5 Minimal endpoint role

An implementation conforming to the core must be able to:

```text
create or restore local endpoint state
retain a verified peer public Package
prepare a send on the local Leader Turn
prepare a receive on the expected peer Turn
commit a complete pending state atomically
recover after a crash according to its storage capability
```

Transport, Mailbox, Path, and application code may be replaced if they preserve the
interfaces and invariants in this specification.

## 0.6 Core design sentence

> An Application message is authenticated protocol input; it is not a credential granting
> ownership of the receiver's private state.
