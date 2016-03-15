jwt cross-organizational draft from Argonauts
=========

 4.5.3 Authorization Token

The authorization request SHALL BE a JWT, as defined in RFC7519 and SHALL contain the details the EHR-B authorization server will need to know in order to mediate the request for access to a FHIR resource. The HTTP parameters for transporting the authorization JWT from the EHR-A authorization server to the EHR-B authorization server's token endpoint SHALL BE as defined in The OAuth Assertion Framework RFC7521, with the following specific parameter values and encodings (as shown in the sequence diagram above):

    The value of the "grant_type" MUST BE "urn:ietf:params:oauth:grant-type:jwt-bearer."
    The value of the "assertion" parameter MUST contain a single signed authorization JWT.

Example:

POST body: 
grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer&
assertion={signed authorization JWT}&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion+{signed authentication JWT}

The authorization JWT MUST be digitally signed by the EHR-A authorization server as specified in RFC7515, JSON Web Signature, using the private key counterpart to the public key registered with EHR-B. The EHR-B authorization server MUST support the RS256 signature method and MAY use other asymmetric signature methods listed in JSON Web Algorithms (JWA): draft-ietf-jose-json-web-algorithms-14.

Upon receipt, the EHR-B authorization server MUST validate the digital signature and MUST reject authorization JWTs with an invalid digital signature. The algorithm used to validate the signature, and the mechanism for designating the secret used to generate the signature, are outside the scope of this specification.

The authorization JWT SHALL contain claims relating to the resource being requested (e.g., FHIR patient resource, data scope, requesting practitioner, reason) and claims necessary to help ensure the security of the exchange (expiration time, issuer, subject, a token identifier; see RFC7523 for details):
Claim 	Priority 	Description
iss 	REQUIRED 	Requesting EHR's issuer URI.
sub 	REQUIRED 	EHR-A's id for the user on whose behalf this request is being made. Matches requesting_practitioner.id
acr 	REQUIRED 	Level of assurance of the requesting user's identity (e.g. NIST level 0-4, as defined in NIST SP 800-63-2)
aud 	REQUIRED 	EHR-B authorization server's token_URL (the URL to which this authorization JWT will be posted)
requested_record 	REQUIRED 	The FHIR patient resource being requested
requested_scopes 	REQUIRED 	Patient data being requested
requesting_practitioner 	REQUIRED 	FHIR practitioner resource making the request
reason_for_request 	REQUIRED 	Purpose for which access is being requested
exp 	REQUIRED 	Expiration time integer after which this authorization JWT MUST be considered invalid; expressed in seconds since the "Epoch" (1970-01-01T00:00:00Z UTC). This time MUST be no more than five minutes in the future.
kid 	REQUIRED 	Key id of the encryption key used to digitally sign this token
jti 	REQUIRED 	A nonce string value that uniquely identifies this authorization JWT. MUST have at least 128 bits of entropy and MUST NOT be reused in another token. EHR-B MUST check for reuse of jti values and MUST reject all tokens issued with duplicate jti values.
iat 	REQUIRED 	The UTC time the JWT was created by EHR-A.
Example

{
  "iss": "https://ehr-a.com",
  "sub": "128641521",
  "aud": "https://ehr-b.com/token",
  "acr": "http://nist.gov/id-proofing/level/3",
  "jit": "some-nonce-abc",
  "iat": 1418698788,
  "exp": 1422568860,
  "requested_record": {
    "resourceType": "Patient",
    "name": {
      "text": "Pauline Smith",
    },
    "address": {
      "postalCode": "94118"
    }
    "gender": "female",
    "birthDate": "1970-05-18"
  },
  "reason_for_request": "treatment",
  "requested_scopes": "patient/*.read",
  "requesting_practitioner": {
    "resourceType": "Practitioner",
    "id": "128641521",
    "identifier": [{
      "system": "https://ehr-a.com",
      "value": "123"
    },{
      "system": "https://nppes.cms.hhs.gov/",
      "value": "1770589525"
    }],
    "name": {
      "text": "Juri van Gelder"
    },
    "practitionerRole": [{
      "role": {
        "coding": [{
          "system": "http://snomed.info/sct",
          "code": "36682004",
          "display": "Physical therapist"
        }]}}]}}
