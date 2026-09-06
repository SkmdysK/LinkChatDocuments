# 07. Transport and Mailbox Interfaces

## 7.1 Boundary

Transport, Mailbox, and Path carry opaque Envelopes. They do not own Application state
authority.

```text
Application Core
    ↑ verified Envelope / delivery result
Transport Adapter
    ↑ bytes / retry / timing
Mailbox Adapter
    ↑ Token / Envelope record
Storage Adapter
```

This specification defines interface semantics, not a public server, DHT algorithm, Relay
topology, or network connection manager.

## 7.2 Mailbox operations

```text
put(token, envelope, metadata) -> PutResult
get(token, limits)              -> GetResult
delete(token, record_id)        -> DeleteResult (optional)
```

The Token is a 32-byte opaque capability. A Mailbox MAY map it to a database key, HTTP path,
or DHT key, but MUST NOT treat it as a Message Key or Receiver Secret.

## 7.3 Untrusted behavior

A Mailbox or network may:

```text
drop, delay, duplicate, reorder, withhold,
return malformed bytes, return old data, return fake data,
limit result count, or become unavailable
```

The endpoint MUST process all returned bytes as untrusted input. Availability behavior is
outside the cryptographic state claim.

## 7.4 ACK and SACK

ACK and SACK are Transport/Control objects. Their protocol version is carried by the
common 9-byte object header. The ACK payload contains:

```text
cipher_suite, session_id, turn, direction,
message_id, delivery_status
```

The v1.0 ACK status enumeration is:

```text
0 Accepted
1 Rejected
2 Duplicate
3 Replay
```

An optional SACK payload contains:

```text
cipher_suite, session_id, turn, direction,
base_message_id, bitmap
```

ACK/SACK MUST NOT:

```text
advance Application Turn
rotate a Receiver Package
mark an Application message consumed
restore an erased Message Key
```

An authenticated Delivery Receipt requires a separately versioned extension.

## 7.5 Retry and idempotence

Transport MAY retry the same Envelope. The core behavior is:

```text
first complete validation and commit → Accepted
same message after commit             → Duplicate
```

Retry MUST NOT create a second KEM encapsulation, Message Key, Package rotation, or commit.

## 7.6 Transport metadata

Transport MAY maintain:

```text
record_id, received_at, attempt_count,
expiry, route metadata, delivery status
```

These fields are not Application Header fields. They MUST NOT directly write Receiver
Secret State or change the Application Turn.

## 7.7 Time and length policy

The endpoint API MAY receive `now`, TTL, and next-expiration values from the caller. The
core MUST NOT silently invent a clock policy. The Envelope remains 4096 bytes regardless of
logical application length.

The formal observation model MAY include event time, logical length, Envelope length,
ACK/error, accept/reject, state-update count, and Package-update count.

## 7.8 Token lifecycle

```text
created → active → selected → consumed/expired
```

A successful Mailbox `GET` is not a successful Application receive. Only the Application
state machine can consume a message or rotate a Package.

## 7.9 Path state

Path state is independent of Receiver Secret State, Token, and Message Key:

```text
PathSeed ← CSPRNG
PathKey  = HKDF-Expand(..., "LinkChat/path/v1", ...)
```

Multi-hop transport is a deployment choice and is not required by the core state machine.

## 7.10 Availability boundary

Replication, redundant queries, re-publication, TTL, and rate limits may improve delivery
behavior. They do not create a protocol availability theorem. A malicious Mailbox may retain
or refuse ciphertext indefinitely.
