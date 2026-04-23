# Enterprise Operations Runbook

This runbook describes how to operate SierraNode with stronger production guardrails.

## 1) Recommended runtime flags

Use strict mode and observability outputs:

```bash
python3 -m app.cli \
  --enterprise-strict \
  --harden-memory \
  --require-memory-hardening \
  --metrics-file /var/lib/sierranode/metrics.prom \
  --metrics-interval-sec 15 \
  --audit-log /var/log/sierranode/audit.jsonl \
  listen --peer-alias <trusted_alias> 0.0.0.0 9000
```

Notes:
- `--enterprise-strict` blocks relay open-egress mode and enforces stricter password + file checks.
- `--harden-memory` and `--require-memory-hardening` add process-level memory hardening startup controls.
- run under a dedicated OS user with restricted filesystem permissions.

## 2) Deployment guardrails

- Keep keystore and contacts in private directories only.
- Do not run relay with `--allow-open-egress` in production.
- Pin peers with `--peer-alias` where possible.
- Use process supervision (systemd/nssm/container health checks) for auto-restart.
- Apply resource limits:
  - max open files/sockets,
  - process memory limits,
  - CPU quotas if multi-tenant.
- Enforce log rotation and retention for audit files.

## 3) Edge config automation (VLESS/VMess fleets)

Use the generator to build and continuously refresh client/server bundles for many nodes:

```bash
python3 scripts/generate_v2ray_configs.py \
  --servers-file ./ops/servers.txt \
  --transport ws \
  --security tls \
  --ws-path /ws \
  --check \
  --only-working \
  --check-timeout-sec 2.5 \
  --check-workers 128 \
  --out-dir ./ops/v2ray
```

For continuously changing fleet health, use watch mode:

```bash
python3 scripts/generate_v2ray_configs.py \
  --servers-file ./ops/servers.txt \
  --transport ws \
  --security tls \
  --ws-path /ws \
  --watch \
  --watch-interval-sec 30 \
  --out-dir ./ops/v2ray_live
```

Operational outputs to monitor:
- `probe_results.json`: full current reachability/latency status.
- `working_servers.txt`: currently reachable endpoints only.
- `best_servers.txt`: reachable endpoints sorted by lowest latency first.
- `live/index.json`: machine-readable generated-link/config inventory for current working set.
- `watch_history.jsonl`: cycle-by-cycle trend log for alerting and SLO reporting.

## 4) Metrics you should monitor

Handshake/session health:
- `session_handshake_attempt_total`
- `session_handshake_success_total`
- `session_handshake_failure_total`
- `session_decrypt_failure_total`
- `session_state_transition_total{from,to}`

Listener/relay pressure:
- `server_accept_total`, `server_accept_drop_total{reason}`
- `relay_accept_total`, `relay_accept_drop_total{reason}`
- `relay_reject_total{reason}`
- `relay_dial_success_total`, `relay_dial_failure_total`

CLI/process lifecycle:
- `app_process_start_total`
- `cli_command_success_total{cmd}`
- `cli_command_failure_total{cmd}`
- `memory_hardening_control_total{control,result}`

## 5) Baseline alerts (starting point)

- Handshake failure ratio:
  - alert if `session_handshake_failure_total / session_handshake_attempt_total` exceeds 5% for 10m.
- Decrypt failures:
  - alert on sustained growth in `session_decrypt_failure_total` (possible active probing or replay attempts).
- Rate-limit drops:
  - alert on high `server_accept_drop_total` or `relay_accept_drop_total` spikes.
- Relay dial failures:
  - alert if `relay_dial_failure_total` rapidly increases (upstream connectivity or abuse).

Tune thresholds against your normal traffic profile.

## 6) Audit log handling

Audit events are JSONL. Recommended:
- store on encrypted disk,
- protect with least-privilege ACLs,
- ship to centralized log storage,
- monitor for:
  - `session_handshake_failed`,
  - repeated relay rejects,
  - repeated startup/stop churn.

Do not add plaintext or key material to logs.

## 7) Verification checklist before release

1. `python3 -m compileall -q app tests scripts`
2. `python3 -m unittest discover -s tests -p 'test_*.py'`
3. `python3 scripts/ci_smoke.py`
4. `python3 scripts/check_vendor_crypto_freshness.py`
5. `python3 scripts/fuzz_campaign.py --iterations 1500`
6. `python3 scripts/chaos_soak.py --duration-sec 120 --nodes 3 --clients 12 --min-roundtrips 1000`
7. `python3 scripts/build_audit_bundle.py --include-chaos --chaos-duration-sec 20`
8. `python3 scripts/generate_v2ray_configs.py --host 127.0.0.1 --transport tcp --security none --check --out-dir ./artifacts/v2ray_smoke --overwrite`
9. Review `docs/PROTOCOL_SPEC.md` and `docs/SPEC_TRACEABILITY.md` for drift.
