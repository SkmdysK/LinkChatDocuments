# 08. Storage, Commit, and Recovery

## 8.1 Storage role

Storage maps protocol pending/current states to recoverable local records. It is not a
cryptographic primitive and it is not automatically rollback-resistant.

Abstract operations:

```text
load()                  -> CurrentState
prepare_commit(pending) -> PreparedCommit
commit(prepared)        -> Result
recover()               -> CurrentState
```

## 8.2 Record contents

A persistent record SHOULD include:

```text
SessionID
expected Turn
current private Receiver Package
current verified peer public Package
previous Package checkpoint
current generation
current protocol Package Hash
previous protocol Package Hash
consumed MessageIDs
pending record, when present
commit marker
```

Protocol Package Hash and storage-record Hash are different values and MUST remain separate.

## 8.3 Commit ordering

The recommended durable order is:

```text
1. Write and validate Pending.
2. Complete a durability barrier.
3. Write the commit marker.
4. Complete a durability barrier.
5. Atomically publish Current.
6. Complete the required directory or medium barrier.
7. Retire the old state.
8. Attempt secure erasure.
```

Old state MUST NOT be erased before the new state is durable.

## 8.4 Generation and chain rules

Every accepted Application Turn produces:

```text
generation_new    = generation_old + 1
previous_hash_new = package_hash_old
```

Recovery MUST reject generation rollback, chain discontinuity, retired KeyID reuse, and a
private/public key projection mismatch.

## 8.5 Recovery matrix

| Crash condition | Required recovery result |
| --- | --- |
| Pending incomplete | discard Pending, retain Current |
| Pending complete, no marker | discard Pending, retain Current |
| Marker present, Current not published | adopt Pending as Current |
| Current published, old state remains | use new Current; retire old state later |
| Record hash mismatch | report storage corruption and refuse normal recovery |
| Package chain mismatch | report rollback/fork and refuse normal recovery |
| Private/public projection mismatch | refuse recovery |

An implementation may use another medium, but the same persisted image MUST NOT yield two
different valid Application states under two ordinary restart paths.

## 8.6 Capability declaration

An implementation MUST expose or document:

```text
rollback_resistant()         -> true / false
durable_commit_supported()   -> true / false
cross_process_coordination() -> true / false
```

Atomic rename, `fsync`, or directory synchronization alone does not establish rollback
resistance. A true claim requires an appropriate monotonic external or hardware-backed
property.

## 8.7 FFI and process boundaries

An in-process memory adapter proves only in-process behavior. Cross-process use requires a
stable record format, locking or equivalent coordination, marker recovery, restart rules,
and defined handle behavior after errors.

An opaque FFI handle does not provide these properties by itself.

## 8.8 Concurrency boundary

v1.0 does not permit two Application Turns to commit concurrently against one endpoint.
Transport may fetch in parallel, but Application commit MUST be serialized.

ACK/SACK, retry, and Mailbox replication MUST NOT bypass storage commit.

## 8.9 Erasure

Old private keys, Message Keys, temporary KEM material, and temporary plaintext buffers
SHOULD enter an implementation-defined zeroization procedure once no longer referenced.
The implementation must document allocator copies, snapshots, backups, crash dumps, and
swap risks. Ordinary `drop` or file deletion alone is not a proof of physical erasure.
