# SierraNode: Absolute - Buyer Brief

## What it is

Proprietary self-hosted secure messaging software with optional onion-style relay mode, delivered as source code plus documentation and validation tooling.

## What problems it solves

- Reduces dependency on third-party chat SaaS for sensitive internal coordination.
- Provides infrastructure-level control for organizations with strict privacy/compliance requirements.
- Supplies an auditable technical package (docs + tests + hardening controls) for enterprise review workflows.

## Who it is for

Best fit:
- Security-focused organizations operating private communications infrastructure.
- Engineering/security teams that can self-host, manage keys, and run internal controls.
- Operators who prefer source-level control over managed SaaS tooling.

Not a fit:
- Buyers expecting SaaS-style managed onboarding by default.
- Non-technical teams without infrastructure ownership.
- Consumer-style messaging expectations.

## Deployment model

- Self-hosted only (private network, cloud VPC, or hybrid).
- Direct peer mode or relay-assisted mode.
- Optional strict runtime controls, metrics, and audit logs.

## Security properties

- Signed ephemeral ECDH handshake.
- Directional AES-GCM session keys.
- Replay-window enforcement.
- Strict framing/version/length checks.
- Contact fingerprint trust pinning.
- Relay egress restrictions by default.

## Why trust it

- Protocol, threat, and traceability docs are included and cross-referenced.
- Reproducible validation tooling exists (unit, smoke, fuzz, chaos, regression).
- Enterprise guardrails and observability are implemented (strict mode, metrics, audit logs).
- Audit/pentest readiness templates are provided for independent assessor workflows.

## Limitations

- No absolute security guarantee.
- Python runtime implies best-effort memory hygiene.
- Buyer owns production hardening, key management, monitoring, and incident response.
- External audit/pentest recommended before sensitive production use.

## What buyer receives

- Full proprietary source package.
- Protocol/threat/traceability/operations docs.
- Validation tools (unit, smoke, fuzz, chaos, security regression).
- Audit/pentest templates and audit bundle script.

## License Availability
- SierraNode: Absolute is issued under a deliberately limited commercial model.
- A maximum of 100 lifetime commercial licenses will ever be granted.
- This cap exists to preserve controlled distribution, review quality, and support discipline.
- License availability is therefore finite and subject to seller approval.

## Price tiers (baseline)

- Evaluation/base License: USD 7,500 equivalent.
- Team / expanded operational License: USD 12,000 equivalent.
- Enterprise License: starts at USD 25,000 equivalent.

## Payment methods

- Primary: crypto transfer.
- Optional (mutual agreement): escrow settlement.

## Contact method

Primary contact: Mori (seller) via direct message channel used for negotiation.

Include in first message: organization name, use case, requested license tier, and preferred settlement method.

## Evaluation flow

1. Read this buyer brief.
2. Review `docs/FAQ_SHEET.md`.
3. Run `python3 scripts/demo_setup.py --local --clean`.
4. Review `docs/TECHNICAL_PROOF_PACK.md`.
5. Request commercial terms and license availability if aligned.
