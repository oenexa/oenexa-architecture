# ADR-005: OX Engine & OENEXAHUB Unified Engineering Operating System

- **Status**: ACCEPTED
- **Date**: 2026-11-24
- **Author**: OX Agent (Autonomous Principal Engineering & Architecture Agent)
- **Reviewers**: Enterprise Architecture Council, Security Architecture, Chief Technology Officer
- **Scope**: Entire OENEXA Polyrepo Ecosystem (25 Repositories, L1 Node, Exchange, Portal, Infrastructure)

---

## 1. Context and Problem Statement
The OENEXA ecosystem has scaled into an institutional banking-grade polyrepo network spanning 25 distinct repositories, smart contracts, L1 consensus nodes, matching engines, payments gateways, AI agents, and production infrastructure on VPS `62.210.92.60`.

Maintaining institutional compliance, zero-mock testing (Law 6), strict code brevity (< 385 LoC per Law 7), Four-Eyes verification, zero legacy presence, disaster recovery SLAs (RTO < 10s, RPO = 0), and immutable audit evidence requires a dedicated, unified engineering operating system. Individual disconnected scripts lack global state reconciliation, automated drift detection, and centralized agent orchestration.

## 2. Decision: OX Engine & OENEXAHUB
We establish **OX Engine** as the core engineering operating system and **OENEXAHUB** as its unified control portal and API authority.

OX Engine provides 12 foundational engineering domains unified under a single autonomous kernel:
1. **Repository Management (`ox.repo`)**: Multi-repo registry, dual-push synchronization (`origin` and `server`), branch hygiene, and drift detection.
2. **Development Workflows (`ox.workflow`)**: Standardized lifecycle enforcement (Discover -> Plan -> Execute -> Test -> Document -> Evidence -> Release).
3. **CI/CD Pipeline Engine (`ox.cicd`)**: 7-gate enterprise verification runner, deterministic test executor, and artifact attestation.
4. **Infrastructure & Topology (`ox.infra`)**: Production VPS topology management, container health reconciliation, and automated PITR drill execution.
5. **Security & Cryptography (`ox.sec`)**: FIDO2 clone detection, DPoP (RFC 9449) token binding, SPIFFE Zero-Trust guards, and secret scanning.
6. **Governance & Invariants (`ox.gov`)**: Strict Law 7 (< 385 LoC), Law 6 (zero-mock 100% pass rate), Four-Eyes validation, and zero legacy presence enforcement.
7. **Compliance & FinTech (`ox.comp`)**: ISO 20022 (`pain.001`, `camt.053`), FATF Travel Rule (IVMS 101), and MiCA statutory disclosures.
8. **AI Agents (`ox.agent`)**: Autonomous agent registry, capability routing, task dispatching, and audit logging.
9. **Evidence Collection (`ox.evidence`)**: RFC 8785 canonical JSON generation, SHA-256 evidence hashing, and Traceability Matrix synchronization.
10. **Zero-Downtime Deployments (`ox.deploy`)**: Zero-cache container builds, health probes, and automated rollback triggers.
11. **Monitoring & Telemetry (`ox.mon`)**: Synthetic domain health probes (`https://oenexa.com/*`), error budget tracking, and SLA verification.
12. **Release Management (`ox.release`)**: SemVer 2.0.0 calculation, automated changelogs, release manifests, and signed git tagging.

## 3. Consequences
### Positive:
- **Single Pane of Glass**: One unified operating system manages repositories, pipelines, evidence, and deployments.
- **Autonomous Enforcement**: Invariants like Law 7 (< 385 lines) and Four-Eyes verification are evaluated programmatically before any change is finalized.
- **Cryptographic Traceability**: All test runs and deployment artifacts are linked to verifiable SHA-256 evidence entries in `oenexa-evidence`.
- **Zero Legacy Presence**: Continuous scanning guarantees zero reappearance of legacy artifacts.

### Trade-offs:
- Strict gatekeeper: Pull requests or changes that violate Law 7, Law 6, or Four-Eyes principles are blocked until remediated.
- Runtime footprint: Monitored via lightweight TypeScript execution within `oenexa-platform`.
