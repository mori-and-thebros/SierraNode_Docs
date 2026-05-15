# SierraNode FAQ Sheet

## What is SierraNode?

SierraNode: Absolute is a proprietary, self-hosted secure messaging platform with optional relay routing, designed for teams that want direct infrastructure ownership and security-oriented controls.

It is sold as a licensable software product for technically literate buyers, not as a consumer messaging app or managed hosted service.

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

The default offer is self-hosted and self-managed.
Standard website-issued licensing is documentation-led and written-process-driven.
Evaluator support is intentionally limited to fit assessment and technical review acceleration.
Operator support is stronger within the standard written scope because that tier is intended for real internal deployment.
Any broader commercial support expectations, if available, must be handled through separate written agreement.

## What payment methods are accepted?

Current website-issued standard path:
- BTC settlement accepted.
- ETH settlement accepted.
- USD reference pricing converted at the written payment instruction stage.

Card payments, bank payments, wire/ACH, and KYC document collection are not part of the standard website-issued purchase path.
Serious buyers should confirm the selected tier and current payable amount in writing before funds move.

## How should buyers think about Evaluator vs Operator?

- Evaluator is for serious review when source inspection, fit validation, and internal technical scrutiny still need to happen before broader deployment rights make sense.
- Operator is for buyers who already know they want private internal deployment rights and the strongest standard handoff package.
- Qualified Evaluator fees may be credited toward a qualifying Operator upgrade when approved in writing, which helps keep the review path commercially progressive rather than dead-end.

## Is there a refund policy?

Default posture for standard website-issued licensing is that payments are treated as final once private delivery handling begins, unless a separate written exception is explicitly agreed beforehand.

## What limitations and responsibilities should buyers understand?

- Operators remain responsible for secure deployment, monitoring, key custody, and incident response.
- External audit/pentest is recommended before high-assurance production use.
- Runtime-level constraints (including managed-language memory handling limits) are transparently documented.

## Evaluation flow

1. Read `docs/BUYER_BRIEF.md`.
2. Read this FAQ.
3. Review `docs/TECHNICAL_PROOF_PACK.md`.
4. Review `docs/PRODUCTION_READINESS_REVIEW.md` and `docs/ENTERPRISE_OPERATIONS.md`.
5. If aligned, move into the written buyer-pack, inquiry, or standard payment flow.
