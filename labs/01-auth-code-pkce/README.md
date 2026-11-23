# Lab 01: Authorization code flow with PKCE

**Goal:** run the full flow by hand and see what each parameter does.
**Status:** in progress   **Time spent:** 45 min  **Date:** 2026-11-23

## Setup
- Keycloak via `docker compose up`(version 26.8.2)
- Realm `lab`, user `alice`, public client `lab-public`, PKCE S256 required, redirect URI `http://localhost:8081/callback`

### 0. One-time Keycloak setup (admin console, http://localhost:8080)
1. Create realm `lab`.
2. Users > Add user `alice`, set a password (turn off "temporary").
3. Clients > Create client: ID `lab-public`, Client authentication OFF, Standard flow ON.
4. Valid redirect URIs: `http://localhost:8081/callback` (nothing needs to listen there).
5. Advanced > Proof Key for Code Exchange Code Challenge Method = `S256`.

## What I did
### 1. Make a PKCE verifier and challenge
```
VERIFIER=$(openssl rand -base64 48 | tr -d '=+/' | cut -c1-64)
CHALLENGE=$(printf '%s' "$VERIFIER" | openssl dgst -sha256 -binary | openssl base64 -A | tr '+/' '-_' | tr -d '=')
echo "$VERIFIER"; echo "$CHALLENGE"
```

### 2. Open this URL in your browser, log in as alice
```
http://localhost:8080/realms/lab/protocol/openid-connect/auth?client_id=lab-public&response_type=code&scope=openid&redirect_uri=http://localhost:8081/callback&code_challenge=<CHALLENGE>&code_challenge_method=S256&state=abc123
```
The browser will fail to load `localhost:8081`. That is fine: copy the `code=` value from the address bar. Check that `state` matches.

### 3. Exchange the code for tokens
```bash
exchange() {  # usage: exchange <code> <redirect_uri> <verifier>
  curl -s -X POST http://localhost:8080/realms/lab/protocol/openid-connect/token \
    -d grant_type=authorization_code \
    -d client_id=lab-public \
    -d code="$1" \
    --data-urlencode redirect_uri="$2" \
    -d code_verifier="$3"
  echo
}

CODE='<paste fresh code>'
exchange "$CODE" "http://localhost:8081/callback" "$VERIFIER"
```
Codes are single-use and short-lived, so use a fresh one for every attempt.

### 4. Decode a token payload
```
python3 -c "import sys,json,base64;p=sys.argv[1].split('.')[1];p+='='*(-len(p)%4);print(json.dumps(json.loads(base64.urlsafe_b64decode(p)),indent=2))" <ACCESS_OR_ID_TOKEN>

OR

use www.jwt.io (fine for test tokens)
```

## What I observed
Decoded ID token and access token (synthetic data only). Which claims surprised you?

## What broke
1. **Typed the placeholder literally.** I ran `CODE=<paste code>`. Bash treated `<` and `>` as redirection and printed a syntax error, so the curl sent an empty code and got `Code not valid`.
2. **Expired and used-up codes.** The access code lifespan is 1 minute by default, and my manual copy-paste was too slow. Failed attempts also burned the code, so retries with the same code never worked. Fix: raise Access Code Lifespan for the lab, and use a fresh code for every attempt.
3. **Confounded experiments.** My first round of tests gave misleading errors (a "wrong verifier" test returned `Incorrect redirect_uri`) because earlier attempts had already consumed the code and the redirect URI was also off. Fix: change one thing per request, with a fresh code each time.
4. **Session.** Not being asked to log in again was expected, because Keycloak keeps a login session cookie.

## Attack / failure experiments

Each test used a fresh authorization code and changed exactly one thing.

| Test | Change | Result | What it shows |
| --- | --- | --- | --- |
| A | Wrong `code_verifier` | `invalid_grant: PKCE verification failed: Invalid code verifier` | A stolen code is useless without the verifier |
| B | Wrong `redirect_uri` (`/callback2`) | `invalid_grant: Incorrect redirect_uri` | The redirect URI must exactly match the one in the authorization request |
| C | Everything correct (control) | Tokens issued | Baseline works |
| D | Reused a code a second time | `invalid_grant: Code not valid` | Codes are single-use |


## References
- RFC 6749, RFC 7636, OIDC Core
