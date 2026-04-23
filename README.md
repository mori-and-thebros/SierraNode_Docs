# SierraNode: Absolute
SierraNode: Absolute is a security-hardened build for a secure, offline-capable, end-to-end encrypted peer-to-peer messenger with optional onion helper primitives. A packaged demo binary is available for evaluation.

## Security hardening highlights

- Ephemeral ECDH handshake authenticated by long-term identity signatures.
- Direction-specific AES-GCM keys and nonce prefixes.
- Replay protection window on inbound encrypted messages.
- Strict framed TCP message length/version checks.
- Partial read handling and socket timeout control.
- Handshake/tunnel setup reads enforce deadline timeouts (anti-slowloris).
- Token-bucket accept-rate limiting and worker caps on listener/relay sockets.
- Keystore encrypted at rest with password-protected private key PEM.
- Contact fingerprint verification with constant-time comparison.
- Atomic JSON writes for keystore/contacts.
- Route files and relay key files written atomically with private permissions.
- Route files store relay key file references (not embedded relay keys).
- Legacy route files with embedded `key_b64` are blocked by default (opt-in only).
- Safer error handling (no raw sensitive exception text shown to peers; malformed traffic is dropped quietly).
- Best-effort in-memory secret hygiene: mutable buffer zeroization plus OS memory-lock attempts (`mlock`/`VirtualLock`).
- Onion helper logic fixed to use correct next-hop layering.
- Relay egress allowlists to avoid accidental open-proxy behavior (open-egress mode only forwards to global, non-local targets).
- CLI peer output sanitizes terminal control characters to reduce prompt/console spoofing.
- Private file writes now attempt per-platform permission hardening (POSIX mode + Windows ACL tightening).
- Inbound chat file staging now uses private random temp files to prevent transfer-id-based symlink clobbering.
- License activation now rewrites validated JSON with private file permissions (mode `0600` on POSIX).
- Offline license checks now verify Ed25519 signatures with a public key (no embedded signing secret).
- `--enterprise-strict` now enforces stronger file-encryption password length (>= 12 bytes).
- Explicit exception taxonomy for protocol/auth/replay/network/session/tunnel failures.
- Explicit lifecycle state machines for session, listener, and relay runtime.

## Project structure

Top-level layout (major folders/files):

```text
The_App_Named_Absolute_Hardened/
  app/                  # Runtime modules (CLI, session, crypto, file transfer, relay)
  sierranode/           # Packaged license/runtime constants
  tests/                # Unit + adversarial + regression-support tests
  scripts/              # CI smoke, fuzz, soak, docs/pdf, and safety checks
  docs/                 # Protocol, operations, readiness, and sales-facing docs
  artifacts/            # Generated demo/audit/test outputs
  security_regression.py
  generate_license.py
  README.md
  testing_for_newbies_linux.txt
  testing_for_newbies_windows.txt
  testing_for_newbies_mixed_OS.txt
  testin_on_windows_ONLY_one_device.txt
```

For module-level ownership and flow, see `docs/CODEBASE_GUIDE.md`.

## Quick start

From this folder(Where your App Folder located):

```bash
python3 -m app.cli init
```

Show your public fingerprint:

```bash
python3 -m app.cli show-fingerprint
```

Export your public key:

```bash
python3 -m app.cli export-pubkey --out identity_pub.pem
```

Add a trusted contact:

```bash
python3 -m app.cli add-contact <alias> <ip> <port> <peer_pubkey.pem>
```

Listen:

```bash
python3 -m app.cli listen 0.0.0.0 9000
```

If only one contact exists, listener auto-pins to that contact.
If multiple contacts exist, provide `--peer-alias` (recommended) or explicitly allow any trusted contact:

```bash
python3 -m app.cli listen --allow-any-trusted 0.0.0.0 9000
```

Connect:

```bash
python3 -m app.cli connect <peer_ip> <peer_port>
```

`connect` auto-pins the expected peer fingerprint from `contacts.json` using the exact target `<peer_ip>:<peer_port>`.
If multiple contacts share that endpoint, pass `--peer-alias`.

In-chat file transfer is also supported once connected:
- `/sendfile <path> [--name <display_name>]` sends a plain file over the existing secure session.
- `/sendfile-encrypted <path_to_file.gapf> [--name <display_name>]` sends a pre-encrypted `.gapf` file.
- Received files are saved under `incoming_files/` by default.
- Use `/help` in chat to list commands.

Pin to a specific known contact fingerprint while connecting:

```bash
python3 -m app.cli connect --peer-alias <alias> <peer_ip> <peer_port>
```

