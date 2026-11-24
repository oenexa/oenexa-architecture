# OENEXA Banking-Grade Custody & Trading Platform
## Complete Master Architecture Specification (Phases 1 through 6)

```
   ┌────────────────────────────────────────────────────────────────────────┐
   │                               OENEXAHUB                                │
   │           Unified Institutional Portal & Sovereign Control Plane       │
   └───────────────────────────────────┬────────────────────────────────────┘
                                       │
     ┌──────────────────┬──────────────┼──────────────┬──────────────────┐
     ▼                  ▼              ▼              ▼                  ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   PHASE 1    │ │   PHASE 2    │ │   PHASE 3    │ │   PHASE 4    │ │  PHASE 5 & 6 │
│   CUSTODY    │ │  FINANCIAL   │ │  EXCHANGE &  │ │   TRADING    │ │  SECURITY,   │
│  FOUNDATION  │ │     CORE     │ │   TRADING    │ │  AUTOMATION  │ │COMPLIANCE,OPS│
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
```

---

## 1. PHASE 1: Custody Foundation

### 1.1 Cold / Warm / Hot Storage Tiering Architecture

OENEXA implements a strictly segregated three-tier treasury and custody architecture:

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                           INCOMING DEPOSITS                             │
  └────────────────────────────────────┬────────────────────────────────────┘
                                       │
                                       ▼
  ┌─────────────────────────────────────────────────────────────────────────┐
  │ Tier 1: HOT WALLET (Max 3-5% of Total Reserves)                         │
  │ • Automated intraday withdrawals (< $25,000 equivalent)                 │
  │ • 2-of-3 MPC Threshold Signatures (Co-signer nodes + HSM)               │
  │ • Strict rate-limiting & automated rebalancing thresholds              │
  └──────────────────┬─────────────────────────────────▲────────────────────┘
                     │                                 │
     Sweep Excess    │                                 │ Timelocked Refill
                     ▼                                 │
  ┌────────────────────────────────────────────────────┴────────────────────┐
  │ Tier 2: WARM TREASURY (10-15% of Reserves)                              │
  │ • Batch withdrawals & liquidity replenishment                           │
  │ • 3-of-5 Multisig (FIPS 140-3 Level 4 Cloud HSM + Multi-Party Auth)    │
  │ • Mandatory 4-hour timelock on all outbound allocations                 │
  └──────────────────┬──────────────────────────────────────────────────────┘
                     │
     Deep Vaulting   │
                     ▼
  ┌─────────────────────────────────────────────────────────────────────────┐
  │ Tier 3: COLD AIR-GAPPED VAULT (80-85% of Reserves)                      │
  │ • Institutional core reserve backing                                    │
  │ • Fully air-gapped hardware ceremony (Isolated HSMs in Faraday cages)   │
  │ • M-of-N Sharded Shamir Secret Sharing across geodistributed vaults     │
  │ • Minimum 24-hour timelock + Four-Eyes Executive Quorum (C-level auth)  │
  └─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Key Management Architecture (HSM / MPC Threshold Integration)
- **MPC Scheme:** Threshold Ed25519 (RFC 8032) and ECDSA (secp256k1) using Lindell'17 / Gennaro-Goldfeder (GG20) with proactive secret resharing. No full private key ever exists in memory, disk, or network at any instant in time.
- **Hardware Security Modules (HSM):** FIPS 140-3 Level 4 certified hardware vaults handling key generation, attestation, and signature verification.
- **Zero-Trust Device Binding:** Hardware-backed WebAuthn / FIDO2 Passkeys with origin binding, counter validation (clone detection), and PRF key derivation.

### 1.3 Deposit & Withdrawal Workflow with Full Audit Logging
- **Deposit Ingestion:** `BlockchainDepositListener` observes mempool and confirmed blocks across supported chains. Upon required confirmation depth (e.g. 12 blocks for Ethereum/EVM, 1 for Solana, 3 for Oenexa L1), emits CloudEvent `com.oenexa.wallet.deposit.confirmed.v1`.
- **Withdrawal Pipeline:**
  1. User initiates withdrawal via FIDO2 passkey challenge.
  2. Device risk analysis checks IP velocity (impossible travel), structuring patterns, and OFAC sanctions lists.
  3. Pre-flight ledger reservation creates a timelocked hold in `DoubleEntryLedgerService`.
  4. MPC threshold signature ceremony signs the transaction payload.
  5. Transaction is broadcast to network; confirmation triggers settlement in `DoubleEntryLedgerService` and emits immutable event.

---

## 2. PHASE 2: Financial Core & Double-Entry Accounting

### 2.1 The Invariant Rules of Accounting
1. **Never Update Balances Directly:** Mutating a balance column directly is strictly prohibited at the database, service, and API layers. Balances are derived projections of immutable journal entries.
2. **Double-Entry Equation:** Every single financial event must consist of at least one debit and one credit such that:
   $$\sum \text{Debits} \equiv \sum \text{Credits}$$
3. **Transaction Lineage:** Every journal entry must reference a verified external event ID (e.g. blockchain tx hash, matching engine trade ID, bank deposit wire ref).

### 2.2 Account Chart & Topology
- **Assets (`1000-1999`):** `1010` Hot Wallet Vault, `1020` Warm Treasury, `1030` Cold Vault, `1040` Exchange Clearing Account.
- **Liabilities (`2000-2999`):** `2010` User Spot Trading Deposits, `2020` Institutional Custody Balances, `2030` Pending Withdrawal Holds.
- **Equity (`3000-3999`):** `3010` Exchange Retained Capital, `3020` Treasury Reserves.
- **Revenue (`4000-4999`):** `4010` Trading Fees, `4020` Withdrawal Processing Fees.
- **Expenses (`5000-5999`):** `5010` Blockchain Gas/Network Fees.

