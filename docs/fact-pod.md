# Fact Pod — General Information

A **Fact Pod** is a modular component that allows websites or online services to securely expose **user‑authorized data (“facts”)** to external agents—such as GPT assistants, recommendation engines, or other AI‑powered services—using the **OpenProfile.ai** protocol.

Think of a Fact Pod as a **trusted data provider**: it only delivers structured facts when **both** of the following conditions are met:

- The requesting agent (e.g., a Gateway) is successfully authenticated via **OAuth 2.0**.
- The user has explicitly granted permission for the requested categories of data.

Fact Pods are typically implemented as **plugins or modules** within platforms that already hold user data (for example: e‑commerce stores, productivity applications, or social platforms).

## Responsibilities of a Fact Pod

To be compliant with **OpenProfile.ai**, a Fact Pod must:

- Support [OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html) for exposing metadata about available endpoints and capabilities.
- Implement [OAuth 2.0 Dynamic Client Registration (RFC 7591)](https://datatracker.ietf.org/doc/html/rfc7591) to simplify and standardize client identity management.
- **Authenticate** incoming requests from Gateways using the OAuth 2.0 **`authorization_code`** and **`refresh_token`** flows.
- **Verify user consent** for each requested fact category before returning data.
- Return facts in a structured and interoperable form, using [Schema.org](https://schema.org/) contracts to describe entities and properties.

> ✅ **Security principle:** Fact Pods must never provide data without validated authentication **and** explicit user authorization.

---

# Fact Pod — Endpoints and Contracts

## `.well-known` Files

A **Fact Pod** must expose specific discovery and metadata files inside the [IETF `.well-known` URI space](https://datatracker.ietf.org/doc/html/rfc8615). 

These files make it possible for an **OpenProfile.ai Gateway** or other clients to **automatically discover your Fact Pod’s capabilities, endpoints, and cryptographic information**.

OpenProfile.ai currently requires these two `.well-known` endpoints:

### 1. `.well-known/openprofile.json`

This JSON file contains **self‑describing metadata** about your Fact Pod, including:

| Field                    | Description                                                                                         |
|--------------------------|-----------------------------------------------------------------------------------------------------|
| `openprofile`            | Block, containing OpenProfile.ai metadata like: version.                                            |
| `fact_pod`               | Block, containing Fact Pod information like categories, title, etc in Schema.org notation.          |
| `issuer`                 | Base URL identifying the Fact Pod provider.                                                         |
| `authorization_endpoint` | URL where the user begins the OAuth 2.0 Authorization Code flow.                                    |
| `token_endpoint`         | URL used by clients to exchange an authorization code for tokens, or to refresh tokens.             |
| `registration_endpoint`  | Endpoint for [RFC 7591 Dynamic Client Registration](https://datatracker.ietf.org/doc/html/rfc7591). |
| `jwks_uri`               | Link to the Fact Pod’s JSON Web Key Set, used to verify signatures.                                 |
| `scopes_supported`       | List of available OAuth scopes (maps to fact categories).                                           |

**Example**:

```json
{
  "issuer": "https://domain.com",
  "authorization_endpoint": "https://domain.com/wp-json/openprofile/oauth/authorize",
  "token_endpoint": "https://domain.com/wp-json/openprofile/oauth/access_token",
  "registration_endpoint": "https://domain.com/wp-json/openprofile/oauth/register",
  "jwks_uri": "https://domain.com/.well-known/openprofile-jwks.json",
  "response_types_supported": [
    "code"
  ],
  "grant_types_supported": [
    "authorization_code",
    "refresh_token"
  ],
  "token_endpoint_auth_methods_supported": [
    "client_secret_basic",
    "client_secret_post"
  ],
  "scopes_supported": [
    "facts:category-16",
    "facts:wishlist"
  ]
}
```

### 2. `.well-known/openprofile-jwks.json`

The [RFC 7517 JSON Web Key Set (JWKS)](https://datatracker.ietf.org/doc/html/rfc7517) file contains the **public keys** the Fact Pod uses for signing and/or encryption.  
Gateways use this document to **verify the integrity of JWTs** (such as ID or access tokens) issued by your Fact Pod.

| Field  | Description                                                   |
|--------|---------------------------------------------------------------|
| `keys` | An array of keys available for verifying tokens.              |
| `kty`  | Key type (e.g., `RSA`, `EC`).                                 |
| `e`    | Exponent for the public key (RSA).                            |
| `use`  | Intended use for this key (`sig` for signature verification). |
| `kid`  | Key ID — allows key rotation without confusion.               |
| `alg`  | Algorithm used with the key (`RS256` = RSA SHA‑256).          |
| `n`    | RSA modulus; the main component of the RSA public key.        |

**Example**:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "abc123key",
      "alg": "RS256",
      "n": "v8e4hLbW8X3x_c4lFhWJcZ1vckwRtm5..."
    }
  ]
}
```

---

## OAuth 2.0 Client Registration

The registration_endpoint in a Fact Pod is defined in .well-known/openprofile.json and enables Dynamic Client Registration according to RFC 7591.