Tune file-transfer receive behavior on connect/listen:

```bash
python3 -m app.cli connect \
  --incoming-files-dir ./incoming_secure \
  --file-chunk-bytes 98304 \
  --file-max-bytes 16777216 \
  <peer_ip> <peer_port>
```

`--file-chunk-bytes` now has strict overhead-aware validation (chunk base64 + control-frame JSON + protocol prefix),
so values that would overflow a session frame are rejected before transfer starts.

Pin inbound sessions to one trusted contact:

```bash
python3 -m app.cli listen --peer-alias <alias> 0.0.0.0 9000
```

## One-command local demo

For buyer/evaluator sessions, bootstrap a ready two-node localhost demo:

```bash
python3 scripts/demo_setup.py --local --clean
```

This creates demo keystores/contacts and prints exact `listen`/`connect` commands.
It also writes `artifacts/demo_local/DEMO_COMMANDS.txt` for copy/paste execution.

Automation note: the bootstrap uses temporary environment-variable password handling internally for demo setup only.
Do not treat environment-variable password patterns as a production credential strategy.

## Onion relay mode

Generate one relay key per hop:

```bash
python3 -m app.cli gen-relay-key --out relay1_key.b64
python3 -m app.cli gen-relay-key --out relay2_key.b64
```

Create a route file automatically:

```bash
python3 -m app.cli make-route --out route.json \
  --relay 127.0.0.1:7001:relay1_key.b64 \
  --relay 127.0.0.1:7002:relay2_key.b64
```

IPv6 relay specs are also supported (bracketed or plain literal), for example:

```bash
python3 -m app.cli make-route --out route_v6.json \
  --relay [2001:db8::10]:7001:relay1_key.b64
```

Run relays:

```bash
python3 -m app.cli relay 127.0.0.1 7001 --key-file relay1_key.b64 --allow-next 127.0.0.1:7002
python3 -m app.cli relay 127.0.0.1 7002 --key-file relay2_key.b64 --allow-next <peer_ip>:<peer_port>
```

Relay startup now refuses open egress by default. To run with broader egress (not recommended), pass:

```bash
python3 -m app.cli relay 127.0.0.1 7001 --key-file relay1_key.b64 --allow-open-egress
```

When `--allow-open-egress` is enabled, relay forwarding is still restricted to public IP literals (no loopback/private/link-local/multicast targets).
Allowlists (`--allow-next` or `--allowlist`) accept IP literal destinations only.

Connect through the relay chain:

```bash
python3 -m app.cli connect --route route.json <peer_ip> <peer_port>
```

If you must connect with a legacy route file that still embeds `key_b64`, opt in explicitly (not recommended):

```bash
python3 -m app.cli connect --route route.json --allow-legacy-route-keys <peer_ip> <peer_port>
```

## Optional file encryptor

If you want optional local file encryption (independent from chat transport), use:

```bash
python3 -m app.cli file-encrypt ./document.pdf
```

Default output is `<input>.gapf`. You can choose a custom output:

```bash
python3 -m app.cli file-encrypt ./document.pdf --out ./document_encrypted.gapf
```

Inspect encrypted-file metadata:

```bash
python3 -m app.cli file-info ./document.pdf.gapf
```

Decrypt back:

```bash
python3 -m app.cli file-decrypt ./document.pdf.gapf
```

For automation/testing, password can come from `ABS_FILE_PASSWORD`:

```bash
export ABS_FILE_PASSWORD='replace-with-strong-password'
python3 -m app.cli file-encrypt ./document.pdf
```

Optional KDF tuning is available:

```bash
python3 -m app.cli file-encrypt ./document.pdf --scrypt-n 32768 --scrypt-r 8 --scrypt-p 1
```

If Argon2id runtime support is installed, you can opt in:

```bash
python3 -m app.cli file-encrypt ./document.pdf \
  --kdf argon2id \
  --argon2-time-cost 3 \
  --argon2-memory-kib 65536 \
  --argon2-parallelism 1
```

Quick runtime probe for Argon2id support in the current interpreter:

```bash
python3 - <<'PY'
from app.file_encryptor import file_argon2id_available
print(file_argon2id_available())
PY
```

If this prints `False`, install Argon2 support in the same Python runtime you use to run SierraNode.
On mixed Windows/WSL setups, `pip install` in Windows Python does not install into WSL Python automatically.

You can enforce a local size limit when encrypting/decrypting:

```bash
python3 -m app.cli file-encrypt ./document.pdf --max-file-bytes 67108864
python3 -m app.cli file-decrypt ./document.pdf.gapf --max-file-bytes 67108864
```

