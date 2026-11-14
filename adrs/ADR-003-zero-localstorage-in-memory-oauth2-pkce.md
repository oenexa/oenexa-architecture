# ADR-003: Zero-LocalStorage Auth Architecture via In-Memory PKCE

## Status
Accepted

## Context
Storing JWT access tokens or refresh tokens in browser `localStorage` or `sessionStorage` exposes user accounts to XSS token exfiltration, violating OWASP ASVS v4.0.3 Level 3 requirements for fintech platforms.

## Decision
1. **In-Memory Access Tokens**: Short-lived (15-minute) JWT access tokens are held exclusively in process memory via `@oenexa/auth-sdk`.
2. **HttpOnly SameSite=Strict Refresh Cookies**: Long-lived refresh tokens are handled solely by the backend identity service via `HttpOnly`, `Secure`, `SameSite=Strict` cookies.
3. **PKCE S256**: All authorization code exchanges mandate cryptographically random code verifiers and SHA-256 challenges.

## Consequences
- Total mitigation of XSS-based token theft.
- Seamless automatic silent token refresh via iframe or background fetch.
