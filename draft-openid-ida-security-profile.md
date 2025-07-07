---
title: "Identity Assurance Profile for OpenID Connect"
abbrev: "IDA Profile for OIDC"
category: info

docname: draft-openid-ida-security-profile-latest
submissiontype: "independent"
number:
date:
v: 3
# area: AREA
workgroup: eKYC-IDA
keyword:
 - identity assurance
venue:
  group: eKYC-IDA
  type: Working Group
  mail: openid-specs-ekyc-ida@lists.openid.net
  arch: https://openid.net/wg/ekyc-ida/
  github: "oktadev/openid-ida-oidc-security-profile"
  latest: "https://oktadev.github.io/openid-ida-oidc-security-profile/draft-openid-ida-security-profile.html"

author:
 -
    fullname: Scott Bermon
    organization: Okta
    email: scott.bermon@okta.com
 -
    fullname: Aaron Parecki
    organization: Okta
    url: https://aaronparecki.com

normative:

  OpenID:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore

informative:



--- abstract

This document defines an Identity Assurance Profile for OpenID Connect that establishes security requirements and best practices for a relying party to generate an identity verification request and receive identity assurance from an OpenID provider.

The profile builds upon OpenID Connect Core 1.0 and specifies additional constraints, mandatory features, and security considerations necessary for identity assurance use cases where verified identity attributes are required. This specification leverages OpenID Connect for Identity Assurance (OIDC4IA) mechanisms to enable standardized communication of identity verification requirements and assurance levels between relying parties and identity providers. An extension to the Verified Claims to support fuzzy matching of verified claims is also defined to accommodate minor discrepancies in user-provided data.

--- middle

# Introduction

In today's digital ecosystem, organizations increasingly rely on third-party identity verification providers to establish trust and comply with regulatory requirements. However, the broad integration requirements for identity verification providers have created significant challenges for both relying parties and identity providers. Each provider often implements proprietary interfaces, varying data formats, and inconsistent assurance level indicators, leading to complex integration efforts and limited interoperability.

This specification defines an Identity Assurance Profile for OpenID Connect. The profile establishes common patterns for requesting identity verification services, communicating assurance requirements, and evaluating resulting identity assurance claims. It builds upon the OpenID Connect framework to enable integration across identity verification providers while maintaining security and privacy for identity assurance use cases.

The profile specification is intended to benefit both relying parties and identity providers by reducing integration complexity, improving interoperability, and establishing clear expectations for identity verification processes across different organizational boundaries and regulatory jurisdictions.

# Relying Party Profile for Identity Assurance

Relying Parties (RPs) require a standardized, interoperable way to request and receive verified identity information from OpenID Providers (OPs). In many regulated or high-assurance environments, RPs must ensure that user attributes—such as name, date of birth, or address—have been verified to a specific level of assurance and with acceptable evidence.

Through this profile, RPs can more easily obtain the verified identity information they need, with confidence in the assurance level and verification process, while minimizing integration overhead and improving interoperability.

## Request to Pushed Authorization Request (PAR) Endpoint

This profile outlines the use of Pushed Authorization Request (PAR) as defined in RFC 9126 for initializing an identity verification requests. The PAR mechanism provides enhanced security and privacy for identity assurance use cases by enabling the relying party to securely communicate complex identity verification requirements to the identity provider.

PAR initiated by the relying party makes an HTTP POST request to the authorization server's PAR endpoint containing the authorization request parameters that would normally be sent in the authorization request URL. The authorization server validates the request, stores the parameters, and returns a `request_uri` value and `expires_in` parameter. This `request_uri` is then used in place of individual parameters in the subsequent authorization request to the authorization endpoint.

A claims parameter included in the request is derived from the OpenID Connect for Identity Assurance (OIDC4IA) specification, which allows the relying party to specify the identity attributes that need to be verified and the assurance requirements for each claim.

### Request Parameters

When building a PAR request for identity verification, the relying party sends an authorization request parameters directly to the PAR endpoint. While a typical parameter set includes standard OAuth 2.0 and OpenID Connect parameters, this request must include the `claim` parameter the defines the OIDC4IA `verified_claims` structure to specify detailed identity verification requirements.

The following describes the PAR parameters when initiating an identity verification request.

`response_type`
: REQUIRED. MUST be set to `code` to use the authorization code flow.

