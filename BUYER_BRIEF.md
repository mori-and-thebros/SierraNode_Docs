# SierraNode: Absolute - Buyer Brief

## What it is

Proprietary self-hosted secure messaging software with optional onion-style relay mode, delivered as source code plus documentation and validation tooling.

Commercially, SierraNode is offered as a licensable software asset for technically literate buyers who want direct deployment ownership, private operational control, and a documented engineering posture.

## What problems it solves

- Reduces dependency on third-party chat SaaS for sensitive internal coordination.
- Provides infrastructure-level control for organizations with strict privacy/compliance requirements.
- Supplies an auditable technical package (docs + tests + hardening controls) for enterprise review workflows.
- Avoids months of internal prototyping, protocol framing, deployment hardening, and documentation assembly for buyers who need a controllable starting point now.

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

Important commercial note:
- SierraNode is not sold as a managed SaaS product.
- Operator licenses are capped in the initial website release window for support and issuance discipline. Evaluator licenses do not count toward this capped total.
- Custom rights, source discussions, and acquisition-level conversations are handled separately from standard website-issued licenses.

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

## Legal structure

The standard website-issued legal stack is:
- `LICENSE.txt`
- `LICENSE_EVALUATOR_ADDENDUM.txt`
- `LICENSE_OPERATOR_ADDENDUM.txt`
- `CUSTOM_RIGHTS_BASELINE.txt`

Standard website-issued licenses are term-based and valid for the period stated in the issued license record.
Broader rights exist only through separate written terms.

## What buyer receives

- Full proprietary source package appropriate to the purchased tier.
- The applicable legal terms for the purchased path.
- Protocol, threat, traceability, and operations docs.
- Validation tools (unit, smoke, fuzz, chaos, security regression) appropriate to the purchased package.
- Written evaluation and commercial-routing materials aligned with the website buyer-pack flow.
- Operator-oriented deployment material and stronger handoff utility in the Operator path versus the Evaluator path.

## Price tiers (current website-issued standard path)

The pricing is based on licensing specialized encrypted communications software, documentation transfer, proof material, and operational time saved versus building this category internally from zero.

- Evaluator License: USD 3,999 reference pricing; source-backed technical review, controlled testing rights, documentation access, and a structured written evaluation path. Qualified Evaluator fees may be credited toward a qualifying Operator upgrade.
- Operator License: USD 9,999 reference pricing; standard internal/private deployment rights, broader operator-oriented materials, stronger standard handoff value, and priority written support in standard scope.
- Custom / Source / Acquisition discussions: contact us for written commercial terms based on rights scope, negotiated terms, source-access scope, and the required commercial structure.

Operator licenses are capped at 100 total in the initial release window. Evaluator licenses do not count toward this cap.
This cap is intended as an operational quality-control and support-discipline measure, not as artificial scarcity.
Operator buyers whose scope expands beyond the standard path should move into separate custom-rights discussion.

## Payment methods

- Current website-issued license settlement rails: BTC or ETH.
- Prices are presented as USD reference pricing and settled at the applicable written conversion at payment time.
- Card payments, bank payments, wire/ACH, and KYC document collection are not part of the standard website-issued purchase path.
- Broader commercial structures, if any, are handled only through separate written discussion.

## Contact method

Primary contact: @_SierraNode_ on X for written buyer, payment, and rights inquiries.

Include in the first message:
- organization or buyer name,
- intended use case,
- requested license tier or rights scope,
- preferred settlement rail if payment timing is being discussed.

## Evaluation flow

1. Read this buyer brief.
2. Review `docs/FAQ_SHEET.md`.
3. Review `docs/TECHNICAL_PROOF_PACK.md`.
4. Review `docs/PRODUCTION_READINESS_REVIEW.md` and `docs/ENTERPRISE_OPERATIONS.md`.
5. If aligned, move to the written buyer-pack or written inquiry path for licensing, payment, or broader rights discussion.
