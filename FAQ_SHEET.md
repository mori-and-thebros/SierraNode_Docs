# SierraNode FAQ Sheet

## What is SierraNode?

SierraNode: Absolute is a proprietary, self-hosted secure messaging platform with optional relay routing, designed for teams that want direct infrastructure ownership and security-oriented controls.

## Who is it for?

Best fit buyers:
- Technically capable security/engineering teams.
- Private infrastructure operators handling sensitive internal coordination.
- Organizations that require self-hosted communications and source-level control.

Not a fit:
- Teams expecting SaaS/managed hosting by default.
- Non-technical teams without secure infrastructure ownership.
- Buyers seeking consumer-style messaging simplicity.

## How does the security model work?

SierraNode establishes trust through contact fingerprint pinning and signature-verified authenticated handshake flows. Session transport uses directional AEAD keys, replay protection, strict frame validation, and controlled relay behavior.

No system can guarantee absolute security. SierraNode focuses on reducing practical risk through protocol discipline, explicit controls, and clear operational guidance. Where runtime constraints exist, they are documented explicitly rather than hidden.

## Is it self-hosted?

Yes. SierraNode is self-hosted by design.

## What are system requirements?

Baseline:
- Python 3.10+.
- `cryptography` from `requirements.txt`.
- Linux or Windows runtime with standard TCP socket support.
- Operational ownership for key management and host hardening.

## How is trust established between peers?

By explicit contact fingerprint pinning and signature-verified identity checks during session establishment.

## Is it externally audited?

SierraNode is designed to be independently assessable, with audit-readiness materials, protocol documentation, test artifacts, and penetration-test scoping templates included to support third-party review workflows.

## What support is included?

The default offer is self-hosted and self-managed. Optional deployment review, technical clarification, or commercial support terms may be agreed separately for qualifying buyers.

## What payment methods are accepted?

Primary: crypto settlement with verifiable transfer.
Optional: wire/ACH or escrow by written agreement before delivery.

## Is there a refund policy?

Default policy is no refunds after source delivery/escrow completion.

## What limitations and responsibilities should buyers understand?

- Operators remain responsible for secure deployment, monitoring, key custody, and incident response.
- External audit/pentest is recommended before high-assurance production use.
- Runtime-level constraints (including managed-language memory handling limits) are transparently documented.

## Evaluation flow

1. Read `docs/BUYER_BRIEF.md`.
2. Read this FAQ.
3. Run `python3 scripts/demo_setup.py --local --clean` and follow `docs/DEMO_QUICKSTART.md`.
4. Review `docs/TECHNICAL_PROOF_PACK.md`.
5. If aligned, request commercial terms and license availability.