`client_id`
: REQUIRED. The client identifier registered with the OpenID Provider.

`client_secret`
: REQUIRED. The client secret associated with the client_id.

`scope`
: REQUIRED. MUST include `openid`, `profile` and `identity_assurance`.

`state`
: REQUIRED. An unguessable random string used by the relying party to maintain state between the request and callback to prevent CSRF attacks.

`redirect_uri`
: REQUIRED. The URI where the OpenID Provider will redirect the user after processing the identity verification request.

`code_challenge` and `code_challenge_method`
: RECOMMENDED. Enabling PKCE protocol set by the client

`login_hint`
: REQUIRED. A hint to the OpenID Provider about the user's identity, such as an email address or username.

`nonce`
: RECOMMENDED. A string value used to associate the client session with the ID Token and provide replay protection.

`claims`
: REQUIRED. A JSON object that specifies which identity attributes need to be verified and the assurance requirements for each claim, following the OpenID Connect for Identity Assurance (OIDC4IA) specification structure for `verified_claims`. The relying party uses this parameter to indicate:
  - Which personal attributes are required (e.g., name, date of birth, address)
  - The verification level needed for each attribute
  - Acceptable evidence types for verification
  - Trust framework requirements as defined in the OIDC4IA `verified_claims` structure

#### Verified Claims Extensions

This profile extends the OpenID Connect for Identity Assurance (OIDC4IA) specification with two key enhancements designed to improve interoperability and user experience in identity verification scenarios:

1. **IDV_DELEGATED Trust Framework**: A new trust framework value that enables delegation of identity verification to the OpenID Provider (OP). This extension includes associated assurance levels (`VERIFIED` and `FAILED`) that communicate the verification outcome to the relying party.

2. **Fuzzy Name Matching**: An extension to the claims structure that allows relying parties to specify whether approximate string matching should be applied when comparing user-provided data against verified claims. This capability accommodates common variations in name formatting and minor discrepancies while maintaining verification integrity.

These extensions enhance the flexibility and practical applicability of OIDC4IA for real-world identity verification use cases while maintaining compatibility with the core specification.

##### Delegated Trustframework

This profile introduces the `IDV_DELEGATED` trust framework value to support scenarios where identity verification is delegated to specialized identity verification providers. When a relying party specifies `IDV_DELEGATED` as the trust framework value, it indicates that the OpenID Provider should delegate the identity verification process to a qualified third-party identity verification service that meets the required assurance levels.

The `IDV_DELEGATED` trust framework enables relying parties to delegate identity verification requirements to OpenID Providers while maintaining standardized communication through the OpenID Connect protocol. This delegation model allows relying parties to offload complex identity verification processes to specialized providers that can meet their assurance requirements. The assurance level requirements specified in the `verified_claims` request support two value types: `VERIFIED` for successful identity verification and `FAILED` for cases where verification could not be completed. These assurance level results are communicated from the OpenID Provider back to the relying party, ensuring that the verification process meets the relying party's requirements without requiring the relying party to implement or manage the underlying verification infrastructure.

The `IDV_DELEGATED` trust framework is in addition to the alternate trust frameworks and assurance levels that are defined in OIDC4IA, which may include:

| Trust Framework | Description | Assurance Levels | Use Cases |
|-----------------|-------------|------------------|-----------|
| `eidas` | European eIDAS regulation framework | `low`, `substantial`, `high` | EU digital identity services, cross-border authentication |
| `nist_800_63A` | NIST Special Publication 800-63A | `ial1`, `ial2`, `ial3` | US federal government systems, FICAM compliance |
| `uk_tfida` | UK Trust Framework for Identity Assurance | `low`, `medium`, `high` | UK government digital services, GOV.UK Verify |
| `ca_pan_canadian` | Canadian Pan-Canadian Trust Framework | `1`, `2`, `3`, `4` | Canadian government services, provincial identity systems |
| `jp_trustdock` | Japanese TrustDock framework | `loa1`, `loa2`, `loa3` | Japanese digital identity verification |
| `de_idnow` | German IDnow framework | `basic`, `professional`, `enterprise` | German financial services, KYC compliance |

Each trust framework defines specific requirements for identity proofing, authentication, and federation that align with regional regulations and industry standards. For example, `eidas` compliance requires specific evidence types and verification procedures for different assurance levels, while `nist_800_63A` defines precise requirements for identity proofing strength and verification methods.

