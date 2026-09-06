# Changelog

## v1.0-research - 2026-09-05

- Reorganized the protocol into an English, topic-separated specification suite.
- Added an exact canonical object-header and field-encoding contract.
- Added fixed field offsets for ReceiverPackage, Header, and Envelope.
- Preserved the 4096-byte Envelope, 1291-byte PayloadFrame, 1289-byte maximum payload,
  1307-byte ciphertext, 1588-byte Header, and 1439-byte public Package decisions.
- Recorded the HeaderAD decision that excludes the encrypted `message_id`.
- Kept ACK/SACK in Transport/Control and outside Application state authority.
- Added explicit storage crash, durability, rollback, and cross-process boundaries.
- Added formal-verification, primitive-certificate, conformance, vector, and extension rules.
