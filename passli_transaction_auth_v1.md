# Passli Transaction Auth v1

Version: `1.0.0`  
Status: Draft

This document defines a simple, JSON-over-HTTPS protocol that allows a Relying Party (RP) to request a **user-approved transaction** via the Passli companion app.

The core flow:

1. RP calls the Passli API to **start a transaction challenge**.
2. Passli sends a **push notification** to the user’s trusted device.
3. User **approves or denies** in the Passli app (optionally with Face ID).
4. Passli calls an RP-registered **callback URL** with a **signed JWT** that contains the result.
5. RP verifies the JWT and proceeds accordingly.

---

## 1. Terminology

- **RP** – Relying Party (your website / service).
- **User** – End user associated with an RP account.
- **Device** – Passli-registered device with the companion app.
- **Challenge** – A specific transaction authorization request.
- **Challenge ID** – Unique identifier for a challenge.
- **Client ID** – Stable identifier for an RP.
- **Callback URL** – HTTPS endpoint on the RP receiving the final JWT.

---

## 2. Cryptography

Passli signs transaction results using:

- **Algorithm**: `ES256`
- **Key type**: EC P-256
- **JWKS endpoint**: `https://id.passli.net/.well-known/jwks.json`

---

## 3. RP Onboarding

RPs must register:

- `client_id`
- client authentication method
- one or more callback URLs

Callback URLs must be pre-registered.

---

## 4. API Overview

Base URL: `https://id.passli.net`

Endpoints:

- `POST /v1/tx/start` – Start a challenge.
- `GET  /v1/tx/status` – Poll challenge.
- Callback from Passli → RP with JWT.

---

## 5. Start Transaction

### Request

`POST /v1/tx/start`

```json
{
  "client_id": "rp_1234",
  "user_id": "user_abc",
  "auth_type": "transaction_sign",
  "display_text": "Approve $250.00 payment to Example, Inc.",
  "callback_url": "https://rp.example.com/passli/callback",
  "nonce": "d7f4a5...",
  "ttl_seconds": 120,
  "tx_metadata": {
    "amount": "250.00",
    "currency": "USD",
    "merchant": "Example, Inc."
  }
}
```

### Response

```json
{
  "challenge_id": "ch_9f83bc...",
  "status": "pending",
  "expires_at": 1732212345
}
```

---

## 6. Callback From Passli → RP

Passli POSTs:

```json
{
  "jwt": "<header>.<payload>.<signature>"
}
```

### JWT Payload Example

```json
{
  "iss": "https://id.passli.net",
  "sub": "user_abc",
  "aud": "rp_1234",
  "iat": 1732212200,
  "exp": 1732212320,
  "jti": "ch_9f83bc...",

  "result": "approved",
  "challenge_id": "ch_9f83bc...",
  "nonce": "d7f4a5...",
  "auth_type": "transaction_sign",

  "device_id": "dev_001122",
  "rp_display_name": "Example Store",
  "tx_hash": "4a0f9c..."
}
```

---

## 7. Polling Status (Optional)

`GET /v1/tx/status?client_id=...&challenge_id=...`

Returns:

```json
{
  "challenge_id": "ch_9f83bc...",
  "status": "approved",
  "result_jwt": "<jwt or null>"
}
```

---

## 8. Security Considerations

- Callback URLs must be pre-registered.
- Nonce binds transaction to RP session.
- `jti` prevents replay.
- Short token validity recommended.
- Client authentication required.

---

## 9. Example RP Verification Code

### Node.js Example

```ts
import { createRemoteJWKSet, jwtVerify } from "jose";

const PASSLI_ISSUER = "https://id.passli.net";
const jwks = createRemoteJWKSet(new URL("https://id.passli.net/.well-known/jwks.json"));

export async function verifyPassliTxJWT(token, clientId) {
  const { payload } = await jwtVerify(token, jwks, {
    issuer: PASSLI_ISSUER,
    audience: clientId,
  });

  if (!["approved", "denied", "expired"].includes(payload.result)) {
    throw new Error("Unexpected result");
  }

  return payload;
}
```

### Python Example

```python
import jwt
from jwt import PyJWKClient

PASSLI_ISSUER = "https://id.passli.net"
JWKS_URL = PASSLI_ISSUER + "/.well-known/jwks.json"

def verify_passli_jwt(token, client_id):
    jwks_client = PyJWKClient(JWKS_URL)
    signing_key = jwks_client.get_signing_key_from_jwt(token)

    payload = jwt.decode(
        token,
        signing_key.key,
        algorithms=["ES256"],
        issuer=PASSLI_ISSUER,
        audience=client_id,
    )

    if payload["result"] not in ["approved", "denied", "expired"]:
        raise Exception("Invalid result")

    return payload
```

---

## 10. Versioning

This is `Passli Transaction Auth v1`.  
Breaking changes will result in `/v2/...` endpoints or `tx_ver` claim changes.

