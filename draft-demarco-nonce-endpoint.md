---
title: "OAuth 2.0 Nonce Endpoint"
abbrev: "Nonce Endpoint"
category: info

docname: draft-demarco-nonce-endpoint-latest
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
  github: "peppelinux/draft-demarco-nonce-endpoint"
  latest: "https://peppelinux.github.io/draft-demarco-nonce-endpoint/draft-demarco-nonce-endpoint.html"

author:
 -
    fullname: Giuseppe De Marco
    organization: Independent
    email: demarcog83@gmail.com

normative:
  RFC2119: RFC2119
  RFC5246: RFC5246
  RFC6749: RFC6749
  RFC7159: RFC7159
  RFC7519: RFC7519

informative:
  BCP195: BCP195

--- abstract

This document defines the nonce endpoint for OAuth 2.0 implementations [RFC6749].
The nonce endpoint allows a client to request and obtain server-generated opaque nonces from a server, such as an OAuth 2.0 Authorization Server.

--- middle

# Introduction

This specification defines a method for a client to query a server to request and obtain a new nonce. The nonce is an arbitrary and randomic string used only once.

OAuth 2.0 deployments using this endpoint uses cryptographic mechanisms for the issuance of the nonces to provide confidentiality of the information carried within the nonces. These information can be, for instance: the origin of the nonce, the time of issuance, the time of expiration and its audiences. An encrypted nonce value is used in infrastructures that do not use shared memory between multiple servers to store and share the issued nonces within their domain.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

# Nonce Endpoint

The Nonce endpoint is an HTTP endpoint that is capable of issuing new nonces. This endpoint MUST be protected by TLS [RFC5246].
A Client requesting a nonce to a server uses an HTTP GET request to the server's nonce endpoint.
The server providing a nonce to a Client responds with a JSON object [RFC7159] containing the nonce.

Below is a non normative example of the HTTP Request made by a Client to the nonce endpoint provided by a server.

~~~~ http
GET /nonce HTTP/1.1
Host: server.example.com
~~~~

The server responds with a JSON object containing the nonce. The response MUST use the HTTP Header Content-Type value set to `application/json` and MUST provide a JSON object containing the member `nonce`. Below is a non-normative example of the response given by a server that provides the nonce.

~~~~ http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "nonce": "d2JhY2NhbG91cmVqdWFuZGFt"
}
~~~~

The nonce value MAY use Base64-urlencoded string or a JSON Web Token [RFC7519].

The nonce value MUST be encrypted with an encryption key that:

- MUST NOT be provided to the Client by the server.
- MUST NOT be in control of the Client.

# Errors

When a server requires the use of nonces in the request for a specific resource and the Client doesn't provide it in its request,
the server MUST return an HTTP response with the `400` status and an `error` field with the value `"nonce_required"`.

This HTTP response MUST contain the `Nonce-Endpoint-URI` HTTP header with the value set to the URL corresponding to the server nonce endpoint, where a new nonce can be provided to the Client.

The Client SHOULD use the HTTPs URL provided in the `Nonce-Endpoint-URI` HTTP header to request a new nonce before renewing the previous request.

Below is a non-normative example of an error response issued by a server that requires the nonce in the Client request, the response informs the Client about the nonce endpoint:

~~~~ http
HTTP/1.1 400 Bad Request
Nonce-Endpoint-URI: http://server.example.org/nonce-endpoint

{
  "error": "nonce_required",
  "error_description":
    "Authorization server requires the nonce in the request"
}
~~~~

# Nonce Payload Non-normative Examples

The decrypted nonce payload may be a JSON object and may include any kind of implementation-specific attributes.
Below are provided some non-normative examples about how the decrypted and serialized nonce payload may be represented:

~~~~
{
  "iss": "https://issuer.example.com",
  "iat": 1615908701,
  "exp": 1615995101,
  "source_endpoint": "https://service.example.com/nonce-endpoint",
  "aud": "https://service.example.com/endpoint"
}
~~~~

Please note that the values represented in the previous examples may depend on domain specific requirements and implementation and these MUST NOT be intended as normative.

# Security Considerations

The nonce endpoint MUST be protected by TLS to prevent eavesdropping and man-in-the-middle attacks, therefore the practices defined in [BCP195] must be followed.

The server MUST securely generate and store the symmetric key used to encrypt the nonce. The key MUST NOT be provided to the Client.

The robustness of the encryption key plays a crucial role in the security of the nonce endpoint. The following considerations should be taken into account:

1. Key Strength: The symmetric key used for encrypting the nonce should be of sufficient length to resist brute-force attacks. For example, a key length of 256 bits is currently considered to provide a good level of security.

2. Key Management: The symmetric key should be securely managed. It should be securely generated, stored, and revoked. Access to the key should be strictly controlled and limited to authorized entities only.

3. Key Rotation: Regular key rotation is a good practice to mitigate the risk of key compromise. The frequency of key rotation depends on the specific requirements and threat model, but a common practice is to rotate keys frequently.

4. Randomness: The symmetric key, when used, should be generated using a secure random number generator to ensure its randomness. Predictable keys can be easily guessed by attackers.

5. Secure Transmission: If the symmetric key needs to be transmitted over a network, it should be securely transmitted using secure protocols such as TLS.

6. Backup and Recovery: Secure backup and recovery procedures should be in place for the symmetric key. This is to ensure that the key can be recovered in case of loss, while preventing unauthorized access to the backup.

The security of the nonce endpoint is only as strong as the security of the encryption key. Therefore, proper key management practices are essential.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
