# Ether Basic HTLC Atomic Swap

- RFC-Number: 007
- Status: Draft
- Discussion-issue: [#48](https://github.com/comit-network/RFCs/issues/48)
- Created on: 2019-03-12

**Table of Contents**
- [Description](#description)
- [The Ethereum Identity](#the-ethereum-identity)
- [Hash Time Lock Contract](#hash-time-lock-contract)
    - [Hash Functions](#hash-functions)
    - [Parameters](#parameters)
    - [Contract](#contract)
- [Execution Phase](#execution-phase)
    - [Deployment](#deployment)
    - [Redeem](#redeem)
    - [Refund](#refund)
- [Registry extension](#registry-extension)
- [RFC003 SWAP REQUEST](#rfc003-swap-request)
- [RFC003 SWAP RESPONSE](#rfc003-swap-response)
- [HTLC](#htlc)

## Description

This RFC defines how to execute a [RFC003](./RFC-003-SWAP-Basic.md) SWAP where
one of the ledgers is Ethereum and the associated asset is the native Ether
asset.

For definitions of the Ethereum ledger and Ether asset see
[RFC006](./RFC-006-Ethereum.md).

To fulfil the requirements of [RFC003](./RFC-003-SWAP-Basic.md) this RFC
defines:

- The [identity](./RFC-003-SWAP-Basic.md#identity) to be used when negotiating a
  SWAP on the Ethereum ledger.

- How to construct a Hash Time Lock Contract (HTLC) to lock the Ether asset on
  the Ethereum blockchain.

- How to deploy, redeem and refund the HTLC during the execution phase of
  [RFC003](./RFC-003-SWAP-Basic.md).

## The Ethereum Identity

[RFC003](./RFC-003-SWAP-Basic.md) requires ledgers have an *identity* type
specified to negotiate a SWAP.

The identity to be used on Ethereum is the Ethereum *address* as defined in
equation 284 of the
[Ethereum Yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf): the
right most 160-bits (20 bytes) of the Keccak-256 hash of the corresponding ECDSA
public key.

In the JSON encoding, an ethereum address MUST be encoded as this 20 byte hex
string prefixed by `0x` (as is standard in the Ethereum ecosystem).
Furthermore, implementations MUST also accept
[EIP50](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md) mixed case
addresses and MAY verify the checksum.

This RFC extends the [registry](./registry.md) with the following entry in the
identity table:

| Ledger   | Identity Name | JSON Encoding         | Description         |
|:---------|:--------------|:----------------------|---------------------|
| Ethereum | `address`     | `0x` prefixed address | An Ethereum Address |

Note that since *contract* addresses and *user* addresses are indistinguishable,
a contract address could be used as an identity.  RFCs that use the Ethereum
identity defined here should explain the impact (if any) this has on the
protocol.

## Hash Time Lock Contract

### Hash Functions

This RFC specifies SHA-256 as the only value the `hash_function` parameter to
`comit-rfc-003` may take if Ethereum is used as a ledger.  This may be expanded
in subsequent RFCs.

### Parameters

The parameters for the Ether HTLC follow
[RFC003](./RFC-003-SWAP-Basic.md#hash-time-lock-contract-htlc) and are described
concretely in the following table:

| Variable        | Description                                                                 |
|:----------------|:----------------------------------------------------------------------------|
| asset           | The quantity of wei                                                         |
| secret_hash     | The hash of the `secret` (32 bytes for SHA-256)                             |
| redeem_identity | The `address` of the redeeming party                                        |
| refund_identity | The `address` of the refunding party                                        |
| expiry          | The absolute UNIX timestamp in seconds after which the HTLC can be refunded |

### Contract

This RFC defines a single disposable contract as the HTLC.  It uses the
`selfdestruct` EVM opcode to release the funds to the intended party.

This approach was chosen over a stateful contract written in solidity so that
the contract can be precisely defined, verified and updated.

The `redeem_identity` and `refund_identity` are hard coded into the contract.
If called with the correct `secret` as the calldata it will transfer all the
Ether to the `redeem_identity` using `selfdestruct`.  If called without any
calldata after the `expiry` it will transfer all the Ether to the
`refund_identity` using `selfdestruct`.

The contract never checks the caller so the redeem and refund transactions can
be sent by anyone.  The funds will only ever be transferred to the
`redeem_identity` or the `refund_identity`.  No special considerations need to
be made when either identity is a contract address as they simply receive funds.

The contract is compiled from the following EVM assembly:

``` asm
{
    // Load received secret size
    calldatasize

    // Check if secret is zero length
    iszero

    // If secret is zero length, jump to branch that checks if expiry time has been reached
    check_expiry
    jumpi

    // Load expected secret size
    32

    // Load received secret size
    calldatasize

    // Compare secret size
    eq
    iszero

    // If passed secret is wrong size, jump to exit contract
    exit
    jumpi

    // Load secret into memory
    calldatacopy(0, 0, 32)

    // Hash secret with SHA-256 (pre-compiled contract 0x02)
    call(72, 0x02, 0, 0, 32, 33, 32)

    // Placeholder for correct secret hash
    <secret_hash>

    // Load hashed secret from memory
    mload(33)

    // Compare hashed secret with existing one
    eq

    // Combine `eq` result with `call` result
    and

    // Jump to redeem if hashes match
    redeem
    jumpi

    // Exit if hashes don't match
    return(0, 0)

check_expiry:
    // Timestamp of the current block in seconds since the epoch
    timestamp

    // Placeholder for refund timestamp
    <expiry>

    // Compare refund timestamp with current timestamp
    lt

    // Jump to refund if time is expired
    refund
    jumpi

exit:
    // Exit
    return(0, 0)

redeem:
    log1(0, 32, 0xB8CAC300E37F03AD332E581DEA21B2F0B84EAAADC184A295FEF71E81F44A7413) // log keccak256("Redeemed()")
    selfdestruct(<redeem_identity>)

refund:
    log1(0, 0, 0x5D26862916391BF49478B2F5103B0720A842B45EF145A268F2CD1FB2AED55178) // log keccak256("Refunded()")
    selfdestruct(<refund_identity>)
}

```

The following is the contract deployment code template encoded as hex which will deploy the above contract when the placeholder values are replaced with the parameters.

```
6100dc61000f6000396100dc6000f336156051576020361415605c57602060006000376020602160206000600060026048f17f1000000000000000000000000000000000000000000000000000000000000001602151141660625760006000f35b42632000000210609f575b60006000f35b7fb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a741360206000a1733000000000000000000000000000000000000003ff5b7f5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed5517860006000a1734000000000000000000000000000000000000004ff
```

To compile the contract code replace the placeholder values with the HTLC parameters at the following offsets:


| Name               | Byte Range | Length (bytes) |
| :---               | :---       | :---           |
| `secret_hash`      | 51..83     | 32             |
| `refund_timestamp` | 99..103    | 4              |
| `redeem_address`   | 153..173   | 20             |
| `refund_address`   | 214..234   | 20             |

The contract emits two logs:

| Topic        | keccack256(Topic)                                                  | Data     | Emitted When             |
|:-------------|:-------------------------------------------------------------------|----------|:-------------------------|
| `Redeemed()` | 0xb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a7413 | `secret` | The contract is redeemed |
| `Refunded()` | 0x5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed55178 | -        | The contract is refunded |


Implementations SHOULD use the following gas limits on the transactions related to the contract:

| Transaction | recommended gas limit |
|:------------|-----------------------|
| Deployment  | 121,800               |
| Redeem      | 100,000               |
| Refund      | 100,000               |

## Execution Phase

The following section describes how both parties should interact with the
Ethereum blockchain during the
[RFC003 execution phase](./RFC-003-SWAP-Basic.md#execution-phase).

### Deployment

At the start of the deployment stage, both parties compile the contract code as
described in the previous section.  We will call this value `contract_code`.

To deploy the Ether HTLC, the *funder* must deploy the `contract_code`.  They
SHOULD do this by sending a contract deployment transaction to the relevant
Ethereum blockchain with:

- `contract_code` as the `data` of the transaction

- The quantity of wei specified in the Ether Asset header as the `value` of the
  transaction

The funder SHOULD NOT do this through another contract, i.e. by executing the
`CREATE` opcode.

To be notified of the deployment event, both parties MAY watch the blockchain
for a transaction with the `contract_code` as the data.  Upon observing the
deployment transaction, both parties SHOULD record the address the contract was
deployed to (referred to as `contract_address` from now on).

### Redeem

Before redeeming, the redeemer SHOULD wait until the deployment transaction has
enough confirmation such that they consider it permanent.

To redeem the HTLC, the redeemer SHOULD send a transaction to the
`contract_address` with the `data` of the transaction set to the `secret`.

To be notified of the redeem event, both parties SHOULD watch the blockchain for
`contract_address` emitting the `Redeemed()` log.  If Ethereum is the
`beta_ledger` (see [RFC003](./RFC-003-SWAP-Basic.md#execution-phase)), then the
funder MUST watch for such a log, extract the `secret` from the transaction
receipt and continue the protocol.  In this case, Bob MUST NOT watch for a
transaction sent to `contract_address` with the `secret` as `data`.  This would
be insufficient, because he would miss learning the `secret` if the contract is
redeemed by a call from another contract (rather than from a transaction).

### Refund

To refund the HTLC, the funder MUST send a transaction to `contract_address`
with empty data in a block with timestamp greater than `expiry`.

To be notified of the refund event, both parties SHOULD watch the blockchain for
`contract_address` emitting the `Refunded()` log.

## Registry extension

This RFC extends the [registry](./registry.md#identities) with an identity
definition for the Ethereum ledger:

| Ledger   | Identity Name | JSON Encoding         | Description         |
|:---------|:--------------|:----------------------|---------------------|
| Ethereum | `address`     | `0x` prefixed address | An Ethereum Address |

# Examples/Test vectors

## RFC003 SWAP REQUEST

The following shows an [RFC003](RFC-003-SWAP-Basic.md) SWAP REQUEST where the
`alpha_ledger` is Ethereum, the `alpha_asset` is 1 Ether (with `...` being used
where the value is only relevant for the `beta_ledger`).

``` json
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "ethereum",
      "parameters": { "network": "mainnet" }
    },
    "beta_ledger": {...},
    "alpha_asset": {
      "value": "ether",
      "parameters": { "quantity": "1000000000000000000" }
    },
    "beta_asset": {...},
    "protocol": {
        "value" : "comit-rfc-003",
        "parameters" : { "hash_function" : "SHA-256" }
    }
  },
  "body": {
    "aplha_ledger_refund_identity": "0x0f59e9e105be01d5e2206792a267406f255c5ea5",
    "alpha_expiry": 1552263040,
    "secret_hash" : "ac5a18da6431ed256965b873ef49dc15a70a0a66e2d28d0c226b5db040123727",
    "beta_ledger_redeem_identity" : "...",
    "beta_expiry" : ...
  },
}
```

Note, the pre-image of `secret_hash` is
`51a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c`.

## RFC003 SWAP RESPONSE

A valid `RESPONSE` to the above `REQUEST` could look like:

``` json
{
  "status" : "OK00",
  "body": {
     "alpha_ledger_redeem_identity": "0x53fd2cac865d3aa1ad6fbdebaa00802c94239fba",
     "beta_ledger_refund_identity": "..."
  }
}
```

## HTLC

The above `REQUEST` and `RESPONSE` results in the following parameters to the HTLC:

| Parameter       | value                                                              |
|:----------------|--------------------------------------------------------------------|
| redeem_identity | `0x53fd2cac865d3aa1ad6fbdebaa00802c94239fba`                       |
| redund_identity | `0x0f59e9e105be01d5e2206792a267406f255c5ea5`                       |
| secret_hash     | `ac5a18da6431ed256965b873ef49dc15a70a0a66e2d28d0c226b5db040123727` |
| expiry          | 1552263040                                                         |

Which should compile to the following `contract_code`:

```
6100dc61000f6000396100dc6000f336156051576020361415605c57602060006000376020602160206000600060026048f17fac5a18da6431ed256965b873ef49dc15a70a0a66e2d28d0c226b5db040123727602151141660625760006000f35b42635c85a78010609f575b60006000f35b7fb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a741360206000a17353fd2cac865d3aa1ad6fbdebaa00802c94239fbaff5b7f5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed5517860006000a1730f59e9e105be01d5e2206792a267406f255c5ea5ff
```
