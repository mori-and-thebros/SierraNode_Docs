# SierraNode Protocol Specification (v2)

This document is the canonical wire-level reference for SierraNode transport, handshake, session framing, and onion relay setup.

## 1) Outer TCP frame

Each TCP message is framed as:

- `version`: 1 byte, unsigned integer, network byte order, required, must equal `2`.
- `length`: 4 bytes, unsigned integer (`uint32`), big-endian, required, must be `<= MAX_MSG_LEN`.
- `payload`: `length` bytes, opaque bytes, required.

Reject conditions:
- bad `version`
- `length > MAX_MSG_LEN`
- EOF/connection close before full frame is read

## 2) Handshake message (`type = "hello"`)

Handshake payloads are UTF-8 JSON objects, canonicalized for signing (`sort_keys=True`, no extra spaces).

Fields:
- `type`: string, required, must be `"hello"`.
- `proto`: string, required, must equal protocol version (`"2"`).
- `identity_pub_b64`: string, required, base64 of PEM EC public key, curve must be `secp256r1`.
- `ephemeral_pub_b64`: string, required, base64 of PEM EC public key, curve must be `secp256r1`.
- `nonce_b64`: string, required, base64 of 32 random bytes.
- `sig`: string, required, base64 ECDSA-SHA256 signature over canonical JSON of all handshake fields except `sig`.

Validation rules:
- JSON root must be an object.
- all required fields must exist and decode.
- peer identity fingerprint must be in local trusted contacts.
- if an expected peer fingerprint is pinned, it must match exactly.
- handshake timeout is bounded by `HANDSHAKE_TIMEOUT_SEC`.

Reject conditions:
- malformed JSON or malformed base64
- missing required fields
- unsupported protocol version
- untrusted/unknown fingerprint
- invalid or failed signature verification
- wrong EC key type/curve
- invalid nonce length

## 3) Handshake sequence

```
Initiator                                     Responder
---------                                     ---------
build/signed hello (identity + eph + nonce)
send hello  ------------------------------->  recv hello
                                               verify trust + signature + fields
                                send hello  <-------------------------------  build/signed hello
recv hello
verify trust + signature + fields
derive shared ECDH secret + transcript salt (HKDF)
install directional session ciphers
session state -> ESTABLISHED on both sides
```

## 4) Session key derivation

- ECDH shared secret over ephemeral keys.
- HKDF-SHA256 output length: `(2 * SESS_KEY_LEN) + (2 * NONCE_PREFIX_LEN)`.
- Transcript salt includes protocol version, both nonces, both identity fingerprints, and SHA-256 of both ephemeral public keys.
- Directional split:
  - initiator->responder key + nonce prefix
  - responder->initiator key + nonce prefix

## 5) Session encrypted payload

Ciphertext blob format:
- `nonce`: `SESS_NONCE_LEN` bytes = `nonce_prefix (4)` + `counter (8, big-endian)`.
- `ciphertext_tag`: AES-GCM encrypted payload plus 16-byte tag.

Rules:
- per-direction nonce counter must be monotonic; exhaustion is terminal for that session.
- inbound replay window size is `REPLAY_WINDOW`.
- duplicate counters are rejected.
- packets older than replay window boundary are rejected.
- decrypted plaintext must be `<= MAX_PLAINTEXT_MSG_LEN`.

Reject conditions:
- short blob (`< SESS_NONCE_LEN + 16`)
- nonce prefix mismatch
- AES-GCM authentication failure
- replay window violation
- plaintext size violation

## 6) Onion relay setup blob

Tunnel setup frame (inside relay TCP stream):
- `length`: 4 bytes (`uint32`, big-endian), required, must be `<= MAX_MSG_LEN`.
- `blob`: encrypted onion layer bytes, required.

Onion layer decrypted JSON fields:
- `v`: integer, required, must equal `1`.
- `next_ip`: string, required, non-empty IP literal.
- `next_port`: integer, required, `1..65535`.
- `next_is_relay`: boolean, required.
- `data_b64`: string, required, base64 payload for next hop/final target.

Reject conditions:
- malformed blob length framing
- malformed JSON layer
- unsupported onion layer version
- non-boolean `next_is_relay`
- invalid next hop address or port
- allowlist/open-egress policy violation
- setup timeout

## 7) Lifecycle states

Session states:
- `NEW -> HANDSHAKING -> ESTABLISHED -> CLOSING -> CLOSED`
- failure path: `HANDSHAKING/ESTABLISHED -> FAILED -> CLOSING/CLOSED`
- no reactivation after `CLOSED`.

Server/Relay lifecycle:
- `INIT -> RUNNING -> STOPPING -> STOPPED`

