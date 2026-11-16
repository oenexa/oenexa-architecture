# ADR-004: Pure Autonomous Polyrepo Isolation, Database-per-Service & API-Only Communication

## Status
Accepted (Mandatory for all Developers and AI Agents)

## Context
As the Oenexa ecosystem scales across 30 repositories spanning Layer-1 Blockchain, Multi-Agent AI Swarms, Institutional Trading, and Quick Commerce (OenGo), past development introduced architectural couplings:
1. Orchestrator and stray files were placed in the parent workspace directory (`oenexarepo`).
2. Relative file system path dependencies (`file:../oenexa-platform`) were referenced in `package.json`, breaking isolated Docker container builds.
3. Database instances (`oenexa-postgres`) were shared between unrelated domains (`oenexa-ai-core` and `oengo-api`).

## Decision
1. **Workspace Root Cleanliness:**
   The `oenexarepo` directory is strictly a container for cloned Git polyrepos. No repository-specific files, shared build artifacts, or duplicate markdown files may exist at the workspace root.
2. **Zero Cross-Repo File Dependencies:**
   No repository may reference another repository via local file paths (`file:../`, relative imports). Each repository must be self-contained.
3. **Strict Database-per-Service Isolation:**
   Every microservice owns a dedicated, physically isolated database container, storage volume, and credentials (e.g. `oenexa-ai-postgres` for AI Core; `oengo-postgres` for Commerce; embedded LevelDB for Blockchain). Direct SQL access across domains is strictly prohibited.
4. **Pure API-Only Inter-Service Communication:**
   All data sharing and remote procedure invocation must occur exclusively across the network via standard protocols (HTTP/REST, Web3 JSON-RPC, or WebSocket).
5. **Standalone Execution per Repo:**
   Every runnable service repository must maintain its own `Dockerfile`, `docker-compose.yml`, `.env.example`, and test suite. Developers must be able to run and test any single repository without cloning or launching other repositories.
6. **Zero-Mock Policy & SRP Line Limit:**
   All tests must run against real domain records without Mockito. All source files must remain under 385 lines.
7. **Dual-Remote Push:**
   All commits must be synchronized to both GitHub (`origin`) and the production VPS (`server` at `62.210.92.60`).

## Consequences
- 100% blast-radius containment: failures in one service cannot impact or lock another service's database.
- Independent, parallel development across human engineers and autonomous AI agents.
- Fast, zero-downtime hot-swap deployments using `docker compose up -d --no-deps <service>`.
