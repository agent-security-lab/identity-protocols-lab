# Lab 01: Authorization code flow with PKCE

**Goal:** run the full flow by hand and see what each parameter does.
**Status:** in progress   **Time spent:**   **Date:** 2026-11-23

## Setup
- Keycloak via `docker compose up` (version: )
- Realm `lab`, user `alice`, public client `lab-public`, PKCE S256 required, redirect URI `http://localhost:8081/callback`

## What I did
Follow `commands.md` and paste what you actually ran.

## What I observed
Decoded ID token and access token (synthetic data only). Which claims surprised you?

## What broke
Mistakes, errors, fixes.

## Attack / failure experiments
- Wrong `code_verifier`:
- Reused authorization code:
- Missing `state`:

## Takeaways
-

## Limitations
-

## References
- RFC 6749, RFC 7636, OIDC Core
