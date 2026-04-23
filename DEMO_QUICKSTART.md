# SierraNode Demo (Buyer-Friendly Path)

This demo is designed for technical evaluators and procurement conversations.

## Fast path (recommended)

Run one command:

```bash
python3 scripts/demo_setup.py --local --clean
```

What it does:
- Creates two local demo identities.
- Initializes keystores.
- Exports public keys.
- Adds trusted contacts for both nodes.
- Prints exact listener/connect commands.
- Writes commands to `artifacts/demo_local/DEMO_COMMANDS.txt`.

Typical time to ready state: under 3 minutes on a standard laptop.

## Run the actual demo session

1) Start listener (Terminal A) using printed command.
2) Start connector (Terminal B) using printed command.
3) Enter the demo password when prompted.
4) Exchange test messages both directions.
5) Optional file-transfer demo in chat:
   - `/sendfile ./artifacts/demo_local/DEMO_COMMANDS.txt`
   - `/sendfile-encrypted ./sample_payload.gapf` (if generated with `file-encrypt`).
   - Receiver saves into `incoming_files/` unless overridden by `--incoming-files-dir`.

## Optional strict-control demonstration

Show runtime guardrails and observability:

```bash
python3 -m app.cli \
  --enterprise-strict \
  --harden-memory \
  --require-memory-hardening \
  --metrics-file ./ops/metrics.prom \
  --metrics-interval-sec 15 \
  --audit-log ./ops/audit.jsonl \
  listen --keystore artifacts/demo_local/b_keystore.json --contacts artifacts/demo_local/b_contacts.json --peer-alias alice \
  --incoming-files-dir ./ops/incoming \
  --file-chunk-bytes 98304 \
  --file-max-bytes 16777216 \
  127.0.0.1 19000
```

Inspect outputs:

```bash
tail -n 20 ./ops/audit.jsonl
cat ./ops/metrics.prom
```

## Manual path (engineering deep dive)

If you want to validate each step manually, use the command sequence in `artifacts/demo_local/DEMO_COMMANDS.txt` after bootstrap or follow the longer CLI flow in `README.md`.

## Security note on password automation

Demo bootstrap uses a temporary environment variable internally for non-interactive setup only.

Do not use environment-variable password patterns as a production credential strategy.

## Optional post-demo proof commands

```bash
python3 scripts/ci_smoke.py
python3 security_regression.py
python3 security_regression.py --strict
```
