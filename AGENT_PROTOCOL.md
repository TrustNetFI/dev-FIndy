# Agent Protocol

FIndy ledger has been used to test out the ability to issue verifiable credentials as defined in the Hyperledger Indy [AnonCreds design](https://github.com/hyperledger/indy-sdk/tree/master/docs/design/002-anoncreds). The implementation still uses the Hyperledger Indy standard [Agent Protocol Specification](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#messaging-protocol) as the basis for the communications between the Issuer and the Holder / Prover.

Further on the plan is to transition to using the [HIPE agent connection protocol](https://github.com/hyperledger/indy-hipe/tree/master/text/0031-connection-protocol) as the basis for the agent-to-agent communications in the FIndy network.

# Protocol Flow

Following steps describe the process of an Issuer issuing a credential to a Holder / Prover. The flow starts with the creation of a pairwise connection between the two parties. This pairwise connection is used to protect the subsequent credential related communication.

## Connection Offer

The flow starts by e.g. an end-user authenticating to self-service portal. In the self-service portal the end-user has the possibility to export his/her verifiable credential to a wallet. This is e.g. done by outputting a connection offer as QR code that can be then scanned by the wallet application. The connection offer schema is defined [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#connection-offer). Following is the FIndy specific version of connection offer:

```

{
 "id": "<offer_nonce>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/connection/1.0/offer",
 "message": {
   "endpoint": "<endpoint URL of the issuer endpoint>"
 }
}
```

## Connection Request and Response

After receiving the connection offer, the holder / prover agent should construct a connection request and send it to the issuer remote endpoint provided in the connection offer. This is done with an HTTP POST request with below JSON request body. Connection request uses the schema definition defined [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#connection-request) as the basis. Following is the FIndy specific version of connection request:

```
{
 "id": "<offer_nonce>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/connection/1.0/request",
 "message": {
   "did": "<holder/prover agent did>",
   "nonce": "<request_nonce>"
 }
}
```

When receiving the connection request, the issuer reads the `verkey` corresponding to the provided `did` from the ledger. After validating the request, the issuer generates pairwise DID, stores it locally and links it to the authenticated user.

The issuer then responds with HTTP 200 and following FIndy specific payload which is based on the schema definition [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#connection-response:).

```

{
 "id": "<request_nonce>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/connection/1.0/response",
 "message": {
    "encrypted":"aGVsbG8K..."
  }
}
```

The `message` field contains following JSON in anon-encrypted and base64 encoded format:

```
{
   "did": "<pairwise did>",
   "verkey": "<pairwise verkey>",
   "nonce": "<request_nonce>"
}
```

## Connection Acknowledgement

After receiving the connection response, the holder / prover acknowledges receiving the connection response by sending an HTTP POST request to the issuer remote endpoint with following FIndy specific JSON request body. The schema is based on the definition that can be found [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#connection-acknowledgement):

```
{
 "id": "<pairwise did>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/connection/1.0/acknowledgement",
 "message": {
    "encrypted":"aGVsbG8K..."
  }
}
```

`message` field contains string `success` in auth-encrypted and Base64 encoded format. If validation is successful, issuer responds with HTTP status code 200. Otherwise it will respond with HTTP status code 400.

## Credential Offer

In order to send a credential request to the issuer, the holder / prover needs to first request a credential offer. This can be done by issuing an HTTP GET call to the issuer remote endpoint. In this case issuer responds with a JSON response body defined below. The schema is described [here](https://hyperledger-indy.readthedocs.io/projects/sdk/en/latest/docs/design/002-anoncreds/README.html).

```
{
  "id": "<offer_nonce>",
  "type": "urn:indy:sov:agent:message_type:findy.fi/credential/1.0/offer",
  "message": {
    "schema_id":"<schema_id>",
    "cred_def_id":"<cred_def_id>",
    "nonce":"<offer_nonce>",
    "key_correctness_proof":"<key_correctness_proof>"
  }
}
```

## Credential Request and Response

After receiving the credential offer, the holder / prover can send a credential request to the issuer remote endpoint. This is done with an HTTP POST request with following JSON provided in the request body. The schema is defined in [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#credential-request) and [here](https://hyperledger-indy.readthedocs.io/projects/sdk/en/latest/docs/design/002-anoncreds/README.html).

```
{
 "id": "<pairwise did>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/credential/1.0/request",
 "message": {
   "encrypted":".."
 }
}
```

`message` contains credential request in anon-encrypted format:

```

{
   "prover_did": "<edge agent did>",
   "cred_def_id": "<cred_def_id>",
   "blinded_ms": "<blinded_ms>",
   "blinded_ms_correctness_proof": "<blinded_ms_correctness_proof>",
   "nonce": "<offer_nonce>"
}
```

Issuer responds with HTTP 200 and following response payload. The schema is described [here](https://hyperledger-indy.readthedocs.io/projects/agent/en/latest/README.html#credential).

```
{
 "id": "<offer_nonce>",
 "type": "urn:indy:sov:agent:message_type:findy.fi/credential/1.0/response",
 "message": {
    "encrypted":"aGVsbG8K..."
  }
}
```

`message` field contains following JSON in anon-encrypted and base64 encoded format:

```
{
         "schema_id": string,
         "cred_def_id": string,
         "rev_reg_def_id", Optional<string>,
         "values": <cred_values>,
         "signature": <signature>,
         "signature_correctness_proof": <signature_correctness_proof>
}
```
