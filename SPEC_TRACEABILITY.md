# Spec-to-Code Traceability Matrix

This table maps protocol requirements to implementation and tests to reduce spec/code drift.

| Spec Item | Code Entry Point | Validation / Enforcement | Tests |
|---|---|---|---|
| Outer frame `version` + `length` checks | `app/network_manager.py:Connection.receive_one` | Protocol version equality and `MAX_MSG_LEN` cap | `tests/test_cli_listen_errors.py`, `scripts/ci_smoke.py` |
| Exact-frame reads + timeout behavior | `app/network_manager.py:Connection._recv_exact` | deadline/idle timeout handling, closed socket detection | `tests/test_state_machines.py`, `scripts/ci_smoke.py` |
| Handshake JSON structure and required fields | `app/session_manager.py:_decode_json_frame`, `_do_handshake` | object-only JSON, required hello fields, nonce length | `scripts/ci_smoke.py` |
| Handshake signature verification | `app/session_manager.py:_verify_signed_payload` | `sig` presence/base64 decode + ECDSA verify | `scripts/ci_smoke.py` |
| Trusted contact fingerprint enforcement | `app/session_manager.py:_trust_check` | fingerprint lookup + constant-time compare + optional pin | `scripts/ci_smoke.py` |
| EC curve policy (`secp256r1`) | `app/key_manager.py:public_key_fingerprint_hex`, `app/session_manager.py:_load_ec_public_key` | key type and curve checks | `scripts/ci_smoke.py` |
| Directional session key schedule | `app/session_manager.py:_derive_ciphers` | HKDF transcript-derived split keys/prefixes | `scripts/ci_smoke.py` |
| Message AEAD + nonce format | `app/encryptor.py:SendCipher.encrypt`, `ReceiveCipher.decrypt` | AES-GCM, nonce prefix, counter handling | `tests/test_encryptor_adversarial.py` |
| Replay window and duplicate rejection | `app/encryptor.py:ReceiveCipher.decrypt` | replay-window boundary + seen counter set | `tests/test_encryptor_adversarial.py` |
| Cipher lifecycle safety (destroy/use-after-destroy) | `app/encryptor.py:SendCipher.destroy`, `ReceiveCipher.destroy` | `SessionStateError` on post-destroy use | `tests/test_encryptor_adversarial.py` |
| Session state transitions | `app/session_manager.py:_transition_state` | legal transition guard + transition metric | `tests/test_state_machines.py` |
| Listener lifecycle transitions | `app/network_manager.py:Server.start/stop` | `INIT/RUNNING/STOPPING/STOPPED` policy | `tests/test_state_machines.py`, `tests/test_bind_logic.py` |
| Relay lifecycle transitions | `app/onion_tunnel.py:OnionRelayServer.start/stop` | `INIT/RUNNING/STOPPING/STOPPED` policy | `tests/test_state_machines.py`, `tests/test_bind_logic.py` |
| Onion layer boolean strictness | `app/onion_router.py:process_onion_layer` | explicit bool check for `next_is_relay` | `tests/test_onion_flow.py` |
| Relay destination policy | `app/onion_tunnel.py:OnionRelayServer._handle_client` | allowlist checks / restricted open-egress checks | `tests/test_onion_flow.py`, `tests/test_enterprise_guardrails.py` |
| Relay dial concurrency guard | `app/onion_tunnel.py:_acquire_target_dial_slot` | max in-flight dials per destination | `tests/test_onion_flow.py` |
| Enterprise guardrails | `app/cli.py:_enforce_enterprise_guardrails` | strict password length, private path checks, open-egress block | `tests/test_enterprise_guardrails.py` |
| Observability output controls | `app/observability.py` | metrics export + audit JSONL emission | `tests/test_observability.py` |

## Update Rule

When protocol behavior changes:
1. update `docs/PROTOCOL_SPEC.md`,
2. update this matrix row(s),
3. add or update at least one automated test row for the change.

