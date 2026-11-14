# ADR-002: Zero Shared Databases & Strict Domain Ownership

## Status
Accepted

## Context
Cross-service database coupling introduces architectural rot, tight coupling of domain models, concurrent locking issues, and severe security vulnerabilities where compromising one service exposes unrelated customer data.

## Decision
1. **Isolated Databases**: Every business domain (Identity, Wallet, Pay, Exchange, Node, OENGO) owns a physically isolated database instance managed via `oenexa-infra`.
2. **Zero Direct Access**: No service is permitted network or credential access to another domain's database. Cross-domain interactions must occur exclusively through gRPC/REST APIs or asynchronous CNCF CloudEvents over Kafka.

## Consequences
- Independent schema migrations and scalability.
- Enforced blast-radius containment in compliance with SOC 2 and PCI-DSS.
