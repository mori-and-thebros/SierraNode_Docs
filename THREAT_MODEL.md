# SierraNode Threat Model

This document defines assets, trust boundaries, adversaries, and residual risks for SierraNode.

## 1) Security goals

- Confidentiality of chat payloads in transit.
- Integrity and authenticity of peer identity during session setup.
- Replay resistance for session traffic.
- Controlled relay behavior (no unintended open-proxy operation by default).
- Operational visibility without logging secret material.

## 2) Assets to protect

- Long-term private identity key in keystore.
- Session keys and derived nonce prefixes.
- Plaintext chat payloads.
- Trusted contact fingerprints and peer identity mapping.
- Audit/metrics integrity for incident investigation.

## 3) Trust boundaries

- Local host boundary:
  - Keystore/contacts files and process memory are trusted only as far as host security permits.
- Network boundary:
  - All inbound transport payloads are untrusted.
- Relay boundary:
  - Relay hops are routing infrastructure, not trusted for payload confidentiality.
- Operator boundary:
  - Misconfiguration by a trusted operator is still a threat source.

## 4) Adversary model

In scope adversaries:
- Remote active network attacker (inject, drop, replay, reorder, delay).
- Malicious or compromised relay endpoint.
- Opportunistic scanner/flood source attempting resource exhaustion.
- Untrusted peer attempting malformed handshake/frame abuse.

Partially in scope:
- Local low-privilege user on same host attempting file-read of weakly permissioned artifacts.

Out of scope (current design):
- Fully compromised host/OS or privileged malware.
- Side-channel resistant memory secrecy guarantees beyond best-effort Python controls.
- Post-quantum resistance.

## 5) Attack surfaces

- Framed TCP transport parser.
- Handshake JSON decode, signature verification, and trust lookup.
- Session decrypt path (AEAD failures, replay boundaries).
- Onion layer parsing and relay next-hop policy.
- CLI/operator flags affecting trust and routing posture.
- Observability output pipeline (metric labels, audit fields).

## 6) Implemented controls

Transport/protocol:
- Fixed protocol version framing with maximum frame caps.
- Exact-length reads with timeout/deadline handling.
- Listener/relay accept-rate limits and worker caps.

Handshake/authentication:
- Signed hello payloads with ECDSA over canonical JSON.
- Peer trust anchored to stored contact fingerprints.
- `secp256r1` curve enforcement for identity and ephemeral keys.

Session crypto:
- ECDH + HKDF directional key derivation.
- AES-GCM authenticated encryption.
- Replay window and duplicate counter rejection.
- Destroy-state guards and best-effort secret zeroization/locking.

Relay controls:
- Closed by default unless allowlist or explicit open-egress mode is chosen.
- Open-egress mode still blocks non-global/non-public destinations.
- Next-hop dial concurrency caps and setup timeouts.

Operations:
- Structured audit events and Prometheus-style metrics.
- Enterprise strict mode for stronger runtime guardrails.

## 7) Key assumptions

- Endpoints protect local keystore/password and process integrity.
- Operators provision trusted contacts correctly.
- System clock monotonic behavior is reasonably stable for timeout logic.
- Cryptography backend is correctly installed and not tampered.

## 8) Residual risks

- Python runtime cannot provide hard memory secrecy guarantees.
- DoS remains possible within configured resource limits.
- Complex concurrency paths may still hide rare race conditions.
- Trust-on-configuration model depends on correct contact pinning by operators.
- Metadata leakage (peer IP/port, timing) remains inherent to TCP relay routing.

## 9) Abuse cases and expected behavior

- Malformed frame/version:
  - rejected as protocol error; connection closed.
- Invalid handshake signature or unknown fingerprint:
  - handshake fails; session not established.
- Replay or tampered ciphertext:
  - decrypt/authentication failure; connection closed.
- Relay target outside allowlist/restrictions:
  - tunnel setup denied.
- Accept flood:
  - rate limiter and worker caps trigger controlled drops.

## 10) Recommended enterprise posture

- Always run with `--enterprise-strict`.
- Use peer alias pinning (`--peer-alias`) whenever possible.
- Keep relay in allowlist mode only.
- Monitor handshake/decrypt failure and drop-rate metrics.
- Rotate/restrict audit logs and review for anomalous churn.