Current format hardening details:
- `.gapf` files use `AES-256-GCM`.
- Key derivation supports `scrypt` (default, hardened minimum `n=16384`) and optional `argon2id` when runtime support is available.
- If `--kdf argon2id` is selected on a runtime without Argon2id support, encryption fails safely with a clear error.
- File format is explicitly versioned (current writer emits `v=3`; older v1/v2 files remain decryptable).
- File header stores both `salt_b64` and `nonce_b64`; ciphertext body is stored separately.
- Critical metadata (KDF params, salt, nonce, filename, size, hash) is AEAD-bound as authenticated data.
- Decryption applies envelope-size and base64-length guards before expensive decode/derive steps.
- Decryption verifies AEAD authenticity and plaintext SHA-256 integrity before writing output.
- Legacy v1 `.gapf` files remain decryptable for backward compatibility.

## Bundled cryptography (no pip on target)

Bundle the currently-installed `cryptography` package into `app/vendor/<platform_tag>/`:

```bash
python3 -m app.cli bundle-crypto --clean
```

This is platform-specific (OS/CPU/Python version must match target machines).

Quick freshness check (matches bundled version against currently installed package version):

```bash
python3 scripts/check_vendor_crypto_freshness.py
```

## Password input

The CLI prompts securely by default.
For safety, plaintext password arguments are not accepted.

Interactive example:

```bash
python3 -m app.cli init
```

Automation/testing-only shortcut:

```bash
export ABS_APP_PASSWORD='replace-with-strong-password'
python3 -m app.cli init
```

Do not treat environment-variable password patterns as a production credential strategy.

## Enterprise operations flags

For production-style operations and guardrails:

```bash
python3 -m app.cli \
  --enterprise-strict \
  --harden-memory \
  --require-memory-hardening \
  --metrics-file ./ops/metrics.prom \
  --metrics-interval-sec 15 \
  --audit-log ./ops/audit.jsonl \
  listen --peer-alias bob 0.0.0.0 9000
```

What these flags do:
- `--enterprise-strict`: blocks unsafe relay open-egress mode, enforces stronger password minimum (>= 12 bytes), and requires verifiable private file permissions.
- `--harden-memory`: attempts process-level memory hardening (disable core dumps, set non-dumpable mode where available, and `mlockall` best-effort).
- `--require-memory-hardening`: fails startup if no memory-hardening controls can be applied.
- `--metrics-file`: exports Prometheus-style counters/gauges for scraping.
- `--audit-log`: writes structured JSONL security/ops lifecycle events.

## Security regression check

Run the local security regression script after updates/hardening changes:

```bash
python3 security_regression.py
```

Run heavier fuzz/stress checks:

```bash
python3 security_regression.py --strict
```

Generate machine-readable output for CI:

```bash
python3 security_regression.py --json --json-out security_report.json
```

List all checks:

```bash
python3 security_regression.py --list
```

Run only selected category/categories:

```bash
python3 security_regression.py --category crypto
python3 security_regression.py --category transport,crypto
python3 security_regression.py --category handshake
python3 security_regression.py --category dos-sim --strict
```

On Windows PowerShell:

```powershell
python .\security_regression.py
```

The script prints a categorized summary (pass/fail/skip per module) and supports JSON output.

## Lightweight tests (no regression harness)

For quick CI/local confidence without running `security_regression.py`:

```bash
python3 -m compileall -q app
python3 scripts/check_vendor_crypto_freshness.py
python3 scripts/ci_smoke.py
python3 scripts/fuzz_campaign.py --iterations 1500
python3 -m unittest discover -s tests -p "test_*.py"
```

Step-by-step manual network test guides:
- Linux hosts: `testing_for_newbies_linux.txt`
- Windows hosts: `testing_for_newbies_windows.txt`
- Mixed Linux/Windows hosts: `testing_for_newbies_mixed_OS.txt`
- Single Windows machine local lab: `testin_on_windows_ONLY_one_device.txt`

Pre-release reliability gate (recommended):

```bash
python3 -m compileall -q app sierranode scripts tests security_regression.py generate_license.py
python3 -m unittest discover -s tests -p "test_*.py"
python3 security_regression.py --strict
python3 scripts/public_repo_safety_check.py
python3 scripts/chaos_soak.py --duration-sec 12 --nodes 3 --clients 8 \
  --server-drop-rate 0 --server-close-rate 0 --server-corrupt-rate 0 \
  --min-roundtrips 500
```

Treat any non-zero exit as a release blocker.

Optional long-run transport resilience soak:

