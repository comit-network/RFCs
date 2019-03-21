# ERC20 Basic HTLC Atomic Swap

- RFC-Number: 009
- Status: Draft
- Discussion-issue: -
- Created on: 2019-03-12

## Description

This RFC defines how to execute a [RFC003](./RFC-003-SWAP-basic.md) SWAP where one of the ledgers is Ethereum and the associated asset is an ERC20 token.

The definition of the Ethereum ledger was introduced in [RFC006](./RFC-006-SWAP-Ethereum.md).
The definition of ERC20 token asset was introduced in [RFC008](./RFC-008-ERC20.md).

To fulfil the requirements of [RFC003](./RFC-003-SWAP-basic.md) this RFC defines:

- How to construct a Hash Time Lock Contract (HTLC) to lock an ERC20 token asset on the Ethereum blockchain.
- How to deploy, redeem and refund the HTLC during the execution phase of [RFC003](./RFC-003-SWAP-basic.md).

## Hash Time Lock Contract

### Hash Functions

SHA-256 is the only value the `hash_function` parameter to `comit-rfc-003` may take if this HTLC is used.
Future RFCs may make modifications to the HTLC to allow other hash functions.

### Parameters

The parameters for the ERC20 HTLC follow [RFC003](./RFC-003-SWAP-basic.md#hash-time-lock-contract-htlc) and are described concretely in the following table:

| Parameter       | Description                                                                              |
|:----------------|:-----------------------------------------------------------------------------------------|
| asset           | The quantity of the token to be locked in the HTLC AND the address of the token contract |
| secret_hash     | The hash of the `secret` (32 bytes for SHA-256)                                          |
| redeem_identity | The Ethereum address of the redeeming party                                              |
| refund_identity | The Ethereum address` of the refunding party                                             |
| expiry          | The absolute UNIX timestamp in seconds after which the HTLC can be refunded              |

We will refer to the token contract address and quantity from the [RFC008](./RFC-008-ERC20.md) ERC20 asset definition as `token_contract` and `token_quantity` respectively.

### Contract

This RFC defines a single disposable contract as the HTLC.

Note that the funding of the HTLC is a two step process:

- The funder deploys the HTLC contract
- The funder transfers tokens to the HTLC contract

It is designed like this because it is impossible to deploy an HTLC and transfer a ownership of ERC20 tokens to that HTLC in one transaction.

The contract uses the [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) `transfer` call to transfer ownership of the tokens

The contract never checks the caller so the redeem and refund transactions can be sent by anyone.
The token will only ever be transferred to the `redeem_identity` or the `refund_identity`.

The contract is compiled from the following EVM assembly with placeholder values marked by `<...>`:

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
    log1(0, 32, 0xB8CAC300E37F03AD332E581DEA21B2F0B84EAAADC184A295FEF71E81F44A7413) // log keccak256(Redeemed())
    mstore(32, <redeem_identity>) // redeem address
    finishTransferTokens
    jump

refund:
    log1(0, 0, 0x5D26862916391BF49478B2F5103B0720A842B45EF145A268F2CD1FB2AED55178) // log keccak256(Refunded())
    mstore(32, <refund_identity>) // refund address
    finishTransferTokens
    jump

finishTransferTokens:
    mstore(0, 0xa9059cbb) // first 4bytes of keccak256("transfer(address,uint256)")
    mstore(64, <token_quantity>) // Amount
    call(
      sub(gas,100000),
      <token_contract>, // Token Contract address
      0,  // Ether to transfer
      28, // = 32-4
      68, // = 2*32+4
      96, // return location
      32  // return size
    )
    pop

    selfdestruct(mload(32))
}
```

The following is the contract deployment code encoded template as hex which will deploy the above contract when the placeholder values (set to `00`s) are replaced with the parameters.


```
61014461000f6000396101446000f3361561005457602036141561006057602060006000376020602160206000600060026048f17f000000000000000000000000000000000000000000000000000000000000000060215114166100665760006000f35b426300000000106100a9575b60006000f35b7fb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a741360206000a17300000000000000000000000000000000000000006020526100ec565b7f5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed5517860006000a17300000000000000000000000000000000000000006020526100ec565b63a9059cbb6000527f0000000000000000000000000000000000000000000000000000000000000064604052602060606044601c6000730000000000000000000000000000000000000000620186a05a03f150602051ff
```

To compile the contract code replace the placeholder values with the HTLC paramters at the following offsets:

| Data              | Position of first byte | Position of last byte | Length (bytes) |
|:------------------|:-----------------------|:----------------------|:---------------|
| `secret_hash`     | 53                     | 84                    | 32             |
| `expiry`          | 102                    | 105                   | 4              |
| `redeem_identity` | 157                    | 176                   | 20             |
| `refund_identity` | 224                    | 253                   | 20             |
| `token_quantity`  | 261                    | 292                   | 32             |
| `token_contract`  | 307                    | 338                   | 32             |



The contract emits two logs:

| Topic        | keccack256(Topic)                                                  | Data     | Emitted When             |
|:-------------|:-------------------------------------------------------------------|----------|:-------------------------|
| `Redeemed()` | 0xb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a7413 | `secret` | The contract is redeemed |
| `Refunded()` | 0x5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed55178 | -        | The contract is refunded |


Implementations SHOULD use the following gas limits on the transactions related to the contract:

| Transaction | Recommended gas limit |
|:------------|-----------------------|
| Deployment  | 167,800               |
| Funding     | 100,000               |
| Redeem      | 100,000               |
| Refund      | 100,000               |


## Execution Phase

The following section describes how both parties should interact with the Ethereum blockchain during the [RFC003 execution phase](./RFC-003-SWAP-basic.md#execution-phase).

Note that with the ERC20 HTLC there's a *funding* stage after the *deployment* stage.
The funding stage MUST be completed before the redeemer tries to redeem.


### Deployment

At the start of the deployment stage, both parties compile the contract code as described in the previous section.
We will call this value `contract_code`.

To deploy the ERC20 HTLC, the *funder* must deploy the `contract_code`.
They SHOULD do this by sending a contract deployment transaction to the relevant Ethereum blockchain with `contract_code` as the `data` of the transaction.


To be notified of the deployment event, both parties MAY watch the blockchain for a transaction with the `contract_code` as the data.
Upon observing the deployment transaction, both parties SHOULD record the address the contract was deployed to (referred to as `htlc_contract` from now on).


### Funding

After deploying the contract, the funder must transfer ownership of the `token_quantity` to the `htlc_contract`.
They MUST do this by calling the `transfer(htlc_contract,token_quantity)` function on the `token_contract`.
This can be done by sending a transaction to the `token_contract` with the following data:

```
a9059cbb <20-byte htlc_contract> <32-byte token_quantity>
```

To be notified of the funding event, both parties MAY watch the blockchain for `token_contract` emitting the `Transfer(address,address,uint256)` log where the second `address` argument is the `htlc_contract`.
(The keccack256 topic filter for the log is: `0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef`).

The redeemer MUST check the transaction receipt that the amount transferred to `token_contract` was at least `token_quantity`.

### Redeem

Before redeeming, the redeemer SHOULD wait until the funding transaction has enough confirmation such that they consider it permanent.

To redeem the HTLC, the redeemer SHOULD send a transaction to the `htlc_contract` with the `data` of the transaction set to the `secret`.

To be notified of the redeem event, both parties SHOULD watch the blockchain for `htlc_contract` emitting the `Redeemed()` log.
If Ethereum is the `beta_ledger`, then the funder (Bob) MUST watch for such a log, extract the `secret` from the transaction receipt and continue the protocol.
In this case, Bob MUST NOT watch for a transaction sent to `htlc_contract` with the `secret` as `data`.
If he does this, he will miss learning the `secret` if the contract is redeemed by a call from another contract (rather than from a transaction).

### Refund

To refund the HTLC, the funder MUST send a transaction to `htlc_contract` with empty data in a block with timestamp greater than `expiry`.

To be notified of the refund event, both parties SHOULD watch the blockchain for `htlc_contract` emitting the `Refunded()` log.

# Examples

## RFC003 SWAP REQUEST

The following shows an [RFC003](RFC-003-SWAP-basic.md) SWAP REQUEST where the `alpha_ledger` is Ethereum, the `alpha_asset` is 1 PAY token (with `...` being used where the value is only relevant for the `beta_ledger`).

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
      "value": "erc20",
      "parameters": {
          "quantity": "1000000000000000000",
          "address": "0xb97048628db6b661d4c2aa833e95dbe1a905b280"
      }

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

Note, the pre-image for the `secret_hash` is `adebd583a094215e963ebe4a1474b9bb4bf48167e64f3f474d71e41a75494bbb`.

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
| token_contract  | `0xb97048628db6b661d4c2aa833e95dbe1a905b280`                       |
| token_quantity  | 1000000000000000000                                                |
| secret_hash     | `ac5a18da6431ed256965b873ef49dc15a70a0a66e2d28d0c226b5db040123727` |
| expiry          | 1552263040                                                         |

Which should compile to the following `contract_code`:

```
61014461000f6000396101446000f3361561005457602036141561006057602060006000376020602160206000600060026048f17fac5a18da6431ed256965b873ef49dc15a70a0a66e2d28d0c226b5db04012372760215114166100665760006000f35b42635c85a780106100a9575b60006000f35b7fb8cac300e37f03ad332e581dea21b2f0b84eaaadc184a295fef71e81f44a741360206000a17353fd2cac865d3aa1ad6fbdebaa00802c94239fba6020526100ec565b7f5d26862916391bf49478b2f5103b0720a842b45ef145a268f2cd1fb2aed5517860006000a1730f59e9e105be01d5e2206792a267406f255c5ea56020526100ec565b63a9059cbb6000527f0000000000000000000000000000000000000000000000000de0b6b3a7640000604052602060606044601c600073b97048628db6b661d4c2aa833e95dbe1a905b280620186a05a03f150602051ff
```
