## Appendix B: OpenID Federation Digital Credentials Profile

### Introduction

This specification profiles OpenID Federation [[ref: OpenID Federation]] for use with digital credentials, specifically focusing on credential issuance via OpenID for Verifiable Credential Issuance [[ref: OID4VCI]] and credential presentation via OpenID for Verifiable Presentations [[ref: OID4VP]].

The profile simplifies federation usage by limiting scope to trust chain resolution and metadata discovery, while omitting more complex features such as federation policies and trust marks.

#### Scope

This profile applies to:

- Credential issuers publishing OID4VCI metadata
- Credential verifiers publishing verifier metadata
- Wallets resolving trust chains to establish trust in issuers and verifiers

This profile does not apply to and does not require conforming implementations to support:

- Chapter 6 (Federation Policy) of OpenID Federation
- Chapter 7 (Trust Marks) of OpenID Federation  
- Chapter 12 (OpenID Connect Client Registration) of OpenID Federation

### Terminology

This specification uses the terms defined in OpenID Federation [[ref: OpenID Federation]], OpenID for Verifiable Credential Issuance [[ref: OID4VCI]], OpenID for Verifiable Presentations [[ref: OID4VP]], and SD-JWT VC [[ref: SD-JWT VC]].

**Entity Identifier**: As defined in OpenID Federation, a URL that uniquely identifies a federation entity.

**Entity Configuration**: As defined in OpenID Federation, a signed JWT published at the Entity Identifier's `/.well-known/openid-federation` endpoint containing metadata and authority hints.

**Trust Chain**: As defined in OpenID Federation, a sequence of Entity Statements from a Leaf Entity through intermediate entities to a Trust Anchor.

**Issuer**: The role of the entity issuing credentials as defined in [[ref: W3C VCDM]]

**Credential Issuer**: The technical service used to issue credentials as defined in [[ref: OID4VCI]]

**Verifier**: The entity that requests, receives, and validates Presentations as defined in [[ref: OID4VCI]]. Note that this specification does not distinguish the role and the technical service of the verifier the same way it does for Issuer and Credential Issuer. For the purposes of this specification Verifier may refer to either the role or the technical service. (Thus, the reference to the definition in [[ref: OID4VCI]] and not the one in [[ref: OID4VP]].)

### Credential Issuance

#### Entity Identifier

The Credential Issuer MUST use the value of the `credential_issuer` in its OID4VCI issuer metadata as its Entity Identifier.

The Credential Issuer's Entity Configuration can be found by appending the string `/.well-known/openid-federation` to the Entity Identifier.

#### Issuer Metadata Publication

The Credential Issuer MUST place the OpenID4VCI issuer metadata into the Entity Configuration, in the `openid_credential_issuer` property.

If the `openid_credential_issuer` property is found in the Entity Configuration, the Wallet MUST use only it and ignore the issuer metadata published in the well-known location defined in OID4VCI.

The Credential Issuer MAY place additional metadata into the `federation_entity` Entity Type Identifier.

The metadata in the `openid_credential_issuer` property overrides the metadata in the `federation_entity` property. For example, if both `openid_credential_issuer.display.name` and `federation_entity.organization_name` exist, the Wallet SHOULD show the value of `openid_credential_issuer.display.name` as the name of the issuer.

#### Example: Credential Issuer Entity Configuration

```json
{
  "iss": "https://credential-issuer.example",
  "sub": "https://credential-issuer.example",
  "iat": 1616239022,
  "exp": 1616239322,
  "metadata": {
    "federation_entity": {
      "organization_name": "Example Credential Issuer",
      "contacts": ["support@credential-issuer.example"]
    },
    "openid_credential_issuer": {
      "issuer": "https://credential-issuer.example",
      "display": [
        {
          "name": "Example Issuer",
          "locale": "en-US",
          "logo": {
            "uri": "https://credential-issuer.example/logo.png",
            "alt_text": "Example Logo"
          }
        }
      ],
      "credential_issuer": "https://credential-issuer.example",
      "authorization_endpoint": "https://credential-issuer.example/authorize",
      "authorization_servers": ["https://credential-issuer.example/authorize"],
      "credential_endpoint": "https://credential-issuer.example/credential",
      "credential_configurations_supported": {
        "sd_jwt_vc_example": "..."
      }
    },
    "openid-configuration": {
      "jwks_uri": "https://credential-issuer.example/jwks"
    }
  },
  "jwks": [
    {
      "kty": "EC",
      "kid": "MJ2BW-rNshp9sjh3SvwnBIkEsYsU92xVtC3-Fv_lcKc",
      "alg": "ES256",
      "crv": "P-256",
      "x": "JTEE5QghmkA_-7_pZoKIluRzGNvQGtzmpNvb_nAswhE",
      "y": "A_iBfIseHsdfE7CmI3lIYtKMdfyXXOIpPX_o6O0h0wY",
      "use": "sig"
    }
  ],
  "authority_hints": [
    "https://trustregistry.example"
  ]
}
```

#### SD-JWT VC Credentials

When the [[ref: Issuer]] issues credentials in the [[ref: SD-JWT VC]] format, the Issuer MUST place its Entity Identifier in the `fed` claim of the credential.

