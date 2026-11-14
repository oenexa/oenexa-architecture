# DELIVERABLE 01: TARGET ENTERPRISE ARCHITECTURE & REPOSITORY STRATEGY

**Document Version**: 2.0.0  
**Classification**: OENEXA Enterprise Architecture Standard  
**Authors**: CTO, Chief Architect, Enterprise Architect  
**Governing Mandate**: Single Portal (`app.oenexa.com`), Sovereign Identity (`id.oenexa.com`), Sovereign L1 Blockchain (`rpc.oenexa.com`).

---

## 1. Executive Mission & Architectural Foundations

OENEXA is designed as a poly-sovereign global ecosystem uniting high-performance institutional fintech, decentralized blockchain custody, real-time commerce, and generative multi-agent AI into a unified customer journey.

### 1.1 First Principles
1. **Single User Entry Point (`app.oenexa.com`)**: All end users experience a cohesive single-pane interface regardless of product line. Subdomains (`exchange.oenexa.com`, `wallet.oenexa.com`) permanently 301-redirect to canonical paths.
2. **Sovereign Decentralized Identity (`id.oenexa.com`)**: OpenID Connect Discovery (RFC 8414) and OAuth 2.0 PKCE (RFC 7636). Zero `localStorage` token storage. State stored in volatile memory with short-lived rotating JWTs.
3. **Domain Sovereignty & Zero Database Sharing**: No microservice or application shares a database or schema. Cross-boundary interaction occurs strictly through versioned asynchronous CloudEvents (RFC specification) or synchronous HTTP/2 REST / gRPC contracts.
4. **Pure GitOps (No Manual Production Changes)**: All infrastructure, manifests, and policies exist in version-controlled repositories (`oenexa-infra`, `oenexa-devops`, `oenexa-security`). Production modifications occur solely via automated reconcilers (ArgoCD, Terraform Cloud).

---

## 2. Deliverable 1: Target Enterprise Architecture Diagram

```mermaid
graph TD
    subgraph Edge Layer [Global Edge & Ingress: 62.210.92.60]
        CF[Cloudflare Managed DNS & DDoS Mitigation]
        Caddy[Caddy Edge Gateway & Automated ACME TLS]
    end

    subgraph Client Experience Layer [Universal Portal]
        UI[oenexa-ui: app.oenexa.com]
        PortalRoutes[Routes: /dashboard, /exchange, /wallet, /pay, /blockchain, /ai, /oengo, /settings]
    end

    subgraph Shared Platform Layer [oenexa-platform]
        Tokens[@oenexa/design-tokens]
        Shell[@oenexa/ui-shell]
        AuthSDK[@oenexa/auth-sdk]
        EventSchemas[@oenexa/event-schemas]
        Contracts[@oenexa/api-contracts]
    end

    subgraph Sovereign Identity Domain [oenexa-identity]
        IAM[id.oenexa.com / Spring Boot SSO Core]
        JWKS[RSA RS256 JWKS Endpoint]
        OIDC[OIDC Discovery RFC 8414]
    end

    subgraph FinTech & Exchange Domains
        Wallet[oenexa-wallet: Double-entry Ledger & Custody]
        Pay[oenexa-pay: Tri-rail Settlement Engine]
        Exchange[oenexa-exchange: FIFO Orderbook Engine]
        Market[oenexa-market-data: OHLCV & Ticker Aggregator]
        Liquidity[oenexa-liquidity: xy=k Constant Product AMM]
    end

    subgraph Sovereign Blockchain Layer
        Node[oenexa-node: G1 Native Chain RPC rpc.oenexa.com]
        Validator[oenexa-validator: PoS Slashing & Consensus]
        Explorer[oenexa-explorer: Explorer GUI]
        ContractsL1[oenexa-contracts: 1B ONXC & ZK Shielded]
    end

    subgraph AI Intelligence Layer
        RAG[oenexa-rag: 256-dim Vector Embeddings]
        Agents[oenexa-agents: Multi-agent Task Swarm]
        AICore[oenexa-ai-core: Neural Orchestration]
    end

    subgraph Quick-Commerce Ecosystem [OENGO Domain]
        Cust[oengo-customer: Quick Basket & Checkout]
        Merch[oengo-merchant: Kitchen & Catalogue Management]
        Driver[oengo-driver: Courier Dispatch & Milestone Tracking]
        Admin[oengo-admin: Merchant KYB & Dispute Arbitration]
    end

    CF --> Caddy
    Caddy --> UI
    Caddy --> IAM
    Caddy --> Node
    UI --> Tokens & Shell & AuthSDK
    IAM --> JWKS & OIDC
    UI --> Exchange & Wallet & Pay & Node & RAG & Cust
```