The `IDV_DELEGATED` trust framework introduced in this profile complements these existing frameworks by providing a mechanism for relying parties to delegate identity verification to qualified providers while maintaining standardized communication through OpenID Connect protocols.


##### Fuzzy Matching

The `claims` parameter may include a `fuzzy` attribute for each claim, indicating whether the OpenID Provider should perform fuzzy matching on the claim value. This allows for flexibility in matching user-provided data against verified claims, accommodating minor discrepancies in user input.

Fuzzy matching is particularly valuable in identity verification scenarios where user-provided information may contain minor variations from the authoritative source data. Common discrepancies include differences in name formatting (e.g., "John" vs "Johnny"), spacing variations, punctuation differences, or minor typographical errors. When the `fuzzy` attribute is set to `true` for a specific claim, the OpenID Provider should apply appropriate string matching algorithms to determine if the user-provided value is sufficiently similar to the verified claim value.

The implementation of fuzzy matching should consider factors such as:
- Character similarity and common substitutions
- Phonetic matching for names that sound similar but are spelled differently
- Handling of diacritics and accents in international names
- Case-insensitive matching
- Tolerance for whitespace and punctuation variations

When `fuzzy` is set to `false`, the OpenID Provider MUST perform exact matching between the user-provided value and the verified claim. This strict matching mode is appropriate for claims where precision is critical, such as government identification numbers, dates of birth, or other fields where exact correspondence is required for compliance or security purposes.

The fuzzy matching capability enables a balance between user experience and verification accuracy, allowing legitimate users with minor data variations to successfully complete identity verification while maintaining the integrity of the verification process.

### Example: POST /oauth2/par request

```
POST /oauth/par HTTP/1.1

Host: idv-vendor.com
Content-Type: application/x-www-form-urlencoded

response_type=code
&client_id=aB3kL9mQ
&client_secret=xP8nM2kQ7sR4
&code_challenge=vT9bN3mL8dF1
&code_challenge_method=S256
&scope=openid+profile+identity_assurance
&claims={
    "id_token": {
      "verified_claims": [
        {
          "verification": {
            "trust_framework": {
              "value": "IDV-DELEGATED",
              "essential": true
            },
            "assurance_level": {
              "value": "VERIFIED",
              "essential": true
            }
          },
          "claims": {
            "given_name": {
              "value": "John",
              "fuzzy": true
            },
            "family_name": {
              "value": "Doe",
              "fuzzy": false
            },
            "birthdate": {
              "value": "1992-01-01",
              "fuzzy": false
            }
          }
        }
      ]
    }
  }
&state=wLPOSunzNXu3ZXf8Rn
&login_hint=user_Ka8mN2pQ3xR7
&redirect_uri=https://relyingparty.com/idp/identity-verification/callback
```

## PAR Response

Upon receiving a valid PAR request, the identity provider generates a unique `request_uri` that:

- References the stored authorization request parameters
- Has a limited lifetime as specified in the PAR response
- Can be used in subsequent authorization requests without exposing sensitive parameters in URL query strings

The use of PAR ensures that sensitive identity verification requirements are transmitted securely and are not exposed in browser history, server logs, or other potential information leakage vectors that could occur with traditional authorization request methods.

### Example: POST /oauth2/par response

```
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-cache, no-store

{
  "request_uri": "urn:ietf:params:oauth:request_uri:6esc_11ACC5bwc014ltc14eY22c",
  "expires_in": 60
}
```

## Error Handling

