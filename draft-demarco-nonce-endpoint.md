---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "OAuth 2.0 Nonce Endpoint"
abbrev: "Nonce Endpoint"
category: info

docname: draft-demarco-nonce-endpoint-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: OAuth 2.0 Working Group
keyword:
 - OAuth 2.0
venue:
  group: WG
  type: Working Group
  mail: demarcog83@gmail.com
  github: peppelinux/draft-demarco-nonce-endpoint
  latest: https://example.com/LATEST

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

This document defines the nonce endpoint for the implementations based on OAuth 2.0 [RFC6749], allowing a client to request and obtain server-generated opaque nonces from a server, such as an OAuth 2.0 Authorization Server.

--- middle

# Introduction

This specification defines a method for a client to query an endpoint in a server, such as an OAuth 2.0 authorization server, to request and obtain a new nonce. The nonce is an arbitrary and randomic string used only once. 

OAuth 2.0 deployments may use encryption, using a symmetric or an asymmetric key that is not provided to the client, to carry any confidential information relating to the nonce, such as the origin of the nonce, its time of issuance and expiration and its audiences, within itself; allowing the use of the nonce within infrastructures that do not use shared memory between multiple servers to store and share the issued nonces within their domain.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

# Nonce Endpoint

The Nonce endpoint is an HTTP endpoint that is capable of issuing new nonces. The endpoint MUST be protected by TLS [RFC5246].
A Client requests a nonce by sending an HTTP GET request to the nonce endpoint and the server responds with a JSON object [RFC7159] containing the nonce.

Below is a non normative example of the HTTP Request made by a Client to a nonce endpoint provided by a server.

````
GET /nonce HTTP/1.1
Host: server.example.com
````

The server responds with a JSON object containing the nonce. The response MUST use the HTTP Header Content-Type value set to `application/json` and MUST provide a JSON object containing the member `nonce`. Below a non-normative example of the response of the server, providing a nonce.

````
HTTP/1.1 200 OK
Content-Type: application/json

{
  "nonce": "d2JhY2NhbG91cmVqdWFuZGFt"
}
````
The nonce value MAY use Base64-urlencoded string or a JSON Web Token [RFC7519].

The nonce value MAY be encrypted with an encryption key, if this happens the following rules are met:

- the encryption key MUST NOT be provided to the Client by the server.
- the encryption key MUST NOT be in control of the Client.

# Errors

When a server requires the use of nonces in the request for a specific resource and the Client doesn't provide it in its request,
the server MUST return an HTTP response with the `400` status and an `error` field with the value `"nonce_required"`.

This HTTP response MUST also contain the `Nonce-Endpoint-URI` HTTP header, with the value of the server nonce endpoint
where the Client can obtain a new nonce.

The Client MUST use the HTTPs URL provided in the `Nonce-Endpoint-URI` HTTP header to request a new nonce before
renewing the previous request, in cases where the request can be renewed.

Below is a non-normative example of an error response issued by a server that requires the nonce in the Client request
and provides, at the same time, the nonce endpoint in the form of HTTPs URL:

````
HTTP/1.1 400 Bad Request
Nonce-Endpoint-URI: http://server.example.org/nonce-endpoint

{
  "error": "nonce_required",
  "error_description":
    "Authorization server requires the nonce in the request"
}
````

# Nonce Payload Non-normative Examples

The nonce payload MAY be a JSON object and include several attributes.
Below are provided some non-normative examples of how the decrypted and serialized nonce payload may be represented:

````
{
  "iss": "https://issuer.example.com",
  "iat": 1615908701,
  "exp": 1615995101,
  "source_endpoint": "https://service.example.com/nonce-endpoint",
  "aud": "https://service.example.com/endpoint"
}
````
Please note that the values represented in the previous examples may depend on domain specific requirements and implementation.

# Security Considerations

The nonce endpoint MUST be protected by TLS to prevent eavesdropping and man-in-the-middle attacks.

The server MUST securely generate and store the symmetric key used to encrypt the nonce. The key MUST NOT be provided to the Client.

The robustness of the encryption key plays a crucial role in the security of the nonce endpoint. The following considerations should be taken into account:

1. Key Strength: The symmetric key used for encrypting the nonce should be of sufficient length to resist brute-force attacks. For example, a key length of 256 bits is currently considered to provide a good level of security.

2. Key Management: The symmetric key should be securely managed. It should be securely generated, stored, and revoked. Access to the key should be strictly controlled and limited to authorized entities only.

3. Key Rotation: Regular key rotation is a good practice to mitigate the risk of key compromise. The frequency of key rotation depends on the specific requirements and threat model, but a common practice is to rotate keys frequently.

4. Randomness: The symmetric key, when used, should be generated using a secure random number generator to ensure its randomness. Predictable keys can be easily guessed by attackers.

5. Secure Transmission: If the symmetric key needs to be transmitted over a network, it should be securely transmitted using secure protocols such as TLS.

6. Backup and Recovery: Secure backup and recovery procedures should be in place for the symmetric key. This is to ensure that the key can be recovered in case of loss, while preventing unauthorized access to the backup.

Remember, the security of the nonce endpoint is only as strong as the security of the encryption key. Therefore, proper key management practices are essential.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
