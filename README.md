# Convertible Cranium Technical Documentation Hub

This public hub presents the architecture, evidence boundaries, and deployment concepts behind the WorthWyl / **Convertible Cranium** engineering portfolio. Proprietary implementation repositories are private while acquisition diligence is in progress; public materials explain the system without exposing the source of the core control plane.

## Principal platform

The portfolio’s central technical thesis is a governed control plane for AI systems:

- **Convertible Cranium Core** governs authority transitions and evidence requirements.
- **Convertible Cranium Synapse** defines attestation, risk-score, and intervention contracts.
- **Convertible Cranium Kernel** provides receipt integrity, replay controls, atomic recovery, and governed memory.
- **Convertible Cranium Ultra** integrates the Core and operating-environment surfaces for demonstration and verification.

Together, these components are intended to make AI-enabled actions auditable, bounded, and resistant to malformed, stale, replayed, or unauthorized transitions.

## Published materials

- [Cranium Core case studies](./CRANIUM_CORE_CASE_STUDIES.md)
- [Cranium Core deployment guide](./CRANIUM_CORE_DEPLOYMENT_GUIDE.md)
- [Cranium Core whitepaper](./CRANIUM_CORE_WHITEPAPER.md)
- [Security policy](./SECURITY.md)

## Evidence boundary

Internal verification includes reproducible builds, adversarial authorization scenarios, signed receipt checks, replay and tamper rejection, atomic recovery testing, and machine-readable campaign receipts. These materials should be understood as engineering evidence, not as independent security certification or a financial valuation.

## Acquisition diligence

Qualified reviewers may receive controlled access to private implementation repositories, a frozen commit manifest, a clean-room reproduction command, threat-model documentation, test fixtures, and artifact hashes. Public documentation intentionally avoids publishing proprietary source or sensitive cryptographic fixtures.

## Contact

For technical or acquisition inquiries:

- Email: worthwyl2022@gmail.com
- Web: worth-wyl-media-d9ead881.base44.app
- Phone: 702-602-7543

© 2026 WorthWyl Corp. All rights reserved.

## WorthWyl ownership and review entry point

The Convertible Cranium Ecosystem is presented through **Convertible Cranium Engineering LLC**, the intended engineering, platform, and licensing entity. **WorthWyl Media** remains the creative and publishing branch, and **WorthWyl Foundation** is a separate nonprofit branch. See [`RIGHTS-AND-LICENSING.md`](./RIGHTS-AND-LICENSING.md) for the ownership boundary.

Visual overview: [`assets/cranium-architecture.svg`](./assets/cranium-architecture.svg).

Public review package: [`cranium-portfolio/public-review`](https://github.com/worthwyl2022-cloud/cranium-portfolio/tree/main/public-review).
