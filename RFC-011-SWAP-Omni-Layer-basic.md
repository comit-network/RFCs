# Omni Layer Basic HTLC Atomic Swap

- RFC-Number: 011
- Status: Draft
- Discussion-issue: -
- Created on: 27 Mar. 2019

**Table of Contents**

- [Description](#description)
- [Limitations](#limitations)
- [Ownership](#ownership)
- [Inputs & Outputs format](#inputs--outputs-format)
  * [Omni Layer data](#omni-layer-data)
  * [Dust](#dust)
  * [Deployment transaction](#deployment-transaction)
  * [Redeem transaction](#redeem-transaction)
  * [Refund transaction](#refund-transaction)
- [Examples/Test vectors](#examplestest-vectors)
  * [RFC003 SWAP REQUEST](#rfc003-swap-request)
  * [RFC003 SWAP RESPONSE](#rfc003-swap-response)
  * [HTLC](#htlc)

## Description

This RFC defines how to execute a [RFC003](./RFC-003-SWAP-basic.md) SWAP where one of the ledgers is Bitcoin and the associated asset is an Omni Layer token asset.

For the definition of the Bitcoin ledger see [RFC004](./RFC-004-SWAP-Bitcoin.md).
For the definition of the Omni Layer token asset see [RFC010](./TODO).

This RFC re-purposes the Hash Time Locked Contract (HTLC) Bitcoin script from [RFC005](./RFC-005-SWAP-Bitcoin-basic.md) to lock Omni Layer tokens instead of Bitcoin.

To fulfil the requirements of [RFC003](./RFC-003-SWAP-basic.md) this RFC defines:

- How to construct Omni layer transactions transferring Omni Layer tokens to a HTLC address
- how to deploy, redeem and refund the HTLC during the execution phase of [RFC003](./RFC-003-SWAP-basic.md)

## Constructing an Omni Layer Transaction

This sections describes how to build Omni Layer transactions for the purpose of a basic swap.
Information from the [Omni Layer Spec](https://github.com/OmniLayer/spec) has been transcribed here to facilitate the description of a Omni Layer Basic HTLC Atomic Swap.

This RFC only supports the [simple send](https://github.com/OmniLayer/spec#transfer-coins-simple-send) Omni Layer transaction type, encoded using Class C format.
Class C transactions uses Bitcoin `OP_RETURN` instruction to store the Omni Layer data.

For clarity purpose, an *input, output owning an Omni Layer token* refers to the fact that such input or output matches an address that owns such token.
See [RFC-010](TODO#ownership) for more details.

### Omni Layer data

The Omni Layer data to be included in `OP_RETURN` is as per the Omni Layer protocol a simple send transaction type.

Such data MAY be constructed manually or the implementer MAY decide to relay on an existing Omni Layer implementation to generate it.
Reference to construct manually:
- [Simple Send Payload](https://github.com/OmniLayer/spec#transfer-coins-simple-send)
- [Encode Class C payload](https://github.com/OmniLayer/omnicore/blob/4c9abdecf56c78e3533a64e8034f920cb1d43eb4/src/omnicore/encoding.cpp#L83)
- [Omni prefix for Class C](https://github.com/OmniLayer/omnicore/blob/4c9abdecf56c78e3533a64e8034f920cb1d43eb4/src/omnicore/omnicore.cpp#L2040)


For example, the following omnicore API MAY be used:
- `property_id`: the property id of the Omni Layer Asset (e.g. `31`)
- `amount`: the quantity of Omni Layer Assets to transfer (e.g. `20`)
- `payload`: the hex-encoded Omni Layer Data (e.g. `000000000000001f0000000077359400`)

#### Example
```
$ omnicore-cli "omni_createpayload_simplesend" 31 "20"
000000000000001f0000000077359400
```
To then embed the Omni Layer Data received from omnicore in an `OP_RETURN` output, it MUST be prefixed it with the Omni Layer header: `6f6d6e69`.

### Dust

Bitcoin Core has limitations on minimum amounts owned by a UTXO. The minimum value is 546 satoshis for a normal transaction and 294 satoshis for SegWit transactions.
See [RFC-010](TODO#Dust) for details.

The values above will be referred as `min_sat` from now on.

`OP_RETURN` outputs are not spendable and hence SHOULD not be assigned any Bitcoin value. Only spendable outputs MUST have at least `min_sat` Satoshis assigned.

### HTLC Address

The HTLC address MUST either be:
1. the P2WSH address of the HTLC, as described in [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md)
2. the P2SH of the P2WSH above, as descriped in [BIP-0141 Segregated Witness](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)

(1) is preferred.
However, not all Omni Layer wallets implementation support bech32 addresses.
Hence, (2) can be used as a fallback.

The simple send MUST NOT be done to the direct P2SH address of the HTLC.

This choice has been made to:
- limit the number of addresses to watch, facilitating implementation
- avoid defining the basic HTLC twice (once, in P2SH format, once in P2WSH format)
- follow the common strategy for segwit backward compatibility (nesting in P2SH)

### Inputs & Outputs format to spend the HTLC

This section describes specific inputs & outputs to construct an Omni Layer transaction for the *redeeming* and *refunding* of an HTLC.
This is to facilitate the constructions and filtering of such transactions for the purpose of a basic swap.

To construct a transaction to spend from the HTLC, the following information is needed:

| Variable           | Description                                                                                                 |
|:---                |:---                                                                                                         |
| `htlc_utxo`        | UTXO locked by the HTLC, created with the deployment transaction                                            |
| `to_address`       | Address to which the Omni Layer token is sent out of the HTLC                                               |
| `change_address`   | Address to which the Bitcoin change will be sent upon redeeming; MUST be different than `to_address`        |
| `change_utxo`      | UTXO locked by `change_address` that owns some Bitcoin asset to pay for the mining fee                      |
| `init_change_sat`  | Amount of Satoshis owned by `change_utxo` before the redeem                                                 |
| `mining_sat`       | The amount of Satoshi to be spent on mining fees                                                            |
| `omni_data`        | The Omni Layer data that transfers the Omni Layer Asset as described in [Omni Layer data](#omni-layer-data) |

 The Bitcoin transaction MUST be constructed with the following inputs, outputs and values.
 The order of the inputs and outputs MUST be as described.
 The *assigned Bitcoin value* on the inputs SHOULD be as described.

 | Inputs (assigned Bitcoin value)      | Outputs (assigned Bitcoin value)                     |
 |:---                                  |:---                                                  |
 | 1. `htlc_utxo` (`min_sat`)           | 1. `to_address` (`min_sat`)                          |
 | 2. `change_utxo` (`init_change_sat`) | 2. `change_address` (`init_change_sat - mining_sat`) |
 |                                      | 3. `OP_RETURN omni_data` (`0`)                       |

The implementor MAY include output (2) `change_address` or MAY decide to omit it and transfer the Bitcoin change to `to_address`.
if output (2) `change_address` is present then input (2) `change_utxo` MUST be included to ensure the Omni Layer Assets are sent to output (1) `to_address`.


## Execution Phase

The following section describes how both parties should interact with the Bitcoin blockchain during the [RFC003 execution phase](./RFC-003-SWAP-basic.md#execution-phase).


### Deployment transaction

At the start of the deployment stage, both parties compile the contract as described in the previous section.
We will call this value `contract_script`.

To deploy the Bitcoin HTLC, the *funder* must confirm an Omni Layer transaction on the relevant Bitcoin blockchain.
The transaction must have the following properties:

- It MUST be a `valid` Omni Layer transaction.
- It MUST be simple send Omni Layer transaction Class C.
- The simple send `amount` MUST be equal to the `quantity` parameter in the Omni Layer asset header.
- It MUST have an output locked with either
  - a Pay-To-Witness-Script-Hash (P2WSH) `scriptPubKey` derived from `contract_script` (See [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#specification) for how to construct the `scriptPubkey` from the `contract_script`).
  - OR a Pay-To-Witness-Script-Hash nested in Pay-To-Script-Hash (P2WSH(P2SH)) `scriptPubKey` derived from `contract_script` (See [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#p2wsh-nested-in-bip16-p2sh) for how to construct the `scriptPubkey` from the `contract_script`).
- The simple send `recipient` MUST be confirmed as a correct HTLC address, as defined in [HTLC Address](#htlc-address).


Both parties MAY watch the blockchain for the deployment transaction, they MUST consider both (1) and (2) addresses.

Such party MUST check the validity of the transaction against Omni Layer Spec.
Not all valid Bitcoin transactions containing Omni Data are valid Omni Layer transactions.

For example, the following RPC call to omnicore can be used to verify the validity of a deployment transaction:
- `txid`: the transaction id of the transaction (e.g. `72abc494e877d7e2491ffa6fea3694648aa75e6166e42d6381b04d41ca933fb5`)
- `referenceaddress`: the HTLC address (e.g. `2N4CHfBpitHNXj9nBcJCFs7BXUaoNVTegBZ`)
- `valid`: MUST be `true`
- `amount`: the quantity of asset (e.g. `5300`)

 ```
$ omnicore-cli "omni_gettransaction" "72abc494e877d7e2491ffa6fea3694648aa75e6166e42d6381b04d41ca933fb5"  
{
  "txid": "72abc494e877d7e2491ffa6fea3694648aa75e6166e42d6381b04d41ca933fb5",
  "fee": "0.00003046",
  "sendingaddress": "mxuetdW3Yq8KEsvB4L37Z5jmMQMTxZF9Be",
  "referenceaddress": "2N4CHfBpitHNXj9nBcJCFs7BXUaoNVTegBZ",
  "ismine": false,
  "version": 0,
  "type_int": 0,
  "type": "Simple Send",
  "propertyid": 2147483652,
  "divisible": false,
  "amount": "5300",
  "valid": true,
  "blockhash": "6e9976c5d6c2b4be2928972c93564a0fc0ccdabe4ac581378fad65753559d0b0",
  "blocktime": 1556091362,
  "positioninblock": 1,
  "block": 441,
  "confirmations": 6
}
```

### Redeem transaction

To redeem the HTLC, the redeemer MUST submit a valid Omni Layer transaction transferring all the omni tokens from the HTLC address to their desired address using simple send.
The transaction should be constructed according to the [Constructing an Omni Layer Transaction](#constructing-an-omni-layer-transaction) section.
To construct the witness data for the P2WSH output, the redeemer MUST use the procedure described in the [Redeem section of RFC005](./RFC-005-SWAP-Bitcoin-basic.md#redeem).

Both parties SHOULD watch the blockchain for the a bitcoin transaction, that spends the HTLC output and is a valid Omni Layer transaction.
Note that the redeemer MAY (but should not) use another type of Omni Layer transaction than simple send, hence monitoring MUST NOT be exclusive to simple send.

For example, the following RPC call to omnicore can be used to verify the validity of a redeem transaction:
- `txid`: the transaction id of the transaction (e.g. `ca30ac83bee95ea6191e74b96a8eae11d88eca8abbb55d688e0617a0b84f18d2`)
- `referenceaddress`: the redeemer address (e.g. `mh9g3jCJxkc4tzV88THmQHGNGiCzUZ1zg6`)
- `sendingaddress`: the HTLC address (e.g. `2N4CHfBpitHNXj9nBcJCFs7BXUaoNVTegBZ`)
- `valid`: MUST be `true`
- `amount`: the quantity of asset (e.g. `5300`)

```
$ omnicore-cli "omni_gettransaction" "ca30ac83bee95ea6191e74b96a8eae11d88eca8abbb55d688e0617a0b84f18d2"  
{
  "txid": "ca30ac83bee95ea6191e74b96a8eae11d88eca8abbb55d688e0617a0b84f18d2",
  "fee": "0.00002500",
  "sendingaddress": "2N4CHfBpitHNXj9nBcJCFs7BXUaoNVTegBZ",
  "referenceaddress": "mh9g3jCJxkc4tzV88THmQHGNGiCzUZ1zg6",
  "ismine": false,
  "version": 0,
  "type_int": 0,
  "type": "Simple Send",
  "propertyid": 2147483652,
  "divisible": false,
  "amount": "5300",
  "valid": true,
  "blockhash": "7e94a2a8eb188f2e08020ecea4aa30959525de2040fb75bc7911b3988f2ad7df",
  "blocktime": 1556091006,
  "positioninblock": 1,
  "block": 442,
  "confirmations": 5
}
```

### Refund transaction

To refund1 the HTLC, the funder MUST submit a valid Omni Layer transaction transferring all the omni tokens from the HTLC address to their desired address using simple send.
The transaction should be constructed according to the [Constructing an Omni Layer Transaction](#constructing-an-omni-layer-transaction) section.
To construct the witness data for the P2WSH output, the funder MUST use the procedure described in the [Redeem section of RFC005](./RFC-005-SWAP-Bitcoin-basic.md#redeem).

Both parties SHOULD watch the blockchain for the a bitcoin transaction, that spends the HTLC output and is a valid Omni Layer transaction.
Note that the funder MAY (but should not) use another type of Omni Layer transaction than simple send, hence monitoring MUST NOT be exclusive to simple send.

## Examples/Test vectors

### RFC003 SWAP REQUEST

The following shows an [RFC003](RFC-003-SWAP-basic.md) SWAP REQUEST where the `alpha_ledger` is Bitcoin, the `alpha_asset` is 1 Omni Layer Asset TetherUS (with `...` being used where the value is only relevant for the `beta_ledger`).

``` json
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "bitcoin",
      "parameters": { "network": "mainnet" }
    },
    "beta_ledger": {...},
    "alpha_asset": {
      "value": "omni",
      "parameters": {
        "quantity": "100000000",
        "property_id": 31,
      }
    },
    "beta_asset": {...},
    "protocol": {
        "value" : "comit-rfc-003",
        "parameters" : { "hash_function" : "SHA-256" }
    }
  },
  "body": {
    "alpha_ledger_refund_identity": "1925a274ac004373bb5429553bdb55c40e57b124",
    "alpha_expiry": 1552263040,
    "secret_hash" : "1f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e",
    "beta_ledger_redeem_identity" : "...",
    "beta_expiry" : ...
  },
}
```

Note, the property id of TetherUS is `31` and TetherUS is divisible;
the secret for the `secret_hash` is `51a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c`.

### RFC003 SWAP RESPONSE
A valid `RESPONSE` to the above `REQUEST` could look like:

``` json
{
  "status" : "OK00",
  "body": {
     "alpha_ledger_redeem_identity": "c021f17be99c6adfbcba5d38ee0d292c0399d2f5",
     "beta_ledger_refund_identity": "..."
  }
}
```


### HTLC

The above `REQUEST` and `RESPONSE` results in the following parameters to the HTLC:

| Parameter       | value                                                              |
|:----------------|--------------------------------------------------------------------|
| redeem_identity | `c021f17be99c6adfbcba5d38ee0d292c0399d2f5`                         |
| refund_identity | `1925a274ac004373bb5429553bdb55c40e57b124`                         |
| secret_hash     | `1f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e` |
| expiry          | 1552263040                                                         |


Which compiles into the following Bitcoin script bytes:

```
6382012088a8201f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e8876a914c021f17be99c6adfbcba5d38ee0d292c0399d2f5670480a7855cb17576a9141925a274ac004373bb5429553bdb55c40e57b1246888ac
```

Which results in the following P2WSH address by network:

| Network   | Address                                                            |
|:----------|--------------------------------------------------------------------|
| `regtest` | `bcrt1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90ws9hj4q3` |
| `testnet` | `tb1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90wsgwcn4t`   |
| `mainnet` | `bc1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90wslxwu0y`   |

and the following P2SH(P2WSH) address by network:

| Network   | Address                               |
|:----------|---------------------------------------|
| `regtest` | `2NEzrgyqU2AB6tHZqeCNnkmHKEDRGpGkDfk` |
| `testnet` | `2NEzrgyqU2AB6tHZqeCNnkmHKEDRGpGkDfk` |
| `mainnet` | `3PSedEuSQhfkgVwHy4kv8pJ41sD6wE8Hc5`  |
