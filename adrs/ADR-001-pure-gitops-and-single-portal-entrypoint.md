# ADR-001: Pure GitOps & Single Portal Entry Point (`app.oenexa.com`)

## Status
Accepted

## Context
The OENEXA ecosystem encompasses multiple high-volume domains (Identity, Wallet, Pay, Exchange, L1 Blockchain, AI Studio, OENGO Commerce). Users previously encountered fragmented entry points and inconsistent navigation, while operational teams relied on manual SSH commands and unversioned cluster changes.

## Decision
1. **Single Entry Point**: All authenticated and exploratory user journeys originate exclusively at `https://app.oenexa.com`. Vanity subdomains (`exchange.oenexa.com`, `trade.oenexa.com`, `wallet.oenexa.com`) issue HTTP 301 permanent redirects to their canonical routes within `app.oenexa.com`.
2. **Pure GitOps**: ArgoCD acts as the sole continuous delivery agent from the `oenexa-devops` repository. Manual `kubectl` execution or SSH modification on production nodes is blocked.

## Consequences
- Unified user experience under a single domain context.
- Full auditability of all production changes via Git commit history.