### 2.3 Reconciliation Engine
- Continuously audits:
  $$\text{Total On-Chain Hot/Warm/Cold Reserves} \ge \text{Sum of All User Liabilities}$$
- Publishes real-time Proof of Reserves (PoR) with Merkle Solvency Sum Trees.

---

## 3. PHASE 3: Exchange & Trading Core

### 3.1 Order Management System (OMS) & Pre-Trade Risk
- **Order Lifecycle:** `NEW` $\to$ `VALIDATED` $\to$ `RISK_CHECKED` $\to$ `RESTING` $\to$ `FILLED` / `PARTIALLY_FILLED` / `CANCELED`.
- **Pre-Trade Risk Collars:**
  - Balance sufficiency check (ensuring available unencumbered funds).
  - Maximum order nominal size limit.
  - Price collar guard: Rejects limit orders priced outside $\pm 10\%$ of current market mid-price.
  - Rate limiting: Max 200 orders/sec per API key.

### 3.2 Ultra-Low Latency Matching Engine
- In-memory L2/L3 order book with $O(1)$ limit order insertion and cancellation via price-level linked lists and hash indexing.
- Price-Time Priority (FIFO) deterministic trade matching.
- Latency Benchmark: P99 $< 50\mu\text{s}$, throughput $> 50,000$ trades/sec.

### 3.3 Post-Trade Settlement Bridge
- Matches generated by the matching engine trigger double-entry journal postings in real time:
  - Buyer: Debit `User Spot Balance (Asset)`, Credit `User Spot Balance (Quote)`.
  - Seller: Debit `User Spot Balance (Quote)`, Credit `User Spot Balance (Asset)`.
  - Fee: Debit `User Spot Balance (Quote)`, Credit `Exchange Fee Revenue`.

---

## 4. PHASE 4: Trading Automation & Automated Risk Controls

### 4.1 Controlled Automated Strategy Engine
Enables institutional algorithmic market making, TWAP/VWAP execution, and portfolio rebalancing under strict automated risk collars.

```
                    TRADING STRATEGY
                           │
                           ▼
              ┌─────────────────────────┐
              │   AUTOMATED RISK GUARD  │
              └────────────┬────────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
      Position Limit  Daily Loss   Volatility
       Check (Max)    Cap (< 5%)   Circuit Breaker
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Pass All Checks?
                     │           │
                    YES          NO ──► REJECT & NOTIFY
                     │
                     ▼
          Matching Engine Order Book
```

### 4.2 Comprehensive Automated Risk Controls
1. **Position Limits:** Hard ceiling on net asset exposure per trading pair.
2. **Daily Loss Limits:** Automated calculation of rolling 24-hour realized + unrealized loss. If loss exceeds configured threshold (e.g. 5% of portfolio equity), all automated strategies are paused.
3. **Gross Exposure Limits:** Restricts total active nominal commitment across open orders and resting inventory.
4. **Volatility Circuit Breaker:** Detects price dislocations $> 5\%$ within a 60-second window, tripping the circuit breaker and canceling active resting quotes to prevent toxic order flow adverse selection.
5. **Emergency Kill-Switch (Panic Stop):** Programmatic and physical panic button that:
   - Instantly revokes automated order submission tokens.
   - Cancels 100% of open resting orders across all order books within 5 milliseconds.
   - Locks trading to read-only mode until executive sign-off.

### 4.3 Backtesting Framework
- Replays historical tick-level L2 market data.
- Computes PnL, Maximum Drawdown (MDD), Sharpe Ratio, Sortino Ratio, and Win/Loss ratios under simulated slippage and exchange fees.

---

## 5. PHASE 5: Security Operations & Regulatory Compliance

### 5.1 SIEM & Tamper-Evident Security Operations
- OpenTelemetry and Grafana Loki capture structured security events.
- Audit records serialized via RFC 8785 Canonical JSON and hashed with SHA-256 into immutable Merkle trees.

### 5.2 Real-Time Fraud & AML Monitoring
- **Impossible Travel:** Calculates geovelocity ($\text{distance} / \text{time}$) between consecutive API or WebAuthn sessions; flags sessions with velocity $> 900\text{ km/h}$.
- **Structuring Detection:** Identifies Smurfing patterns (multiple transactions just below statutory reporting thresholds, e.g. $9,900).
- **OFAC Sanctions Filter:** Real-time screening of all deposit, withdrawal, and counterparty addresses against SDN and FATF blacklists.
- **FATF Travel Rule (IVMS 101):** Interoperable messaging exchanging originator and beneficiary VASP identities for transfers exceeding $1,000 threshold.

---

## 6. PHASE 6: Operations & Disaster Recovery

### 6.1 Point-in-Time Recovery (PITR) & High Availability
- **PostgreSQL 17 Primary + Standby:** Streaming replication with continuous WAL archiving via WAL-G to S3/MinIO.
- **Automated PITR Drills:** Regular drills on production infrastructure with verified Recovery Time Objective (RTO) $< 10\text{ seconds}$ and Recovery Point Objective (RPO) $= 0\text{ seconds}$ (zero data loss).

### 6.2 Service Level Objectives (SLOs)
- **API Availability:** $\ge 99.95\%$
- **Matching Engine Availability:** $\ge 99.99\%$
- **API P95 Latency:** $\le 300\text{ms}$
- **Matching Engine P99 Latency:** $\le 50\mu\text{s}$
- **Error Rate:** $\le 0.1\%$