##### Example: SD-JWT VC with Federation Claim

The following non-normative example shows a decoded payload of an [[ref: SD-JWT VC]] credential with the `fed` claim:

```json
{
  "iss": "https://credential-issuer.example",
  "fed": "https://credential-issuer.example",
  "iat": 1683000000,
  "exp": 1883000000,
  "vct": "https://credentials.example.com/identity_credential",
  "is_over_65": true,
  "address": {
    "street_address": "123 Main St",
    "locality": "Anytown",
    "region": "Anystate",
    "country": "US"
  }
}
```

#### W3C VCDM Credentials

When the [[ref: Issuer]] issues credentials in the [[ref: W3C VCDM]] format, the Issuer MUST place a `termsOfUse` property into the credential. The `type` of this `termsOfUse` property MUST be the string `OpenIDFederation` and the `policyId` MUST be the Issuer's Entity Identifier.

##### Example: W3C VCDM Credential with termsOfUse

The following non-normative example illustrates the use of the `termsOfUse` property:

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2"
  ],
  "id": "urn:uuid:c65d364e-2560-4e08-be03-9c3d944d609d",
  "type": [
    "VerifiableCredential"
  ],
  "issuer": "did:web:credential-issuer.example",
  "validFrom": "2025-12-01T00:00:00Z",
  "validUntil": "2028-12-31T23:59:59Z",
  "credentialSubject": {
  },
  "termsOfUse": {
    "type": "OpenIDFederation",
    "trustFramework": "Example Federation",
    "policyId": "https://credential-issuer.example"
  }
}
```

#### Note on Credential Issuer vs. Issuer

The Issuer defined in the digital credential (the value of the `iss` claim in SD-JWT VC credentials or the value of the `id` of the `issuer` property in W3C VCDM credentials) is not necessarily the same entity as the Credential Issuer defined in the property `credential_issuer` in the OID4VCI metadata.

For example, the OpenID Federation Entity Identifier of the Issuer of the credential could be `https://university-of-utopia.example.edu` and the OpenID Federation Entity Identifier of the Credential Issuer could be `https://credentials.ministryofeducation.example.edu`.

If the Issuer chooses to make this kind of distinction between the entity issuing the credential and the technical service used for issuance, it is RECOMMENDED that the Entity Configuration of the technical service has an `authority_hints` value pointing to the Issuer's Entity Identifier and the Issuer publishes a Subordinate Statement about the technical service.

### Credential Presentation and Verification

#### Client Identifier Scheme

The Verifier MUST use the `openid_federation:` prefix as defined in [[ref: OID4VP]] Section 5.9.3.

#### Verifier Metadata Publication

The Verifier MUST place verifier metadata into the Entity Configuration, in the `openid_credential_verifier` property.

### Trust Establishment

To establish trust with the issuer (ensure that the issuer can be trusted), the Verifier MUST resolve the Trust Chain from the issuer's Entity Configuration until it finds a Federation Entity it trusts.

#### Example: Verifier Entity Configuration

```json
{
  "iss": "https://credential-verifier.example",
  "sub": "https://credential-verifier.example",
  "iat": 1616239023,
  "exp": 1616239323,
  "metadata": {
    "federation_entity": {
      "organization_name": "Example Credential Verifier",
      "contacts": ["support@credential-verifier.example"]
    },
    "openid_credential_verifier": {
      "jwks": {
        "keys": [
          {
            "kty": "EC",
            "crv": "P-256",
            "x": "f83OJ3D2xF4Z1s3QpLQe4qVb8K7q6y1v3z4Yb6k9J0",
            "y": "x_FEzRu9q3u4bWz5n9X2L4q1U8T7c6v5s2d1a0b3C4",
            "alg": "ES256",
            "use": "enc",
            "kid": "ec-key-1"
          }
        ]
      },
      "encrypted_response_enc_values_supported": [
        "A128GCM",
        "A192GCM",
        "A256GCM"
      ],
      "vp_formats_supported": {
        "dc+sd-jwt": {
          "sd-jwt_alg_values": ["ES256"],
          "kb-jwt_alg_values": ["ES256"]
        }
      }
    }
  },
  "jwks": [
    {
      "kty": "EC",
      "kid": "y4nC8uTvcM5uJxOIvUqFjXb2EA6xPGdnt8zvjW94m6U",
      "alg": "ES256",
      "crv": "P-256",
      "x": "K6dA9ayt4P8xBN6SFiCZYOI2qeaFda7VV5wnmHWcl7w",
      "y": "CdE30dUX0geK4NL8IMC9u-rRMOLC9WaScJIGK5rxtKI"
    }
  ],
  "authority_hints": [
    "https://trustregistry.example"
  ]
}
```

### Subordinate Statements

Intermediaries and Trust Anchors MAY authorize their subordinates to issue or request certain credential types by placing metadata values like `openid_credential_issuer.credential_configurations_supported` or `openid_credential_verifier.dcql_query` in the Subordinate Statements.

The meaning of such statements SHOULD be specified in ecosystem rulebooks and are out of the scope of this specification.
