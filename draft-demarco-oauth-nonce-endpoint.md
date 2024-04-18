---
title: "The Nonce Endpoint"
abbrev: "Nonce Endpoint"
category: info

docname: draft-demarco-oauth-nonce-endpoint-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: "Security"
# workgroup: OAuth 2.0 Working Group
keyword:
 - OAuth 2.0
venue:
#  group: WG
#  type: Working Group
#  mail: demarcog83@gmail.com
  github: "peppelinux/draft-demarco-oauth-nonce-endpoint"
  latest: "https://peppelinux.github.io/draft-demarco-oauth-nonce-endpoint/draft-demarco-oauth-nonce-endpoint.html"

author:
 -
    fullname: Giuseppe De Marco
    organization: Independent
    email: demarcog83@gmail.com
 -
    fullname: Orie Steele
    organization: Transmute
    email: orie@transmute.industries

normative:
  RFC2119: RFC2119
  RFC4086: RFC4086
  RFC5246: RFC5246
  RFC6749: RFC6749
  RFC7159: RFC7159
  RFC7519: RFC7519

informative:
  BCP195: BCP195

--- abstract
This document defines a Nonce Endpoint and details how a Server generates and issues opaque Nonces and how a client can learn about this endpoint to obtain the Nonce.

--- middle

# Introduction

This specification presents a comprehensive guide to the Nonce endpoint. It describes in detail how a client can request and receive a server-generated Nonce, which is a unique, one-time use, opaque string. This document provides in-depth insights into the cryptographic methods used in generating Nonces to protect the confidentiality of the information associated with them. In addition, it serves as a resource for developers and system architects who aim to enhance the scalability, security, and efficiency of their systems.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

**Nonce**:
:  A random or pseudo-random number that is generated for a specific use, typically for cryptographic communication. The Nonce is used to protect against replay attacks by ensuring that a message or data cannot be reused or retransmitted. The term "Nonce" stands for "number used once" and it MUST be unique within some scope.

**Nonce Issuer**:
:  The entity that generates and provides the Nonce.

**Nonce Endpoint**:
:  The HTTP endpoint provided by the Nonce Issuer for the issuance of the Nonces.

# Requirements

The **Nonce Endpoint** MUST satisfy the following requirements:

- Employ TLS for securitung the Endpoint [RFC5246].

The **Nonce** MUST satisfy the following requirements:

- It MUST be opaque to receiving Clients.
- It MUST be made unguessable through the use of cryptographic randomness.
- It MUST be used once.
- It MUST have a short validity period.

The **Nonce Issuer** MUST satisfy the following requirements:

- Generate a unique Nonce for each request to ensure the Nonce Issuer never produces identical Nonces, regardless of whether they occur simultaneously or at different times;
- Encrypt the Nonce with an encryption key that:
  - MUST NOT supplied by the Nonce Issuer to the Client;
  - MUST NOT disclosed by the Nonce Issuer to any external entity beyond its domain.

The **audiences of the Nonces** satisfies the following requirements:

- The servers, within the Nonce Issuer's domain, SHOULD decrypt the Nonce and access its decrypted contents. No other entity might decrypt or know the decrypted contents of the Nonce.

# OAuth Authorization Server Metadata Registration

This specification introduces a new metadata value for use in the OAuth 2.0 Authorization Server Metadata [RFC8414]. This metadata enables OAuth 2.0 clients to discover the Nonce Endpoint of the Authorization Server.

## Nonce Endpoint Metadata

The following metadata parameter is registered by this specification:

- **Name**: `nonce_endpoint`
- **Description**: URL of the Server's Nonce Endpoint. This endpoint is used by the client to obtain a nonce value as specified in this document.
- **Specification Document(s)**: This document.

Clients that utilize the Nonce Endpoint SHOULD first retrieve the Server's metadata as described in [RFC8414] and use the `nonce_endpoint` value to discover the location of the Nonce Endpoint.

The OAuth 2.0 Authorization Server MUST include the `nonce_endpoint` metadata in its OAuth 2.0 Authorization Server Metadata if it supports the Nonce Endpoint functionality described in this specification.

### Example Metadata

Below is an example of how an OAuth 2.0 Authorization Server Metadata might include the `nonce_endpoint`:

~~~~
{
  "issuer": "https://server.example.com",
  "authorization_endpoint": "https://server.example.com/oauth2/authorize",
  "token_endpoint": "https://server.example.com/oauth2/token",
  "nonce_endpoint": "https://server.example.com/oauth2/nonce",
  ...
}
~~~~

# Nonce Request

When a Client needs a Nonce, it sends an HTTP GET request to the Nonce Endpoint.

Below is a non normative example of the HTTP Request made by a Client to the Nonce Endpoint.

~~~~ http
GET /nonce HTTP/1.1
Host: server.example.com
~~~~

Below a sequence diagram represents the proactive approach of obtaining a Nonce from the Nonce endpoint
before making a request to the server, thus avoiding the error case where the server requires a
nonce that the Client did not provide. This approach illustrated below ensures a smoother
interaction flow and enhances the understanding of the Nonce acquisition process.

~~~~
Client                Nonce Endpoint                Server
  |                           |                        |
  |--- GET /nonce ----------->|                        |
  |<-- Nonce -----------------|                        |
  |                           |                        |
  |--- Request with Nonce --->|----------------------->|
  |                           |                        |-- Check - >
  |                           |                        |<-- OK ----|
  |                           |<-----------------------|
  |                           |                        |
