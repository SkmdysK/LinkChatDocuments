# 01. Terms, Notation, and Normative Language

## 1.1 Participants

| Term | Meaning |
| --- | --- |
| Alice / Bob | The two endpoint roles in one Session |
| Leader | The endpoint allowed to create the current Application Turn |
| Receiver | The endpoint processing the current Application Turn |
| Peer | The other endpoint from the local endpoint's perspective |
| Adversary | A modelled party controlling allowed inputs and network behavior |

## 1.2 Protocol objects

```text
IK       long-term Ed25519 identity key pair
S0       offline bootstrap secret
SessionID session-scoped identifier
RS       private Receiver Secret State
RP       complete Receiver Package, including private material locally
PK       public projection of a Receiver Package
Token    Mailbox capability
MK       per-message key
HK       per-header key
Turn     strict alternating Application counter
```

`RS` and `PK` are intentionally different types. A public Package MUST NOT encode a
Receiver private key, CSPRNG state, Message Key, or Path secret.

## 1.3 Turn convention

```text
Leader(t) = Alice, when t mod 2 = 0
Leader(t) = Bob,   when t mod 2 = 1
```

Turn zero is the first Application Turn. Transport retries, ACK counts, replica counts,
and storage generations are not Application Turns.

## 1.4 Package terminology

`PackageBody` is the public field set used for Package Hash and Package Auth. The
`package_auth` field is not part of `PackageBody`.

`current package` is the local package currently authorized for use. `peer public package`
is the last verified public package belonging to the peer. `pending package` is a candidate
state that has passed protocol processing but has not yet been durably committed.

## 1.5 Complete adversary view

The formal security model may expose:

```text
controlled endpoint state, plaintexts, randomness, ciphertexts, and Tokens
network, Transport, Mailbox, and DHT traces
honest endpoint public output
ACK/error and accept/reject observations
event time, logical length, fixed Envelope length
state-update and Receiver Package update counts
```

An experiment that exposes only a subset MUST identify it as a projection rather than a
complete view.

## 1.6 Notation

```text
Canonical(x)  unique byte encoding of x
Hash(x)       SHA-256 result under the specified domain
Seal(k,n,a,m) AEAD encryption
Open(k,n,a,c) AEAD decryption and authentication
Gen(owner)    local fresh Package generation by owner
Commit(s)     atomic publication of pending state s
Reject        result with no Application state mutation
```

## 1.7 Normative language

- `MUST`: required for v1.0 interoperability;
- `MUST NOT`: prohibited by the core protocol;
- `SHOULD`: expected unless a documented implementation reason exists;
- `SHOULD NOT`: discouraged unless a documented implementation reason exists;
- `MAY`: permitted but not required for interoperability.
