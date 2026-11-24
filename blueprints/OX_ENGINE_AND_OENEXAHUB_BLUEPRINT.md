# OX Engine & OENEXAHUB Architecture Blueprint
## The Unified Engineering Operating System for OENEXA Polyrepo Ecosystem

```
   ┌────────────────────────────────────────────────────────────────────────┐
   │                               OENEXAHUB                                │
   │                Unified Control Plane & Developer Portal                │
   └───────────────────────────────────┬────────────────────────────────────┘
                                       │
                        ┌──────────────▼──────────────┐
                        │          OX ENGINE          │
                        │ Engineering Operating System│
                        └──────────────┬──────────────┘
                                       │
     ┌──────────────────┬──────────────┼──────────────┬──────────────────┐
     ▼                  ▼              ▼              ▼                  ▼
┌─────────┐       ┌───────────┐  ┌───────────┐  ┌───────────┐      ┌───────────┐
│ ox.repo │       │  ox.cicd  │  │  ox.gov   │  │ox.evidence│      │ ox.deploy │
│Repo Sync│       │7-Gate CI  │  │Law 6 & 7  │  │RFC 8785   │      │Container  │
│Dual-Push│       │Zero-Mock  │  │Four-Eyes  │  │SHA-256    │      │Rollout    │
└─────────┘       └───────────┘  └───────────┘  └───────────┘      └───────────┘
     │                  │              │              │                  │
     ▼                  ▼              ▼              ▼                  ▼
┌─────────┐       ┌───────────┐  ┌───────────┐  ┌───────────┐      ┌───────────┐
│ox.infra │       │  ox.sec   │  │  ox.comp  │  │ ox.agent  │      │  ox.mon   │
│VPS/Drill│       │DPoP/FIDO2 │  │ISO 20022  │  │Autonomous │      │Synthetic  │
│Topology │       │Zero-Trust │  │FATF IVMS  │  │Swarm Sync │      │SLO Health │
└─────────┘       └───────────┘  └───────────┘  └───────────┘      └───────────┘
```

---

## 1. Domain Decomposition

### 1.1 `ox.repo` — Repository Registry & Dual-Push Engine
- Tracks all 25+ ecosystem repositories across `C:\workspace\oenexarepo` and remote hosts.
- Validates dual-push remotes: `origin` (`git@github.com:oenexa/*`) and `server` (`ubuntu@62.210.92.60:/opt/oenexa/repos/*`).
- Detects branch divergence, untracked changes, unpushed commits, and stale HEAD pointers.

### 1.2 `ox.cicd` — 7-Gate Enterprise Pipeline Runner
- Gate 1: Code Style & Static Linter
- Gate 2: Strict Type Check (`tsc --noEmit`, `go vet`, `solhint`)
- Gate 3: Zero-Mock Unit & Component Tests (Law 6: 100% pass rate)
- Gate 4: Security Analysis (Secret scanner, dependency vulnerabilities)
- Gate 5: Production Build (`tsc -b && vite build`, `go build`, `mvn package`)
- Gate 6: Docker Container Verification (Multi-stage build reproducibility)
- Gate 7: Production Readiness & Quality Attestation

### 1.3 `ox.gov` — Governance, Invariants & Auditing
- **Law 7 Validator**: Strict ceiling of $< 385$ lines of code per file across `.ts`, `.tsx`, `.sol`, and `.go`.
- **Four-Eyes Principle**: Validates disjointness of roles: $\text{Creator} \neq \text{Reviewer} \neq \text{Approver}$.
- **Zero Legacy Presence**: Deep scanning for legacy terms (`oengo`) across files, directories, git history, and databases.

### 1.4 `ox.comp` — FinTech & Regulatory Compliance
- Validates ISO 20022 messaging structures (`pain.001` credit transfers and `camt.053` bank-to-customer account statements).
- Validates FATF Travel Rule Recommendation 16 IVMS 101 data structures.
- Evaluates MiCA title II/III regulatory disclosure requirements.

### 1.5 `ox.evidence` — Cryptographic Evidence & Traceability
- Collects test run outputs, benchmark results, and deployment logs.
- Serializes evidence payloads into RFC 8785 Canonical JSON.
- Generates SHA-256 digests and auto-populates `oenexa-evidence/releases/v1.0.0/15-TRACEABILITY-MATRIX.md`.

### 1.6 `ox.infra` & `ox.deploy` — Infrastructure & Zero-Downtime Deployment
- Manages topology on VPS `62.210.92.60` across all services: `oenexa-node`, `oenexa-ai-core`, `oenexa-ai-web`, `oenexa-ui`, `oenexa-explorer`, `oenexa-postgres`, `oenexa-redis`, `oenexa-pgbouncer`, `oenexa-caddy`.
- Executes automated disaster recovery PITR drills with RTO $< 10\text{s}$ and RPO $= 0$.
- Coordinates `--no-cache` builds, rolling restarts, and automated rollback upon unhealthy container probes.

### 1.7 `ox.mon` — Telemetry & Synthetic Health Probes
- Performs synthetic probes against live endpoints:
  - `https://oenexa.com`
  - `https://app.oenexa.com`
  - `https://id.oenexa.com`
  - `https://api.oenexa.com`
  - `https://rpc.oenexa.com`
- Verifies TLS 1.3, HTTP/2, HSTS (`max-age=31536000`), and zero-mock response headers.

### 1.8 `ox.release` — Semantic Versioning & Release Manifest
- Computes next SemVer based on Conventional Commits (`feat:`, `fix:`, `perf:`).
- Generates signed release manifests linking git commit SHA, Docker image digest, and evidence ID.