```bash
python3 scripts/chaos_soak.py --duration-sec 300 --nodes 5 --clients 24 --min-roundtrips 4000
```

This harness is loopback-only (`127.0.0.1`) and is intended for local transport stress checks, not distributed production chaos testing.

Recommended soak profiles:

```bash
# Clean profile (no injected corruption/drops): roundtrip_mismatch should stay 0
python3 scripts/chaos_soak.py --duration-sec 60 --nodes 5 --clients 12 \
  --server-drop-rate 0 --server-close-rate 0 --server-corrupt-rate 0 \
  --min-roundtrips 1500

# Chaos profile (fault injection): roundtrip_mismatch may be non-zero, but process should stay stable
python3 scripts/chaos_soak.py --duration-sec 60 --nodes 5 --clients 12 \
  --server-drop-rate 0.02 --server-close-rate 0.02 --server-corrupt-rate 0.02 \
  --min-roundtrips 800
```

Generate baseline working VLESS/VMess configs and links (Xray format):

```bash
python3 scripts/generate_v2ray_configs.py \
  --host example.com \
  --port 443 \
  --transport ws \
  --security tls \
  --ws-path /ws \
  --cert-file /etc/ssl/private/server.crt \
  --key-file /etc/ssl/private/server.key \
  --out-dir artifacts/v2ray_configs
```

Generated files:
- `xray_server_vless.json`
- `xray_server_vmess.json`
- `xray_client_vless.json`
- `xray_client_vmess.json`
- `links.txt` (redacted share links)
- `links_sensitive.txt` (only when `--emit-secret-links` is set)
- `index.json` (generated target inventory + links metadata)

Generate many server bundles in one run:

```bash
python3 scripts/generate_v2ray_configs.py \
  --server edge-a@example.com:443 \
  --server edge-b@198.51.100.10:443 \
  --server edge-c@[2001:db8::10]:443 \
  --transport ws \
  --security tls \
  --ws-path /ws \
  --cert-file /etc/ssl/private/server.crt \
  --key-file /etc/ssl/private/server.key \
  --out-dir artifacts/v2ray_configs_many
```

Each server gets its own subfolder plus a root `index.json`.

Use a file for larger fleets (`servers.txt` supports comments and blank lines):

```text
# name@host:port
edge-a@example.com:443
edge-b@198.51.100.10:443
edge-c@[2001:db8::10]:443
```

Endpoint checks ("TCP ping") before generation:

```bash
python3 scripts/generate_v2ray_configs.py \
  --servers-file servers.txt \
  --transport tcp \
  --security none \
  --check \
  --only-working \
  --check-timeout-sec 2.0 \
  --check-workers 128 \
  --out-dir artifacts/v2ray_working_only
```

Check outputs:
- `probe_results.json` (reachable/unreachable + latency/error)
- `working_servers.txt` (only reachable endpoints)
- `best_servers.txt` (reachable endpoints sorted by lowest latency)

Continuous auto-refresh mode (re-check + regenerate only live configs):

```bash
python3 scripts/generate_v2ray_configs.py \
  --servers-file servers.txt \
  --transport ws \
  --security tls \
  --ws-path /ws \
  --cert-file /etc/ssl/private/server.crt \
  --key-file /etc/ssl/private/server.key \
  --tls-cert-sha256 '<pin1>' \
  --strict-output-perms \
  --watch \
  --watch-interval-sec 30 \
  --out-dir artifacts/v2ray_live
```

Watch mode outputs:
- `live/` (always current working-only configs)
- `watch_status.json` (latest cycle status)
- `watch_history.jsonl` (append-only history per cycle)
- refreshed `probe_results.json`, `working_servers.txt`, `best_servers.txt` each cycle
- TLS certificate/key paths are validated before generation (existence, parseability, expiry, key/cert match).
- VMess links omit non-standard `sni` by default; opt in with `--vmess-include-sni` only if needed.

