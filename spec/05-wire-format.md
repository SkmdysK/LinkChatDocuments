# 05. Canonical Encoding and Wire Format

## 5.1 Encoding profile

Every wire object uses the following object header:

```text
offset  size  field
0       1     object tag
1       2     canonical format version, unsigned big-endian
3       2     protocol major version, unsigned big-endian
5       2     protocol minor version, unsigned big-endian
7       2     field count, unsigned big-endian
```

The object header is therefore 9 bytes. v1.0 uses canonical format version `1`, protocol
version `1.0`, and rejects unsupported versions or field counts.

## 5.2 Field encoding

Each field is encoded as:

```text
u32_be field_length || field_bytes
```

The length prefix is 4 bytes, big-endian. Fields appear exactly in the order specified by
the object definition. There are no omitted optional fields in the fixed v1.0 objects.

The maximum accepted field length is 65536 bytes. A decoder MUST reject truncation,
overflow, trailing bytes, non-canonical re-encoding, and a field whose semantic length is
not the required fixed length.

## 5.3 Object tags and field counts

```text
0x01  ApplicationMessage  8 fields
0x02  ReceiverPackage     11 fields
0x03  Envelope            6 fields
0x04  ACK                 6 fields
0x05  SACK                6 fields
0x06  MailboxRecord       2 fields
0x07  Header              10 fields
0x7f  Context             variable, used for domain-separated inputs
```

## 5.4 Primitive encodings

```text
u8      exactly 1 byte, unsigned
u16     exactly 2 bytes, big-endian
u64     exactly 8 bytes, big-endian
ID      exactly one u64_be field
hash    exactly 32 bytes
Token   exactly 32 bytes
```

`SessionID`, `MessageID`, `KeyID`, `Turn`, and `PackageGeneration` are each encoded as a
single 8-byte big-endian field. `Direction` and `MessageType` are one-byte enumerations.

## 5.5 ReceiverPackage

The 11 fields, in order, are:

```text
0  cipher_suite          u16
1  session_id            u64
2  turn                  u64
3  generation            u64
4  key_id                u64
5  x25519_public_key     32 bytes
6  mlkem_public_key      1184 bytes
7  mailbox_token         32 bytes
8  expiration            u64
9  previous_package_hash 32 bytes
10 package_auth          64 bytes
```

The Package object tag is `0x02`. Its exact offsets are:

| Range | Content |
| --- | --- |
| 0..8 | object header |
| 9..14 | field 0 length and value |
| 15..26 | field 1 length and value |
| 27..38 | field 2 length and value |
| 39..50 | field 3 length and value |
| 51..62 | field 4 length and value |
| 63..98 | field 5 length and value |
| 99..1286 | field 6 length and value |
| 1287..1322 | field 7 length and value |
| 1323..1334 | field 8 length and value |
| 1335..1370 | field 9 length and value |
| 1371..1438 | field 10 length and value |

The canonical encoded Package length is exactly **1439 bytes**.

## 5.6 Header

The 10 fields, in order, are:

```text
0  cipher_suite
1  session_id
2  turn
3  direction
4  message_id
5  receiver_key_id
6  message_type
7  sender_package_hash
8  receiver_package_hash
9  return_receiver_package
```

The last field is the complete 1439-byte canonical ReceiverPackage. The Header object tag
is `0x07`; its canonical encoded length is exactly **1588 bytes**. The exact field end
offsets are:

```text
object header       0..8
cipher_suite        9..14
session_id          15..26
turn                27..38
direction           39..43
message_id          44..55
receiver_key_id     56..67
message_type        68..72
sender_package_hash 73..108
receiver_package_hash 109..144
return_package      145..1587
```

The ranges include each field's 4-byte length prefix.

## 5.7 Envelope

The six fields, in order, are:

```text
0  mailbox_token
1  x25519_kem_ciphertext
2  mlkem_ciphertext
3  encrypted_header
4  ciphertext
5  padding
```

The Envelope object tag is `0x03`. v1.0 fixes the field values and offsets:

| Offset range | Content | Value length |
| --- | --- | ---: |
| 0..8 | object header | 9 |
| 9..44 | mailbox token field | 32 |
| 45..80 | X25519 ciphertext field | 32 |
| 81..1172 | ML-KEM ciphertext field | 1088 |
| 1173..2780 | encrypted Header field | 1604 |
| 2781..4091 | Message ciphertext field | 1307 |
| 4092..4095 | zero-length padding field | 0 |

The complete canonical Envelope is exactly **4096 bytes**.

## 5.8 PayloadFrame

```text
PayloadFrame = u16_be logical_length || application_bytes || zero_padding
```

The frame is exactly **1291 bytes**. The maximum application payload is **1289 bytes**.
The receiver MUST check the length and verify that every byte after the logical payload is
zero. The encrypted Message ciphertext is the 1291-byte frame plus the 16-byte AEAD tag,
for a fixed length of **1307 bytes**.

## 5.9 Canonical Context objects

Domain-separated inputs use object tag `0x7f`, canonical format version `1`, protocol
version `1.0`, and the field count matching the following sequence:

```text
HeaderAD:     domain, major, minor, suite, session, turn, direction, key_id
              8 fields
MessageAD:    domain, major, minor, suite, session, turn, direction, message_id, key_id
              9 fields
HeaderInfo:   same shape as HeaderAD with header domain
MessageInfo:  same shape as MessageAD with message domain
NonceInfo:    domain, session, turn, direction, message_id, key_id
              6 fields
```

The `domain` is a length-prefixed byte string such as
`LinkChat/header-ad/v1`. Numeric values use the same big-endian field encoding as above.

## 5.10 PackageBody and signatures

`PackageBody` contains all public Package fields except `package_auth`. Its canonical
encoding is the input to both the Package Hash and Package Auth domain wrappers. A public
Package MUST NOT encode private keys, Message Keys, Path secrets, or CSPRNG state.

## 5.11 ApplicationMessage, ACK, SACK, and MailboxRecord

ApplicationMessage fields are:

```text
cipher_suite, session_id, turn, direction,
message_id, receiver_key_id, receiver_token, payload
```

`protocol_version` is carried by the 9-byte object header and is not repeated as a
field. Therefore the object has 8 fields after its object header, matching the
`field_count` value.

ACK fields are:

```text
cipher_suite, session_id, turn, direction,
message_id, delivery_status
```

ACK has 6 fields after its object header. Its protocol version is carried by the
object header.

SACK fields are:

```text
cipher_suite, session_id, turn, direction,
base_message_id, bitmap
```

SACK has 6 fields after its object header. Its protocol version is carried by the
object header.

MailboxRecord fields are `token` and a nested canonical Envelope. These objects belong to
the Transport/Control boundary and MUST NOT be interpreted as Application state commands.

## 5.12 Decoder requirements

A decoder MUST re-encode the decoded object and compare the bytes to the input before
accepting it. It MUST reject a supported-looking object with an unsupported tag, format
version, protocol version, field count, field length, semantic enumeration, or trailing
bytes.
