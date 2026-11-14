# DELIVERABLES 17-20: SRE, INCIDENT MANAGEMENT & DISASTER RECOVERY

**Document Version**: 2.0.0  
**Classification**: OENEXA Enterprise Architecture Standard  
**Authors**: Site Reliability Engineer (SRE), Incident Response Engineer, Production Engineer  

---

## 1. Deliverable 17: Incident Management Playbook

### 1.1 Incident Classification & Severity Levels
| Severity | Description | Target Response | Incident Commander |
| :--- | :--- | :--- | :--- |
| **SEV-1 (Critical)** | Core outage (app.oenexa.com down, consensus halted, orderbook frozen) | < 5 minutes | VP of Engineering / CTO |
| **SEV-2 (Major)** | Degraded functionality (single payment rail down, partial AI latency) | < 15 minutes | Staff SRE / Lead Architect |
| **SEV-3 (Minor)** | Non-blocking bug or non-critical cosmetic failure | < 2 business hours | On-Call Product Engineer |

### 1.2 Incident Command Structure
1. **Incident Commander (IC)**: Holds sole decision-making authority over rollbacks, traffic draining, and mitigations.
2. **Operations Lead**: Executes approved actions and infrastructure adjustments.
3. **Communications Lead**: Updates internal status pages and customer portal banners every 15 minutes.
4. **Postmortem Lead**: Authors root-cause analysis (RCA) and blameless retrospective within 48 hours.

---

## 2. Deliverable 18: Disaster Recovery Plan (RTO & RPO Targets)

### 2.1 Recovery Targets
- **Recovery Time Objective (RTO)**: **< 15 minutes** for full ecosystem restoration.
- **Recovery Point Objective (RPO)**: **< 0 seconds (zero data loss)** for financial ledgers; **< 10 seconds** for high-frequency market tickers.

### 2.2 Cold & Warm Recovery Scenarios
- **Complete Datacenter Outage**: Terraform automated multi-region spin-up in secondary cloud region (AWS EU-West -> AWS EU-Central) orchestrated via Terraform Cloud state snapshots.
- **Database Corruption / Ransomware**: Point-in-time recovery (PITR) via continuous WAL streaming to immutable, WORM-locked (Write Once Read Many) Cloud Storage buckets.

---

## 3. Deliverable 19: Observability Strategy (The Three Pillars)

### 3.1 Metrics (Prometheus & OpenTelemetry)
- Standard RED Method (Rate, Errors, Duration) for all HTTP/2 and gRPC microservice interfaces.
- Standard USE Method (Utilization, Saturation, Errors) for host instances, memory buffers, and disk I/O.

### 3.2 Structured Logging
- JSON structured format output directly to `stdout`.
- Every log message contains: `timestamp`, `level`, `service`, `correlationId`, `userId`, `traceId`, `spanId`.

### 3.3 Distributed Tracing (Tempo / Jaeger)
- W3C TraceContext propagation across client headers, Caddy gateway, and all internal microservices.

---

## 4. Deliverable 20: Service Level Objectives (SLOs) & Error Budgets

| Service / Ingress | Target SLI | Monthly SLO | Monthly Error Budget |
| :--- | :--- | :--- | :--- |
| `app.oenexa.com` (Portal) | Successful HTTP 2xx/3xx requests | **99.95%** | 21.6 minutes downtime |
| `id.oenexa.com` (IAM) | Successful token verifications | **99.99%** | 4.32 minutes downtime |
| `rpc.oenexa.com` (L1 Chain) | Valid JSON-RPC block responses | **99.95%** | 21.6 minutes downtime |
| `oenexa-exchange` | Order fill execution < 25ms | **99.9%** | 43.8 minutes degraded |
| `oenexa-pay` | ISO 20022 settlement completions | **99.95%** | 21.6 minutes downtime |
