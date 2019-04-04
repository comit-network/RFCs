# Basic HTLC Atomic Swap

- RFC-Number: 003
- Status: Draft
- Discussion issue: [#19](https://github.com/comit-network/RFCs/issues/19)
- Created on: 15 Jan. 2019

**Table of Contents**
- [Description](#description)

- [Concepts](#concepts)
    - [Identity](#identity)
    - [Hash Time Lock Contract (HTLC)](#hash-time-lock-contract-htlc)
- [Setup Phase](#setup-phase)
    - [SWAP Request Header](#swap-request-header)
        - [`hash_function`](#hash_function)
    - [SWAP Request Body](#swap-request-body)
    - [Swap Response (Accept)](#swap-response-accept)
    - [Swap Response (Decline)](#swap-response-decline)
        - [`timeouts-too-tight`](#timeouts-too-tight)
- [Execution Phase](#execution-phase)
    - [1. Alice deploys α-HTLC](#1-alice-deploys-%CE%B1-htlc)
    - [2. Bob deploys β-HTLC](#2-bob-deploys-%CE%B2-htlc)
    - [3. Alice redeems](#3-alice-redeems)
    - [4. Bob redeems](#4-bob-redeems)
- [Application Considerations](#application-considerations)
- [Security Considerations](#security-considerations)
- [Registry Extensions](#registry-extensions)
- [References](#references)
- [Bitcoin to Ethereum](#bitcoin-to-ethereum)
    - [SWAP REQUEST](#swap-request)
    - [SWAP RESPONSE (accept)](#swap-response-accept)
    - [SWAP RESPONSE (reject: `timeouts-too-tight`)](#swap-response-reject-timeouts-too-tight)

## Description

This RFC describes a basic atomic swap protocol and the related parameters required to use it as a COMIT [RFC002](./RFC-002-SWAP.md#protocol) SWAP protocol.
It uses Hash Time Locked Contracts to swap ownership of two assets on different ledgers between two parties.
It is a simplified version of the protocol originally described by TierNolan[¹](#references).

## Concepts

### Identity

This RFC introduces the notion of an "identity".
An identity belongs to a user and represents the minimal information required to transfer an asset to that user on a particular ledger.
On the ledger, the identity is assigned the asset and a user owns the asset in so far as they own the identity.
On blockchain based ledgers a user's identity is usually some function of a public key and the user owns the identity by having exclusive knowledge of the private key.

This RFC extends the registry to include a section specifying the identity for each supported ledger.
Any subsequent RFCs adding a ledger definition MUST specify its identity.

### Hash Time Lock Contract (HTLC)

The Hash Time Locked Contract (HTLC) is the primary construct used in this protocol.
A HTLC locks an asset until someone activates one of two possible paths:

- **Activation with secret**: The contract is activated with a *secret* whose hash matches the hash in the contract.
- **Activation after expiry**: The contract is activated after an expiry time specified in the contract.

Each activation path transfers the asset to a different party.
The parties are decided at the time of contract creation.
In this RFC, the expiry activation returns the asset to the original owner while a secret activation transfers it to the other party.
Therefore this document will refer to these paths as *refund* and *redeem* respectively.

The HTLCs defined in this specification use *absolute time locks* where the expiry is set to a specific time in the future.
Absolute time locks are necessary because the protocol is only secure if the expiration of the time locks for HTLCs on different ledgers are fixed relative to each other.
*Relative time lock* HTLCs have an expiration time that is relative to their inclusion in the ledger.
In this protocol, they would allow an attacker to manipulate the relative expiration times of the HTLCs if they are able to delay the inclusion of a HTLC onto one of the ledgers.

HTLCs MUST enforce the length of the secret to be equal to the hash function's output length.
If this is not enforced, a secret may be able to active the redeem path on one HTLC but not on the other.

In this RFC, HTLCs are constructed with the following parameters:

  - **asset**: The asset locked in the HTLC.
  - **redeem_identity**: The identity the asset is transferred to upon activation of the redeem path.
  - **refund_identity**: The identity the asset is transferred to upon activation of the refund path.
  - **expiry**: The absolute time after which the refund path may be activated.
  - **secret_hash**: The hash whose pre-image is required to activate the redeem path.
  - **hash_function**: The cryptographic hash function that is used to produce the secret hash.

How to construct an HTLC for each ledger will be defined in subsequent RFCs.

## Setup Phase

In the setup phase the two parties exchange [RFC002](./RFC-002-SWAP.md) SWAP messages to negotiate the parameters of the HTLCs.
The values α, β, **A** and **B** used below refer to the ledgers and assets described by the SWAP headers `alpha_ledger`, `beta_ledger`, `alpha_asset` and `beta_asset` respectively.
Additionally, α-HTLC and β-HTLC refer to the HTLCs deployed on the α and β ledgers.

### SWAP Request Header

The protocol begins with one party (the sender) sending a SWAP REQUEST message to another (the receiver) with the `protocol` header's value set to `comit-rfc-003`.
The header MUST have the following parameters:

#### `hash_function`
Type: [Hash Function](./registry.md#hash-function)

The cryptographic hash function used in the construction of both HTLCs.
It MUST be available on the `alpha_ledger` and the `beta_ledger`.
This RFC defines `SHA-256`[²](#references) as an initial value for this parameter.
How to construct HTLCs based on SHA-256 and other hash functions for particular ledgers will be described in subsequent RFCs.

### SWAP Request Body
When `comit-rfc-003` is used as the value for `protocol` for a SWAP REQUEST message the body MUST have the following fields:

| Name                    | JSON Encoding       | Description                                                                                               |
|:-------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------|
| `alpha_expiry`          | `u32`               | The UNIX timestamp of the time-lock on the alpha HTLC                                                     |
| `beta_expiry`           | `u32`               | The UNIX timestamp of the time-lock on the beta HTLC                                                      |
| `alpha_refund_identity` | `α::Identity`       | The identity on α that **A** can be transferred to after `alpha_expiry`                                   |
| `beta_redeem_identity`  | `β::Identity`       | The identity on β that **B** will be transferred to when the β-HTLC is activated with the correct secret  |
| `secret_hash`           | `hex-encoded-bytes` | The output of calling `hash_function` on the secret                                                       |

If `alpha_expiry` or `beta_expiry` are in the past, implementations SHOULD consider the request to be invalid.

### Swap Response (Accept)

If responding with `OK00`, the responder MUST include the following fields in the response body:

| Name                    | JSON Encoding | Description                                                                                               |
|-------------------------|---------------|-----------------------------------------------------------------------------------------------------------|
| `alpha_redeem_identity` | `α::Identity` | The identity on α that **A** will be transferred to when the α-HTLC is activated with the correct secret  |
| `beta_refund_identity`  | `β::Identity` | The identity on β that **B** will be transferred to when the β-HTLC is activated after `beta_expiry`      |


### Swap Response (Decline)

This RFC extends the SWAP [reason header](./RFC-002-SWAP.md#reason) with the following possible values when responding with a `RE20 - Declined` status.

#### `timeouts-too-tight`

This indicates to the sender that the difference between `alpha_expiry` and `beta_expiry` is too small and the receiver may accept the swap if they are given more time.

Parameters:

| Name          | JSON Encoding | Description                                                                         |
| ------------- | ------------- | ----------------------------------------------------------------------------------- |
| min-time      | `u32`         | The minimum time difference between the HLTCs in seconds that the receiver requires |

## Execution Phase

After the Setup phase the sender of the request is designated the role Alice while the responder takes the role of Bob.
The execution phase of the protocol takes place exclusively by interacting with the Ledgers.

The protocol is described below as if both parties have immediate access to the most recent state of the ledger and are able to effect persistent changes to it immediately.
For ledgers where recent transactions may be reverted, parties MUST wait until they have confidence that a transaction is permanent before they take any action depending on it.
Parties should also take this into account when choosing or accepting the `alpha_expiry` and `beta_expiry` parameters (see [Security Considerations](#security-considerations)).

Parties MUST verify that the deployed HTLC is exactly what was negotiated during the setup phase.
The HTLC definitions and how to verify them on particular ledgers will be included in subsequent RFCs.

### 1. Alice deploys α-HTLC

Alice starts the execution phase by deploying the α-HTLC to α with the following parameters determined in the setup phase:

  - asset: `alpha_asset`
  - redeem_identity: `alpha_redeem_identity`
  - refund_identity: `alpha_refund_identity`
  - expiry: `alpha_expiry`
  - secret_hash: `secret_hash`

### 2. Bob deploys β-HTLC

When Bob sees that the α-HTLC is deployed on α he decides whether to deploy the β-HTLC or abort the swap.
He MUST make his decision early enough such that he will be able to deploy the β-HTLC before `beta_expiry`.

If he decides to continue with the swap, he deploys β-HTLC to β with the following parameters determined in the setup phase:

  - asset: `beta_asset`
  - redeem_identity: `beta_redeem_identity`
  - refund_identity: `beta_refund_identity`
  - expiry: `beta_expiry`
  - secret_hash: `secret_hash`

If Bob decides to abort the swap, Alice waits until `alpha_expiry` and then MUST activate the refund path of α-HTLC to retrieve **A**.

### 3. Alice redeems

With both HTLCs deployed, Alice decides whether to activate the redeem path of the β-HTLC or abort the swap.
She MUST make her decision early enough such that she is able to activate the redeem path of β-HTLC before `beta_expiry`.
To activate the redeem path she uses her secret and the procedure defined in the specification of β-HTLC.

If Alice attempts redeeming too close to or after `beta_expiry` she risks having Bob cancel the redeem by activating the refund before her.
If Bob does this successfully, he may learn the secret and therefore gain **A** while also having **B** returned to him.

If she decides to abort the swap, Bob waits until `beta_expiry` and then MUST activate the refund path of the β-HTLC.
Alice then waits until `alpha_expiry` and then MUST activate the refund path of α-HTLC.

### 4. Bob redeems

When Bob learns the secret from Alice's redeem activation of β-HTLC he MUST activate the redeem path of α-HTLC and gain ownership of **A**.
He MUST make sure he does this before `alpha_expiry` or risks both losing **B** and not gaining **A**.
To activate the redeem path he uses the secret and the procedure defined in the specification of the α-HTLC.

## Application Considerations

This protocol offers an application the following functionality:

- **Up for Sale**: Alice puts an asset **A** up for sale until `alpha_expiry`.
- **Give Option**: Bob can give Alice an *option* to exchange **A** for his asset **B** until `beta_expiry`
- **Exercise Option**: Alice may exercise her option and receive **B** in exchange for **A** until `beta_expiry`.

It is important to note that Bob gives Alice an option not an *offer*.
He cannot cancel this option; it simply exists until `beta_expiry`.
If **B** declines in value relative to **A** after Bob has deployed β-HTLC Alice may abort the protocol to her own advantage.
Applications where this behaviour is undesirable should either not use this protocol or mitigate the issue within the application in some way.


## Security Considerations

A security model of the protocol and its associated parameters will be included in a later revision of this RFC.

## Registry Extensions

This RFC extends the [registry](./registry.md) in the following ways:

- **identity**: The ledger section now includes an `identity` table which specifies the exact identity to use on a particular ledger.
- **`reason` header**: Adds an additional possible value `timeouts-too-tight` to the `reason` header.
- **Hash Functions**: Adds a new section for listing hash functions and how to refer to them and adds `SHA-256` to this section.

## References
1. https://en.bitcoin.it/wiki/Atomic_swap
2. https://tools.ietf.org/html/rfc4634#section-4.1

# Examples

## Bitcoin to Ethereum

### SWAP REQUEST
``` json
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "bitcoin",
      "parameters": {
        "network": "regtest"
      }
    },
    "beta_ledger": {
      "value": "ethereum",
      "parameters": {
        "network": "ropsten"
      }
    },
    "alpha_asset": {
      "value": "bitcoin",
      "parameters": {
        "quantity": "100000000"
      }
    },
    "beta_asset": {
      "value": "ether",
      "parameters": {
        "quantity": "21000000000000000000"
      }
    },
    "protocol": {
      "value": "comit-rfc-003",
      "parameters": {
          "hash_function": "SHA-256"
      }
    }
  },
  "body": {
    "alpha_expiry": 1547697466,
    "beta_expiry": 1547611066,
    "alpha_refund_identity": "482d8bc0a2f6bb72f38a62bcafb85c2c04b8e9d4",
    "beta_redeem_identity": "0xdaa74f7c9d4894f396860037ca727a2ba726819d",
    "secret_hash": "07954accecf3a81b484b0bbb9cc63034fb6c1037f234288f5d297aa9f669a756"
  }
}
```

### SWAP RESPONSE (accept)

``` json
{
  "status": "OK00",
  "body": {
    "alpha_redeem_identity": "3c045591864c6b52658db072e47a01f658d53e0f",
    "beta_redeem_identity": "0xb39161874921fd329233a9134e537425bb4e72fb"
  }
}
```

### SWAP RESPONSE (reject: `timeouts-too-tight`)

``` json
{
    "status": "RE20",
    "headers" : {
        "reason" : {
            "value" : "timeouts-too-tight",
            "parameters" : {
                "min-time" : 172800,
            }
        }
    }
}
```