Current checks include (130+ checks/assertions grouped into these areas):
- open-egress destination policy (public/global-only behavior),
- IPv4/IPv6 relay spec parsing (including bracketed IPv6),
- strict IP literal parsing, key-file path parsing edge cases, and invalid-spec rejection,
- allowlist parsing (comments/dedup), invalid-entry rejection, and default closed-relay guard,
- relay/server global limiter-cap and stale-bucket pruning behavior,
- relay/server per-IP token-bucket limiter behavior checks,
- route generation format (`key_file` only) and legacy route handling,
- CLI security argument surface (legacy `--password` rejection, no command abbreviation, friendly missing-file errors),
- terminal control-sequence sanitization for peer output,
- frame/plaintext/tunnel size guards, wire-format checks, timeout checks, and protocol-version rejection,
- connection lifecycle checks (idempotent close, callback-failure shutdown, parser arg guards),
- encryptor replay protection, key-length/type guards, destroy-state guards, and counter-overflow guards,
- session payload/trust guards (signature validation, expected fingerprint enforcement, curve checks),
- full-duplex handshake integration checks across socket-paired peers,
- onion-layer unwrap/tamper/field-validation checks,
- key-manager password minimum, signature verification, overwrite behavior, and contact/keystore tamper checks,
- utility helper checks (JSON canonicalization/loading, atomic writes, base64, RNG, zeroize/lock),
- strict-mode fuzz/stress sweeps for parser surfaces, onion layers, transport framing, crypto replay windows, and JSON/base64 surfaces,
- deterministic parser/handshake/onion/crypto fuzz campaign tooling for larger local campaigns,
- chaos/soak harness for multi-node unstable-network transport stress and long-run drift detection.
- `dos-sim` strict sweeps for local-only bad-version bursts, oversize-frame bursts, global-accept saturation, malformed handshake storms, and flood-smoke checks.

## Protocol and traceability docs

- Protocol wire/spec reference: `docs/PROTOCOL_SPEC.md`
- Spec-to-code matrix: `docs/SPEC_TRACEABILITY.md`
- Codebase module guide: `docs/CODEBASE_GUIDE.md`
- Enterprise operations runbook: `docs/ENTERPRISE_OPERATIONS.md`
- VLESS/VMess config automation guide: `docs/V2RAY_CONFIG_AUTOMATION.md`
- Buyer brief: `docs/BUYER_BRIEF.md`
- Technical proof pack: `docs/TECHNICAL_PROOF_PACK.md`
- Technical positioning brief: `docs/TECHNICAL_POSITIONING.md`
- Production readiness review: `docs/PRODUCTION_READINESS_REVIEW.md`
- Plain-English license summary: `docs/LICENSE_SUMMARY_PLAIN_ENGLISH.md`
- Demo quickstart: `docs/DEMO_QUICKSTART.md`
- FAQ sheet: `docs/FAQ_SHEET.md`
- Public GitHub release guide: `docs/GITHUB_PUBLIC_REPO_GUIDE.md`
- Threat model: `docs/THREAT_MODEL.md`
- External audit readiness: `docs/EXTERNAL_AUDIT_READINESS.md`
- Pentest scope template: `docs/PENTEST_SCOPE_TEMPLATE.md`
- Audit report template: `docs/AUDIT_REPORT_TEMPLATE.md`

Regenerate the bundled PDFs from current markdown docs:

```bash
python3 scripts/build_docs_pdfs.py
```

This command now also builds sales/evaluator PDFs:
- `docs/SierraNode_Buyer_Brief.pdf`
- `docs/SierraNode_Technical_Proof_Pack.pdf`
- `docs/SierraNode_License_Summary.pdf`
- `docs/SierraNode_Demo_Quickstart.pdf`
- `docs/SierraNode_FAQ_Sheet.pdf`

Build an external-audit evidence bundle (tests + docs + manifests):

```bash
python3 scripts/build_audit_bundle.py --include-chaos --chaos-duration-sec 20
```

Public GitHub safety pre-push check:

```bash
python3 scripts/public_repo_safety_check.py
```

Never publish live keystores/contacts/identity material or relay keys in a public repo.

## Production security checklist

- Run with strict guardrails for production-like use:
  - `--enterprise-strict`
  - `--harden-memory`
  - `--require-memory-hardening` (when platform policy allows)
- Keep `keystore.json`, `contacts.json`, and license files on private paths/ACLs only.
- Keep relay mode closed by default (`--allow-next` / `--allowlist`); avoid `--allow-open-egress`.
- Rotate relay keys and route files regularly; do not reuse test/demo keys.
- Prefer sending `.gapf` encrypted attachments for sensitive files.
- Run `python3 security_regression.py` after code or config changes.

## Notes

- `contacts.json` must contain the peer fingerprint before handshake succeeds.
- Keep keystore and contacts files private and backed up.
- Do not commit keystore/contact files; the repository intentionally excludes live identity artifacts.
- Zeroization/locking in Python is best-effort only; immutable/runtime-managed copies may still remain in process memory.
- OS memory lock calls can fail under system policy/limits (for example low `RLIMIT_MEMLOCK`), and failures are handled safely.
- Onion route helpers are in `app/onion_router.py`; relay runtime is in `app/onion_tunnel.py`.