~~~~

# Nonce Response

The Nonce Endpoint provides a Nonce to the Client, encapsulated within a JSON object [RFC7159].
The response MUST use the HTTP Header Content-Type value set to `application/json` and MUST provide in the response message a JSON object with the member `nonce`.

Below is a non-normative example of the response given by a Nonce Endpoint:

~~~~ http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "nonce": "d2JhY2NhbG91cmVqdWFuZGFt"
}
~~~~

# Nonce Endpoint Discovery

When a server requires the use of a Nonce in the request for a specific resource and the Client does not provide it in its request,
the server MUST return an HTTP response with the HTTP status code `400` and an `error` field with the value set to `"nonce_required"`.

This response MUST also contain the `Nonce-Endpoint-URI` HTTP header, with the value set to the URL corresponding to the Nonce Endpoint, where the Client SHOULD request and fetch a new Nonce. Once the Nonce is received, the Client MAY renew the request to the server, including the obtained Nonce.

Below is a non-normative example of an error response issued by an server that requires the Nonce in the Client request, the response informs the Client about the Nonce Endpoint where the Nonce should be requested:

~~~~ http
HTTP/1.1 400 Bad Request
Nonce-Endpoint-URI: https://server.example.org/nonce-endpoint

{
  "error": "nonce_required",
  "error_description":
    "Server requires the nonce in the request"
}
~~~~

In cases where, for some reasons, a correctly issued Nonce can no longer be considered valid by the server that receives it, the server MUST return the generic error `"nonce_required"` reporting the same description as `"error_description"`, as if the Nonce had not been received. The cases when an issued Nonce is considered no longer valid MAY be caused by the rotation of the encryption keys, its expiration or other specific conditions internal to an implementation.

# Non-normative Examples of a Nonce Payload

The decrypted Nonce payload MAY use different formats and encodings, according to the different implementation requirements and contain any kind of implementation-specific claims, such as the issuance time, the time of expiration, the audiences and others where needed.

Below are provided some non-normative examples, describing how a decrypted and JSON serialized Nonce payload MAY appear:

~~~~
{
  "jti": "0452767d-549d-4765-bd43-a0bcc2a6659a",
  "iss": "https://server.example.org",
  "iat": 1615908701,
  "exp": 1615995101,
  "source_endpoint": "https://server.example.org/nonce-endpoint",
  "aud": [
    "https://service.example.com/endpoint",
    "https://another.example.com/cb"
  ]
}
~~~~

Please note that the values represented in the previous examples are informative.

In any case, the payload MUST include some unique value (`"jti"` on the example above), typically generated using a pseudo-random number generator with sufficient entropy [RFC4086], to ensure that the encrypted digest (the actual Nonce) is also unique.

# Security Considerations

The Nonce Endpoint MUST be protected by TLS to prevent eavesdropping and man-in-the-middle attacks, therefore the practices defined in [BCP195] should be followed.

The Nonce Issuer MUST securely generate and store the encryption key used to encrypt the Nonce. The robustness of the encryption key plays a crucial role in the security of the Nonce Endpoint. The following considerations MUST be taken into account:

1. **Key Strength**: The cryptographic key used to encrypt the Nonce requires sufficient length to withstand brute-force attacks. A key length of 256 bits has been proposed as a common practice to ensure a minimum level of security.

2. **Key Management**: The cryptographic key requires secure management, which includes secure generation, storage, and revocation. Access to the key necessitates strict control, with access granted only to authorized entities.

3. **Key Rotation**: Regular key rotation is a good practice to mitigate the risk of key compromise. The frequency of key rotation depends on the specific requirements and threat model, but a common practice is to rotate keys frequently.

4. **Randomness**: To assure the randomness of the cryptographic key, it requires the usage of a safe random number generator. Attackers can simply guess predictable keys.

5. **Secure Transmission**: If the cryptographic key needs to be transmitted over a network and within the Nonce Issuer domain, it requires the usage of secure protocols such as TLS.

6. **Backup and Recovery**: Cryptographic keys require secure backup and recovery mechanisms. This ensures that the key can be retrieved in the event of its loss while also prohibiting unauthorised access to the backup.

The security of the Nonce Endpoint is only as strong as the security of the encryption key. Therefore, proper key management practices are essential.

# Considerations about Nonce vs. jti

This section provides some thought about the main differences and scopes of the Nonce in compared to the `jti` claim defined in [RFC7519].

Both `jti` and Nonces are used to prevent replay attacks, however Nonces offer more implementation flexibility and are considered best practice. They can be created and managed stateless (e.g., by issuing the hmac over the current time as the Nonce), as this document outlines.

The main differences between the use of the `jti` and the Nonces can be summarized as follows:

1. **Generation**: Nonces and jti can be are generated both by the server and the client.

2. **Lifetime**: The life span difference between a Nonce and a `jti` is significant. Nonces are kept just until the Client responds, which happens practically immediately after they are obtained, resulting in a very short lifespan. A `jti`, on the other hand, must be stored until the expiration window of its associated JWT expires, which is a substantially longer length than that of a Nonce.

3. **Freshness**: Nonces prevent replay attacks by ensuring that the proof of possession is fresh. On the other hand, `jti` does not guarantee freshness and using client-generated timestamps has problems, even for non-attacking Clients (e.g. devices with incorrect time-zones or daylight saving settings).

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