If during the PAR request and error occurs, then the error response should follow specification 
OAuth 2.0 Pushed Authorization Requests [RFC 9126](https://www.rfc-editor.org/rfc/rfc9126.html#name-error-response)

For example if the PAR request cannot be completed due to an invalid or missing parameter, then the error response would result in:

```
HTTP/1.1 400 Bad Request
 Content-Type: application/json
 Cache-Control: no-cache, no-store
 {
   "error": "invalid_request",
   "error_description":
     "The redirect_uri is not valid for the given client"
 }
```

# Authorization Request with Request URI

After successfully receiving the `request_uri` from the PAR endpoint, the relying party initiates the second phase of the identity verification flow by redirecting the user to the OP authorization endpoint. This redirect includes the `request_uri` parameter that references the previously pushed authorization request parameters.

The relying party constructs an authorization request URL that directs the user's browser to the OP, where the identity verification process will be conducted. The use of the `request_uri` ensures that all sensitive verification requirements remain securely stored at the OP rather than being exposed in the authorization URL.

## Authorization Request Parameters

The authorization request MUST include the following parameters when redirecting the user to the OP:

`client_id`
: REQUIRED. The client identifier registered with the OP.

`request_uri`
: REQUIRED. The URI returned from the PAR endpoint that references the stored authorization request parameters.

`response_type`
: OPTIONAL. May be included but is redundant since this information is already contained in the pushed request parameters.

Additional parameters from the original authorization request (such as `state`, `nonce`, `redirect_uri`) are not included in the authorization URL since they are already stored and referenced by the `request_uri`.

## Example: GET /oauth2/authorize

The relying party redirects the user to the OP's authorization endpoint using the `request_uri` obtained from the PAR response:

```
HTTP/1.1 302
GET /oauth/authorize?client_id=aB3kL9mQ&request_uri=urn:ietf:params:oauth:request_uri:6esc_11ACC5bwc014ltc14eY22c

Host: idv-vendor.com
Content-Type: application/x-www-form-urlencoded
```

Upon receiving this request, the OP:

1. Validates the `client_id` and `request_uri` parameters
2. Retrieves the stored authorization request parameters associated with the `request_uri`
3. Initiates the identity verification process based on the `verified_claims` requirements
4. Presents the appropriate identity verification interface to the user
5. Conducts the verification process according to the specified trust framework and assurance level requirements

The user completes the identity verification process at the OP, which may include document upload, biometric verification, or other verification methods as determined by the `IDV_DELEGATED` trust framework implementation.

## Authorization Response and Callback

Upon successful completion of the identity verification process, the OP redirects the user back to the relying party's `redirect_uri` that was specified in the original PAR request. This redirect contains the authorization code that enables the relying party to retrieve the identity verification results.

The authorization response follows the standard OAuth 2.0 authorization code flow pattern, with the authorization code serving as a secure reference to the completed identity verification session.

### Authorization Response Parameters

The OP redirects the user to the relying party's callback endpoint with the following parameters:

`code`
: REQUIRED. The authorization code generated by the OP after successful identity verification. This code can be exchanged for an ID token containing the verified claims.

`state`
: REQUIRED. The unguessable random string that was provided in the original PAR request. The relying party MUST verify that this value matches the state parameter from the original request to prevent CSRF attacks.

`iss`
: OPTIONAL. The issuer identifier of the OP, as defined in OAuth 2.0 Authorization Server Issuer Identification.

### Example: Authorization Response Redirect

After completing the identity verification process, the OP redirects the user back to the relying party:

```
HTTP/1.1 302 Found
Location: https://relyingparty.com/idp/identity-verification/callback?code=SplxlOBeZQQYbYS6WxSbIA&state=wLPOSunzNXu3ZXf8Rn&iss=https%3A%2F%2Fidv-vendor.com
```

### Callback Processing

Upon receiving the authorization response, the relying party MUST:

1. **Validate the state parameter**: Verify that the `state` parameter matches the value sent in the original PAR request to prevent CSRF attacks
2. **Extract the authorization code**: Retrieve the `code` parameter that will be used to obtain the identity verification results
3. **Verify the issuer** (if present): Ensure the `iss` parameter matches the expected OP issuer identifier
4. **Handle error responses**: Process any error parameters that may indicate verification failure or other issues

The relying party can then proceed to exchange the authorization code for an ID token containing the verified claims through the token endpoint request.

### Error Handling in Authorization Response

If the identity verification process fails or encounters an error, the OP may redirect the user back to the relying party with error parameters instead of an authorization code:

```
HTTP/1.1 302 Found
Location: https://relyingparty.com/idp/identity-verification/callback?error=access_denied&error_description=The+user+denied+the+request&state=wLPOSunzNXu3ZXf8Rn
```

Common error codes include:
- `access_denied`: The user denied the identity verification request
- `invalid_request`: The request was malformed or contained invalid parameters
- `server_error`: The OP encountered an internal error during processing
- `temporarily_unavailable`: The OP is temporarily unavailable

The relying party should handle these error conditions appropriately and provide meaningful feedback to the user about the identity verification status.

# Token Request for Identity Verification Results

After the user completes the identity verification process at the OpenID Provider, the OP redirects the user back to the relying party's `redirect_uri` with an authorization code. The relying party then exchanges this authorization code for an ID token that contains the results of the identity verification flow.

This token exchange represents the final phase of the identity verification profile, where the relying party receives the verified claims and assurance level information that was processed during the user's verification session with the OP.

## Token Request Parameters

The relying party makes a POST request to the OP's token endpoint to exchange the authorization code for tokens. The token request MUST include the following parameters:

`grant_type`
: REQUIRED. MUST be set to `authorization_code`.

`code`
: REQUIRED. The authorization code received from the authorization endpoint callback.

`client_id`
: REQUIRED. The client identifier registered with the OP.

`client_secret`
: REQUIRED. The client secret associated with the client_id for client authentication.

`redirect_uri`
: REQUIRED. MUST exactly match the redirect_uri used in the original PAR request.

`code_verifier`
: REQUIRED if PKCE was used in the authorization request. The code verifier that corresponds to the code_challenge sent in the PAR request.

## Example: POST /oauth2/token

The relying party exchanges the authorization code for tokens containing the identity verification results:

```
POST /oauth/token HTTP/1.1
Host: idv-vendor.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&client_id=aB3kL9mQ
&client_secret=xP8nM2kQ7sR4
&redirect_uri=https://relyingparty.com/idp/identity-verification/callback
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

## Token Response with Verified Claims

The OP responds with an ID token that contains the results of the identity verification process. The ID token includes verified claims structured according to the OIDC4IA specification with the extensions defined in this profile:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache, no-store

{
  "token_type": "Bearer",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL2lkdi12ZW5kb3IuY29tIiwic3ViIjoidXNlcl9LYThtTjJwUTN4UjciLCJhdWQiOiJhQjNrTDltUSIsImV4cCI6MTY0MjU1NzYwMCwiaWF0IjoxNjQyNTU0MDAwLCJub25jZSI6Im5vbmNlVmFsdWUiLCJ2ZXJpZmllZF9jbGFpbXMiOlt7InZlcmlmaWNhdGlvbiI6eyJ0cnVzdF9mcmFtZXdvcmsiOiJJRFYtREVMRUdBVEVEIiwiYXNzdXJhbmNlX2xldmVsIjoiVkVSSUZJRUQiLCJ2ZXJpZmljYXRpb25fcHJvY2VzcyI6IjEyMzQ1Njc4LWFiY2QtZWZnaC1oaWprLWxtbm9wcXJzdHV2dyIsInRpbWUiOiIyMDI0LTAxLTE1VDEwOjAwOjAwWiIsImV2aWRlbmNlIjpbeyJ0eXBlIjoiZG9jdW1lbnQiLCJtZXRob2QiOiJhdXRvbWF0ZWQiLCJkb2N1bWVudCI6eyJ0eXBlIjoiZHJpdmluZ19saWNlbnNlIiwiaXNzdWVyIjp7Im5hbWUiOiJDQSBETVYiLCJjb3VudHJ5IjoiVVMifX19XX0sImNsYWltcyI6eyJnaXZlbl9uYW1lIjoiSm9obiIsImZhbWlseV9uYW1lIjoiRG9lIiwiYmlydGhkYXRlIjoiMTk5Mi0wMS0wMSJ9fV19"
}
```

### ID Token Structure

The `id_token` is a signed JWT (JSON Web Token) that contains the identity verification results. When decoded, the JWT payload reveals the complete structure of the verified claims and all required OpenID Connect attributes.

#### Decoded JWT Payload

When the `id_token` JWT is decoded and verified, the payload contains the following structure with all required attributes and verified claims results:

```json
{
  "iss": "https://idv-vendor.com",
  "sub": "user_Ka8mN2pQ3xR7",
  "aud": "aB3kL9mQ",
  "exp": 1642557600,
  "iat": 1642554000,
  "auth_time": 1642553800,
  "nonce": "nonceValue",
  "acr": "urn:mace:incommon:iap:silver",
  "amr": ["pwd", "otp"],
  "azp": "aB3kL9mQ",
  "verified_claims": [
    {
      "verification": {
        "trust_framework": "IDV_DELEGATED",
        "assurance_level": "VERIFIED",
        "verification_process": "12345678-abcd-efgh-hijk-lmnopqrstuvw",
        "time": "2024-01-15T10:00:00Z",
        "evidence": [
          {
            "type": "document",
            "method": "automated",
            "document": {
              "type": "driving_license",
              "issuer": {
                "name": "CA DMV",
                "country": "US"
              },
              "number": "D1234567",
              "date_of_issuance": "2020-01-15",
              "date_of_expiry": "2028-01-15"
            }
          }
        ]
      },
      "claims": {
        "given_name": "John",
        "family_name": "Doe",
        "birthdate": "1992-01-01"
      }
    }
  ]
}
```

## ID Token Claims Structure

The ID token contains verified claims with the following structure that reflects the results of the identity verification process:

- **verification**: Contains metadata about the verification process including:
  - `trust_framework`: Set to `IDV_DELEGATED` indicating the verification was delegated
  - `assurance_level`: Either `VERIFIED` or `FAILED` based on the verification outcome
  - `verification_process`: Unique identifier for the verification session
  - `time`: Timestamp when the verification was completed
  - `evidence`: Array of evidence types used in the verification process

- **claims**: Contains the verified identity attributes with their verified values, reflecting any fuzzy matching that was applied during the verification process

The relying party can parse this ID token to extract the verified claims and determine whether the identity verification was successful based on the `assurance_level` value. This completes the identity verification flow, providing the relying party with the verified identity information needed for their use case.

### Failed Identity Verification Results

When identity verification cannot be completed successfully, the OpenID Provider returns an ID token with `assurance_level` set to `FAILED`. In these cases, the `verified_claims` structure must clearly indicate which claims could not be verified and provide appropriate failure information.

#### Representing Unverified Claims

According to the OpenID Connect for Identity Assurance specification, unverified claims can be represented in two ways:

1. **Omission**: Claims that could not be verified are omitted entirely from the `claims` object
2. **Null Values**: Claims that could not be verified are included with `null` values

#### Example: Failed Verification Response

When verification fails, the decoded JWT payload would contain a `null` value for the claim (or claims) that could not be verified:

```json
{
  "iss": "https://idv-vendor.com",
  "sub": "user_Ka8mN2pQ3xR7",
  "aud": "aB3kL9mQ",
  "exp": 1642557600,
  "iat": 1642554000,
  "auth_time": 1642553800,
  "nonce": "nonceValue",
  "verified_claims": [
    {
      "verification": {
        "trust_framework": "IDV_DELEGATED",
        "assurance_level": "FAILED",
        "verification_process": "12345678-abcd-efgh-hijk-lmnopqrstuvw",
        "time": "2024-01-15T10:00:00Z",
        "evidence": [
          {
            "type": "document",
            "method": "automated",
            "document": {
              "type": "driving_license",
              "issuer": {
                "name": "CA DMV",
                "country": "US"
              }
            }
          }
        ]
      },
      "claims": {
        "given_name": "John",
        "family_name": "Doe",
        "birthdate": null
      }
    }
  ]
}
```

Alternatively, with omission approach, exclude the claim (or claims) that could not be verified

```json
{
  "iss": "https://idv-vendor.com",
  "sub": "user_Ka8mN2pQ3xR7",
  "aud": "aB3kL9mQ",
  "exp": 1642557600,
  "iat": 1642554000,
  "auth_time": 1642553800,
  "nonce": "nonceValue",
  "verified_claims": [
    {
      "verification": {
        "trust_framework": "IDV_DELEGATED",
        "assurance_level": "FAILED",
        "verification_process": "12345678-abcd-efgh-hijk-lmnopqrstuvw",
        "time": "2024-01-15T10:00:00Z",
        "evidence": [
          {
            "type": "document",
            "method": "automated",
            "document": {
              "type": "driving_license",
              "issuer": {
                "name": "CA DMV",
                "country": "US"
              }
            }
          }
        ]
      },
      "claims": {}
    }
  ]
}
```

#### Handling Failed Verification

When processing failed verification results, relying parties should:

1. **Check assurance_level**: Always verify that `assurance_level` is `FAILED`
2. **Handle missing claims**: Be prepared to handle both null values and omitted claims
3. **Examine evidence**: Review the evidence object to understand what verification was attempted
4. **Process verification_process**: Use the verification process ID for audit trails and debugging
5. **Implement fallback logic**: Have appropriate fallback mechanisms for cases where verification cannot be completed

The presence of evidence information even in failed cases allows relying parties to understand what verification methods were attempted and can inform decisions about retry strategies or alternative verification approaches.

## Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