---

## 3. Deliverable 2: Repository Strategy

OENEXA adopts a **Federated Multi-Repo Strategy with Poly-Repo Microservices and a Core Platform Monorepo**:

1. **Platform Monorepo (`oenexa-platform`)**: Houses immutable, decoupled shared primitives that have zero business logic. Contains `@oenexa/design-tokens`, `@oenexa/ui-shell`, `@oenexa/auth-sdk`, `@oenexa/event-schemas`, and `@oenexa/api-contracts`.
2. **Business Domain Repositories**: Each business domain maintains independent microservice repositories. They possess isolated build lifecycles, dedicated Dockerfiles, and independent CI/CD release cadence.
3. **Governance & Operations Repositories**: Dedicated repositories for GitOps (`oenexa-devops`), IaC (`oenexa-infra`), Security & Policies (`oenexa-security`), Enterprise Standards (`oenexa-standards`), and Architecture (`oenexa-architecture`).

---

## 4. Deliverable 3: Repository Ownership Matrix

| Repository | Primary Domain Owner | Secondary / SRE Owner | Runtime Technology | Port / Ingress |
| :--- | :--- | :--- | :--- | :--- |
| `oenexa-platform` | Enterprise Platform Team | Architecture Guild | TypeScript / Monorepo | N/A (NPM Package) |
| `oenexa-devops` | Platform & SRE Organization | DevSecOps Lead | Helm / ArgoCD / K8s | GitOps Controller |
| `oenexa-infra` | Cloud Infrastructure Team | Security Architect | Terraform / Cloudflare | IaC Engine |
| `oenexa-security` | Security & Compliance Guild | Enterprise Architect | OPA Rego / Vault HCL | Admission Webhooks |
| `oenexa-standards` | Architecture Review Board | Quality Assurance | Linter / Style Rules | CI Pipeline Gates |
| `oenexa-architecture`| Chief Architect / CTO | Enterprise Architects | Markdown / Architecture | Documentation Portal |
| `oenexa-docs` | Developer Relations | Technical Writers | VitePress / HTML | `docs.oenexa.com` |
| `oenexa-identity` | IAM & Security Guild | FinTech SRE | Spring Boot / Java 21 | `id.oenexa.com:8080` |
| `oenexa-wallet` | FinTech Ledger Guild | Security Architect | Node.js / TypeScript | `:3002` (Internal) |
| `oenexa-pay` | Payments & Banking Guild | Compliance Lead | Node.js / TypeScript | `:3003` (Internal) |
| `oenexa-exchange` | Trading Systems Guild | Low-latency SRE | Node.js / TypeScript | `:3004` (Internal) |
| `oenexa-market-data`| Market Data Engineering | SRE Operations | Node.js / TypeScript | `:3005` (Internal) |
| `oenexa-liquidity` | DeFi Protocol Team | FinTech Architect | Node.js / TypeScript | `:3006` (Internal) |
| `oenexa-node` | Blockchain Protocol Team | Core SRE | Geth Go / EVM Native | `rpc.oenexa.com:8545` |
| `oenexa-validator` | Consensus & Staking Guild | Security Architect | Node.js / TypeScript | `:3007` (Internal) |
| `oenexa-explorer` | Blockchain Frontends | UI Platform Team | React / Vite / Nginx | `explorer.oenexa.com` |
| `oenexa-contracts` | Smart Contract Engineers | Audit & Security Guild | Solidity / Hardhat | On-Chain EVM |
| `oenexa-rag` | AI Engineering Team | Machine Learning Ops | Node.js / TypeScript | `:3008` (Internal) |
| `oenexa-agents` | AI Autonomous Agent Guild| Platform Architect | Node.js / TypeScript | `:3009` (Internal) |
| `oengo-customer` | Commerce Consumer Team | Mobile Platform Team | Node.js / React / PWA | `app.oenexa.com/oengo` |
| `oengo-merchant` | Merchant Solutions Guild| FinTech Architect | Node.js / React | `app.oenexa.com/oengo` |
| `oengo-driver` | Courier & Logistics Guild | Geospatial SRE | Node.js / TypeScript | `:3010` (Internal) |
| `oengo-admin` | Regional Operations Lead | Legal & Compliance | Node.js / TypeScript | `:3011` (Internal) |
| `oenexa-ui` | Core Frontend Organization| Chief Architect | React / TypeScript / Vite | `app.oenexa.com` |

