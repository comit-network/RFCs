# Basic HTLC Atomic Swap

## Description

This RFC describes a basic atomic swap protocol and the related parameters required to use it as a COMIT [RFC002](./RFC-002-SWAP.md#protocol) SWAP protocol.
It uses Hash Time Locked Contracts to swap ownership of two assets on different ledgers between two parties.
It is a simplified version of the protocol originally described by TierNolan¹.

## Status

```
RFC-Number: 003
Status: Draft
```

### Identity

This RFC introduces the notion of an "identity".
An identity belongs to a user and represents the minimal information required to transfer an asset to that user on a particular ledger.
On blockchain based ledgers a user's identity is usually some function of a public key where the private key is only known to the user.

This RFC extends the registry to include a section specifying the identity for each supported ledger and any subsequent RFCs adding support for a ledger must include an identity specification.
The identity definition should be usable across protocols.

### Hash Time Lock Contract (HTLC)

The Hash Time Locked Contract (HTLC) is the primary construct used in this protocol. An HTLC locks an asset until one of two possible paths are activated:

- **Activation with secret**: The contract is activated with a *secret*. The hash of the secret must match the hash in the contract.
- **Activation after expiry**: The contract is activated after a time fixed in the contract.

Each activation path transfers the asset to a different party. The parties are decided at the time of contract creation.
In this RFC, the expiry activation returns the asset to the original owner while a secret activation transfers it to the other party.
Therefore this document will refer to these paths as *refund* and *redeem* respectively.

The HTLCs defined in this specification use *absolute time locks* where the expiry is set at a specific time in the future.
Absolute time locks are used because the protocol is only secure if the expiration of the time locks for HTLCs on different chains are fixed relative to each other.
If HTLCs who's duration is relative to their inclusion in the ledger are used an attacker may be able to delay the inclusion of a HTLC onto the ledger and therefore manipulate the relative length of the HTLC time locks.

HTLCs must enforce the length of the secret to be equal to the hash function's output length.
If this is not enforced, a secret may be able to active the redeem path on one HTLC but not on the other.

In this RFC HTLCs are constructed with the following parameters:

  - **asset**: The asset locked in the HTLC.
  - **redeem_identity**: The identity to transfer ownership of the asset upon activation of the redeem path.
  - **refund_identity**: The identity to transfer ownership of the asset upon activation of the refund path.
  - **expiry**: The absolute time after which the refund path may be activated.
  - **secret_hash**: The hash that the hash of the secret must match when activating with the redeem path.
  - **hash_function**: The cryptographic hash function that is used to produce the secret hash.

How to construct an HTLC for each ledger will be defined in subsequent RFCs.

## Setup Phase

In the setup phase the two parties exchange [RFC002](./RFC-002-SWAP.md) SWAP messages to negotiate the parameters to the HTLCs.
The values α, β, **A** and **B** used below refer to the ledgers and assets described by the SWAP headers `alpha_ledger`, `beta_ledger`, `alpha_asset` and `beta_asset` respectively.
Additionally, α-HTLC and β-HTLC refer to the HTLCs deployed on the α and β ledgers.

### SWAP Request Header

The protocol begins with a SWAP request with the `protocol` header's value set to `COMIT-RFC-003` and the following header parameters:

#### `hash_function`
Type: [Hash Function](./COMIT-registry.md#hash-function)

The cryptographic hash function used in the construction of both HTLCs.
It must be available on the `alpha_ledger` and the `beta_ledger`.
This RFC defines `SHA-256`² as an initial value for this parameter.
How to construct HTLCs based on SHA-256 and other hash functions for particular ledgers will be described in subsequent RFCs.

### SWAP Request Body
When `COMIT-RFC-003` is used as the value for `protocol` for a `SWAP REQUEST` message the body must have the following fields:

| Name                    | JSON Encoding       | Description                                                                                               |
|-------------------------|---------------------|-----------------------------------------------------------------------------------------------------------|
| `alpha_expiry`          | `u32`               | The UNIX timestamp of the time-lock on the alpha HTLC                                                     |
| `beta_expiry`           | `u32`               | The UNIX timestamp of the time-lock of the beta HTLC                                                      |
| `alpha_refund_identity` | `α::Identity`       | The identity on α that **A** can be transferred to after `alpha_expiry`                                   |
| `beta_redeem_identity`  | `β::Identity`       | The identity on β that **B** will be transferred to when the β-HTLC is activated with the correct secret. |
| `secret_hash`           | `hex-encoded-bytes` | The output by calling `hash_function` with the secret as input                                            |


### Swap Response (Accept)

If responding with `OK00`, the responder must include the following fields in the response body.

| Name                    | JSON Encoding | Description                                                                                               |
|-------------------------|---------------|-----------------------------------------------------------------------------------------------------------|
| `alpha_redeem_identity` | `α::Identity` | The identity on α that **A** will be transferred to when the α-HTLC is activated with the correct secret. |
| `beta_refund_identity`  | `β::Identity` | The identity on β that **B** will be transferred to when the β-HTLC is activated after `beta_expiry`.     |


### Swap Response (Decline)

This RFC extends the SWAP [reason header](./RFC-002-SWAP.md#reason) with the following possible values when responding with a `RE20 - Declined` status.

#### `timeouts-too-tight`

This indicates to the sender that the difference between `alpha_expiry` and `beta_expiry` is too small and the receiver may accept the swap if they are given more time.

parameters:

| Name          | JSON Encoding | Description                                                                         |
| ------------- | ------------- | ----------------------------------------------------------------------------------- |
| min-time      | `u32`         | The minimum time difference between the HLTCs in seconds that the receiver requires |

## Execution Phase

After the Setup phase the sender of the request is designated the role Alice while the responder takes the role of Bob.
The execution phase of the protocol takes place exclusively by interacting with the Ledgers.

The protocol is described below as if both parties have immediate access to the most recent state of the ledger and are able to effect persistent changes to it immediately.
For ledgers where recent transactions may be reverted, parties must wait until they have confidence that a transaction is permanent before they take any action depending on it.
Parties must also take this into account when choosing or accepting the `alpha_expiry` and `beta_expiry` parameters (see [Security Considerations](#security-considerations)).

In general, to verify a HTLC deployment parties must check that the deployed HTLC is exactly what was negotiated during the setup phase.
The HTLC definitions and how to verify them on particular ledgers will be included in subsequent RFCs.

### 1. Alice deploys α-HTLC

Alice starts the execution phase by deploying the α-HTLC to α with the following parameters determined in the setup phase:

  - asset: `alpha_asset`
  - redeem_identity: `alpha_redeem_identity`
  - refund_identity: `alpha_refund_identity`
  - expiry: `alpha_expiry`
  - secret_hash: `secret_hash`

### 2. Bob deploys β-HTLC

When Bob sees that α-HTLC contract is deployed on α he must decide whether to deploy the β-HTLC and continue execution of the protocol.
He must make his decision early enough such that he will be able to deploy the β-HTLC before `beta_expiry`.

If so he creates β-HTLC with the following parameters determined during the setup phase:

  - asset: `beta_asset`
  - redeem_identity: `beta_redeem_identity`
  - refund_identity: `beta_refund_identity`
  - expiry: `beta_expiry`
  - secret_hash: `secret_hash`

If he decides to abort the protocol, Alice must wait until `alpha_expiry` and then activate the HTLC with refund to retrieve ownership of **A**.

### 3. Alice redeems

With both HTLCs deployed, Alice must decide whether to activate the redeem path and continue execution of the protocol.
She must make her decision early enough such that she is able to activate the redeem path of β-HTLC before `beta_expiry`.
To activate the redeem path she uses her secret and the procedure defined in the specification of the HTLC.

If she decides to abort the protocol, Bob must wait until `beta_expiry` and then activate the refund path of the β-HTLC and Alice must wait until `alpha_expiry` and then activate the refund path of α-HTLC.

### 4. Bob redeems

When Bob learns the secret from Alice's redeem activation of β-HTLC he must activate the redeem path of α-HTLC and gain ownership of **A**.
There is no decision for him to make.
He must make sure he does this before `beta_expiry` or risks both losing **B** and not gaining **A**.
To activate the redeem path he uses the secret and the procedure defined in the specification of the HTLC.

## Application Considerations

This protocol offers an application the following functionality:

- **Up for Sale**: Alice puts an asset **A** up for sale until `alpha_expiry`
- **Give Option**: Bob can give Alice an *option* to exchange for **A** for his asset **B** until `beta_expiry`
- **Exercise Option**: Alice may exercise her option until `beta_expiry` and receive **B** in exchange for **A**.

It is important to note that Bob gives Alice an option not an *offer*.
He cannot cancel this option; it simply exists until `beta_expiry`.
If **A** declines in value relative to **B** after Bob has deployed β-HTLC Alice may abort the protocol to her own advantage.
Applications where this behaviour is undesirable should either not use this protocol or mitigate the issue within the application in some way.


## Security Considerations

A security model of the protocol and its associated parameters will be included in a later revision of this RFC.

## Registry Extensions

This RFC extends the [COMIT-registry](./COMIT-registry.md) in the following ways:

- **identity**: The ledger section now includes an `identity` table which specifies the exact identity to use on a particular ledger.
- **`reason` header**: Adds an additional possible value to the `timeouts-too-tight` to the `reason` header.
- **Hash Functions**: Adds a new section for listing hash functions and how to refer to them and adds `SHA-256` to this section.

--
1. https://en.bitcoin.it/wiki/Atomic_swap
2. https://tools.ietf.org/html/rfc4634#section-4.1
