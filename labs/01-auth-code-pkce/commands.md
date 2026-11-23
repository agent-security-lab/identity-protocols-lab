# Lab 01 commands (bash; on Windows use WSL or Git Bash)

## 0. One-time Keycloak setup (admin console, http://localhost:8080)
1. Create realm `lab`.
2. Users > Add user `alice`, set a password (turn off "temporary").
3. Clients > Create client: ID `lab-public`, Client authentication OFF, Standard flow ON.
4. Valid redirect URIs: `http://localhost:8081/callback` (nothing needs to listen there).
5. Advanced > Proof Key for Code Exchange Code Challenge Method = `S256`.

## 1. Make a PKCE verifier and challenge
```
VERIFIER=$(openssl rand -base64 48 | tr -d '=+/' | cut -c1-64)
CHALLENGE=$(printf '%s' "$VERIFIER" | openssl dgst -sha256 -binary | openssl base64 -A | tr '+/' '-_' | tr -d '=')
echo "$VERIFIER"; echo "$CHALLENGE"
```

## 2. Open this URL in your browser, log in as alice
```
http://localhost:8080/realms/lab/protocol/openid-connect/auth?client_id=lab-public&response_type=code&scope=openid&redirect_uri=http://localhost:8081/callback&code_challenge=<CHALLENGE>&code_challenge_method=S256&state=abc123
```
The browser will fail to load `localhost:8081`. That is fine: copy the `code=` value from the address bar. Check that `state` matches.

## 3. Exchange the code for tokens
```
CODE=<paste code>
curl -s -X POST http://localhost:8080/realms/lab/protocol/openid-connect/token \
  -d grant_type=authorization_code \
  -d client_id=lab-public \
  -d code="$CODE" \
  -d redirect_uri=http://localhost:8081/callback \
  -d code_verifier="$VERIFIER"
```
Codes are single-use and short-lived: be quick.

## 4. Decode a token payload
```
python3 -c "import sys,json,base64;p=sys.argv[1].split('.')[1];p+='='*(-len(p)%4);print(json.dumps(json.loads(base64.urlsafe_b64decode(p)),indent=2))" <ACCESS_OR_ID_TOKEN>
```

## 5. Break it (record results in README.md)
- Repeat step 3 with a wrong `code_verifier`.
- Repeat step 3 a second time with the same code.
- Change `redirect_uri` in step 3 or step 2 slightly.
