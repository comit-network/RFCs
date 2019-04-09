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

## Limitations

This RFC only supports the [simple send](https://github.com/OmniLayer/spec#transfer-coins-simple-send) Omni Layer transaction type.
<!--  If you didn't use simple send to fund the contract then what would go wrong? If you add the How to construct omni layer transaction section maybe you can remove this sentence -->

This RFC only supports Omni Layer [Class C](https://github.com/OmniLayer/spec#embedding-omni-protocol-data-in-the-block-chain) transactions.
Class C transactions uses Bitcoin `OP_RETURN` instruction to store the Omni Layer data.

## Ownership

In Omni Layer, tokens are not owned by a specific UTXO but by a specific public key.
<!-- Mention "account model"? -->
<!-- We should re-word this around addresses I think +-->
<!-- I would like to remove this explanation to the other RFC +-->

The *ownership* of Omni Layer tokens is the ability to make a valid Bitcoin transaction in which a spent input was unlocked by a public key that owns tokens.
<!-- Can we just say owning Omni Layer tokens is being able to spend from an output that owns omin layer tokens? -->

Because of that not all Omni Layer tokens owned by a given public key have to be spent in one transaction.
The Bitcoin concept of *change address* does not apply to Omni Layer.

If a transaction spends only part of the Omni Layer tokens available, then the remainder is still owned by the original public key.

Thus, Bitcoin transactions that deal with Omni Layer often *re-use* addresses by locking the Bitcoin change against the same public key than the input, instead of using a brand new public key using [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).
<!-- What is an implementor meant to do with this information? If nothing delete.  (Maybe move to other RFC) -->

Finally, using simple send, it seems that only the ownership of tokens of the **first input** are transferred.
The ownership is transferred to the one output which:
- is not already present in the inputs (in term of public key)
- is not an `OP_RETURN` output

<!-- ^ This seems to be the important part for motivating the transaction design-->

## Inputs & Outputs format
<!-- I think put all this useful omni info under a heading with a strong motivation: Constructing Omni Layer Transactions -->
<!-- Include the table from the redeem and refund sections in there and show how to construct the transaction -->

This RFC introduces specific restrictions on the inputs and output of the Bitcoin transactions to allow the swap of Omni Layer tokens.
<!-- I don't think this RFC introduces restrictions on things. It just defines how to send tokens to an HTLC and how to redeem/refund them-->

For clarity purpose, an *input, output owning an Omni Layer token* refers to the fact that such input or output is unlocked by a public key that owns such token.
An *address owning Omni Layer token* is an address constructed from a public key that owns such token.

### Omni Layer data

The Omni Layer data to be included in `OP_RETURN` is as per the Omni Layer protocol for a
<!-- Unfinished sentence  -->

Such data MAY be constructed manually or the implementer MAY decide to relay on an existing Omni Layer implementation to generate it.
<!-- Link to specification of how to construct it -->

For example, the following omnicore API MAY be used:
- `property_id`: the property id of the Omni Layer Asset
- `amount`: the quantity of Omni Layer Assets to transfer
- `payload`: the hex-encoded Omni Layer Data
```
$ [...] omni_createpayload_simplesend <property_id> <amount>
<payload>
```
<!-- Refer to omni layer spec for simple send instead of this -->

To then embed the Omni Layer Data received from omnicore in an `OP_RETURN` output, it MUST be prefixed it with the Omni Layer header: `6f6d6e69`.

### Dust

<!-- Dust actually refers to UTXOs that are not profitable to spend: https://eprint.iacr.org/2018/513 -->
The reference implementation of the Bitcoin consensus, _bitcoind_ enforces that spendable UTXO MUST own enough value to be spent.
<!-- Is this true? Link please.  -->
If the value owned by a UTXO is less than the minimum spendable then it is what we commonly call dust and such transaction would be considered as *non-standard* and rejected.
<!-- Yeah this I think is true. But if a non-standard transaction gets in I think it can be spent? -->

For `bitcoind`, it is currently defined in term of `dustRelayFee` and it is set by default to `3000 satoshis-per-kilobytes`.
The minimum value is 546 satoshis for a normal transaction and 294 satoshis for SegWit transactions.

As [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md) only defines SegWit HTLC then the former value is to be considered and will be referred as `min_sat` (294 Satoshis).
<!-- ^I think this sentence isn't true if you use P2SH(P2WSH) but not sure -->

`OP_RETURN` outputs are not spendable and hence SHOULD not be assigned any Bitcoin value.

The only requirement is that spendable output MUST have at least `min_sat` Satoshis assigned.
<!-- Is this really true? If you're a miner you can mine txs less than min_sat right? So it's a SHOULD I guess -->

### Deployment transaction
<!-- Put this under a section called Execution phase. See the other RFCs. -->
<!-- Each section should start with how to execute the action and then how both parties can tell if the action has been executed -->

The deployment transaction MUST be a simple send of the correct amount and kind of Omni Assets to the HTLC address.

The HTLC address MUST either be:
1. the P2WSH address of the HTLC, as described in [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md)
2. the P2SH of the P2WSH above <!-- Need to link to the BIP describing this -->

(1) is preferred however, not all Omni Layer wallets implementation support bech32 addresses, hence, (2) can be used as a fallback.

The simple send MUST NOT be done to the direct P2SH address of the HTLC.
<!-- Maybe give a reason for this -->

The party watching for the deployment transaction MUST consider both (1) and (2) addresses.

Such party MUST check the validity of the transaction against Omni Layer Spec.
Not all valid Bitcoin transactions containing Omni Data are valid Omni Layer transactions.

For example, the following RPC call to omnicore can be used to verify the validity of a Omni Layer transaction:
- `txid`: the transaction id of the transaction
 ```
$ [...] omni_gettransation <txid>
{
  [...]
  "valid" : true
  [...]
}
```

<!-- we have to check the value that was transferred as well -->

### Redeem transaction

<!-- There is some repeating of yourself between these two sections. -->
<!-- As mentioned above, I think there should be seperate section before this that explains how to transfer omni tokens to and from HTLC -->
<!-- Put these table in there and then refer to it in the deployment, redeem and refund sections -->

<!-- BEIGN HOW LLOYD WOULD WRITE IT -->
To redeem the omni HTLC, the redeemer MUST submit a valid Omni Layer transaction transferring the all the omni tokens from the HTLC address to their desired address.
The transaction should be constructed according to the [Constructing Omni Transactions](..) section.
To construct the witness data for the P2WSH utxo, the redeemer MUST use the procedure described [Redeem section of RFC005](..).

To be notified of the redeem event, both parties SHOULD.... <!-- I don't yet fully get how to do this -->
<!-- END HOW LLOYD WOULD WRITE IT -->

<!-- I think the above is enough for this section -->

to construct the redeem transaction, the following information is needed:

| Variable           | Description                                                                                                 |
|:---                |:---                                                                                                         |
| `htlc_utxo`        | UTXO locked by the HTLC, created with the deployment transaction                                            |
| `redeem_address`   | Address to which the Omni Layer token will be sent upon redeeming                                           |
| `change_address`   | Address to which the Bitcoin change will be sent upon redeeming; MUST be different than `redeem_address`    |
| `change_utxo`      | UTXO locked by `change_address` that owns some Bitcoin asset to pay for the mining fee                      |
| `init_change_sat`  | Amount of Satoshis owned by `change_utxo` before the redeem                                                 |
| `mining_sat`       | The amount of Satoshi the redeemer wishes to spend on mining fees                                           |
| `omni_data`        | The Omni Layer data that transfers the Omni Layer Asset as described in [Omni Layer data](#omni-layer-data) |

 The Bitcoin transaction MUST be constructed with the following inputs, outputs and values.
 The order of the inputs and outputs MUST be as described.
 The *assigned Bitcoin value* on the inputs SHOULD be the following:

 | Inputs (assigned Bitcoin value)      | Outputs (assigned Bitcoin value)                     |
 |:---                                  |:---                                                  |
 | 1. `htlc_utxo` (`min_sat`)           | 1. `redeem_address` (`min_sat`)                      |
 | 2. `change_utxo` (`init_change_sat`) | 2. `change_address` (`init_change_sat - mining_sat`) |
 |                                      | 3. `OP_RETURN omni_data` (`0`)                       |

The redeemer MAY include output (2) `change_address` or MAY decide to omit it and transfer the Bitcoin change to `redeem_address`.
if output (2) `change_address` is present then input (2) `change_utxo` MUST be included to ensure the Omni Layer Assets are sent to output (1) `redeem_address`.

### Refund transaction

To construct the refund transaction, the following information is needed:

| Variable           | Description                                                                                                 |
|:---                |:---                                                                                                         |
| `htlc_utxo`        | UTXO locked by the HTLC, created with the deployment transaction                                            |
| `refund_address`   | Address to which the Omni Layer token will be sent upon refunding                                           |
| `change_address`   | Address to which the Bitcoin change will be sent upon refunding; MUST be different than `refund_address`  |
| `change_utxo`      | UTXO locked by `change_address` that owns some Bitcoin asset to pay for the mining fee                      |
| `init_change_sat`  | Amount of Satoshis owned by `change_utxo` before the refund                                                 |
| `mining_sat`       | The amount of Satoshi the refunder wishes to spend on mining fees                                           |
| `omni_data`        | The Omni Layer data that transfers the Omni Layer Asset as described in [Omni Layer data](#omni-layer-data) |

 The Bitcoin transaction MUST be constructed with the following inputs, outputs and values.
 The order of the inputs and outputs MUST be as described.
 The *assigned Bitcoin value* on the inputs SHOULD be as described:

 | Inputs (assigned Bitcoin value)      | Outputs (assigned Bitcoin value)                     |
 |:---                                  |:---                                                  |
 | 1. `htlc_utxo` (`min_sat`)           | 1. `refund_address` (`min_sat`)                      |
 | 2. `change_utxo` (`init_change_sat`) | 2. `change_address` (`init_change_sat - mining_sat`) |
 |                                      | 3. `OP_RETURN omni_data` (`0`)                       |

The redeemer MAY include output (2) `change_address` or MAY decide to omit it and transfer the Bitcoin change to `refund_address`.
if output (2) `change_address` is present then input (2) `change_utxo` MUST be included to ensure the Omni Layer Assets are sent to output (1) `refund_address`.

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
