# RFC 7636: PKCE

- **Problem it solves:** A public client (no secret) can't prove who it is, so an attacker who intercepts the authorization code could redeem it for tokens.
- **Actors:** User, client app, authorization server (Keycloak), and the attacker who can see the redirect.
- **What carries what (tokens, parameters):** `code_challenge` (hash of the secret) goes in the authorization request; `code_verifier` (the secret itself) goes in the token request; the server hashes the verifier and compares.
- **One attack it prevents:** Authorization code interception. A stolen code can't be redeemed without the verifier.
- **One attack it does not prevent:** A compromised client or malware that steals the verifier as well, and theft of an access token after it has been issued.
- **Open questions:** Why does Keycloak check the redirect URI before the verifier? What does `plain` vs `S256` change?