---

## 5. Deliverable 4: Target Repository Tree Structure

```
C:\workspace\oenexarepo\
├── oenexa-platform/          # Shared immutable SDKs, contracts, tokens
│   ├── packages/
│   │   ├── api-contracts/    # RFC 7807 problem details, standard envelopes
│   │   ├── auth-sdk/         # PKCE auth client, memory token store
│   │   ├── design-tokens/    # CSS custom properties, palette, typography
│   │   ├── event-schemas/    # CloudEvents v1.0 standard payloads
│   │   └── ui-shell/         # Top ecosystem nav, breadcrumbs, drawer
│   └── package.json
├── oenexa-devops/            # Platform engineering & GitOps control
│   ├── argocd/               # Root applications & target namespaces
│   ├── helm/                 # Standard enterprise microservice helm chart
│   ├── environments/         # dev / staging / prod value overrides
│   └── monitoring/           # Prometheus alerts, SLO rules, Grafana
├── oenexa-infra/             # IaC Cloud infrastructure
│   ├── modules/              # Cloudflare DNS, Postgres, VPC
│   └── environments/prod/    # Production main.tf, variables, outputs
├── oenexa-security/          # Admission control & least-privilege
│   ├── gatekeeper/           # OPA Rego constraints (no root, strict limits)
│   ├── vault/                # HashiCorp Vault path policies
│   └── playbooks/            # SEV-1 incident escalation and triage
├── oenexa-architecture/      # Master architecture specifications
│   ├── adrs/                 # Architectural Decision Records
│   ├── blueprints/           # Master Governance Blueprint
│   └── deliverables/         # 24 Canonical Deliverable Documents
├── oenexa-standards/         # API guidelines, linter configurations
├── oenexa-identity/          # Sovereign OAuth2 / OIDC IAM service
├── oenexa-wallet/            # Double-entry ledger, quantum note custody
├── oenexa-pay/               # Tri-rail payment gateway (ONXC / SEPA)
├── oenexa-exchange/          # In-memory FIFO orderbook matching engine
├── oenexa-market-data/       # Aggregated OHLCV candle feeds & tickers
├── oenexa-liquidity/         # Automated Market Maker xy=k pool engine
├── oenexa-node/              # Sovereign Layer 1 Geth daemon
├── oenexa-validator/         # Proof-of-Stake consensus & slashing daemon
├── oenexa-explorer/          # Sovereign L1 block explorer interface
├── oenexa-contracts/         # 1B ONXC & Shielded Pool ZK contracts
├── oenexa-rag/               # 256-dim vector retrieval engine
├── oenexa-agents/            # Autonomous multi-agent swarm orchestrator
├── oengo-customer/           # Consumer quick-commerce ordering app
├── oengo-merchant/           # Merchant catalogue & kitchen display app
├── oengo-driver/             # Courier dispatch & delivery milestone app
├── oengo-admin/              # Regional commerce governance & KYB app
└── oenexa-ui/                # Unified ecosystem portal (app.oenexa.com)
```
