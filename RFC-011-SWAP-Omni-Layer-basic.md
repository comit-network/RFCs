# Omni Layer Basic HTLC Atomic Swap

- RFC-Number: 011
- Status: Draft
- Discussion-issue: -
- Created on: 27 Mar. 2019

# Table of Contents

<!-- toc -->

## Description

This RFC defines how to execute a [RFC003](./RFC-003-SWAP-basic.md) SWAP where one of the ledgers is Bitcoin and the associated asset is an Omni Layer token asset.

For the definition of the Bitcoin ledger see [RFC004](./RFC-004-SWAP-Bitcoin.md).
For the definition of the Omni Layer token asset see [RFC010](./TODO).

While it is possible to swap any kind of assets supporting [RFC003](./RFC-003-SWAP-basic.md) with Omni Layer token, it is recommended to use this RFC and [RFC003](./RFC-003-SWAP-basic.md) for inter-ledger swaps.
For swap between assets on the same Ledger (Bitcoin), subsequent RFC may introduced better fitted swap protocols.

To fulfil the requirements of [RFC003](./RFC-003-SWAP-basic.md) this RFC defines:
- How to construct a Bitcoin transaction with a Hash Time Lock Contract (HTLC) to lock the Omni Layer token asset on the Bitcoin blockchain.
- How to construct a Bitcoin transaction to deploy, redeem and refund the HTLC during the execution phase of [RFC003](./RFC-003-SWAP-basic.md).

The Bitcoin contract to be used and its execution are to be done exactly as defined in [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md) for the Bitcoin asset.

## Limitations 

This RFC only supports the [simple send](https://github.com/OmniLayer/spec#transfer-coins-simple-send) Omni Layer transaction type.


Thsi RFC only supports Omni Layer [Class C](https://github.com/OmniLayer/spec#embedding-omni-protocol-data-in-the-block-chain) transactions. Class C transactions uses Bitcoin `OP_RETURN` instruction to store the Omni Layer data.

## Ownership

In Omni Layer, tokens are not owned by a specific UTXO but by a specific public key.

Really, the *ownership* of Omni Layer tokens is the ability to make a valid Nitcoin transaction in which one of the spent input was unlocked by a public key that owns tokens.

Because of that not all Omni Layer tokens owned by a given public key have to be spent in one transaction.
The Bitcoin concept of *change address* does not apply to Omni Layer.

If a transaction spends only part of the Omni Layer tokens available, then the remainder is still owned by the original public key.

Thus, Bitcoin transactions that deal with Omni Layer often *re-use* addresses by locking the Bitcoin change against the same public key than the input, instead of using a brand new public key using [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

Finally, using simple send, it seems that only the ownership of tokens of the **first input** are transferred. 

## Inputs & Outputs format

This RFC introduces specific restrictions on the inputs and output of the Bitcoin transactions to allow the swap of Omni Layer tokens.

For clarity purpose, an *input, output or address owning an Omni Layer token* refers to the fact that such input output is unlocked by a public key that owns such Omni Layer token.

### Omni Layer data

<!-- TODO: describes what to put in the OP_RETURN transaction -->


### Dust

The reference implementation of the Bitcoin consensus, `bitcoind` enforces that spendable UTXO have enough value to be spent.
If the value owned by a UTXO is less than the minimum spendable then it is what we commonly call dust.

For `bitcoind`, it is currently defined in term of `dustRelayFee` and it is set by default to `3000 satoshis-per-kilobytes`.
For a normal transaction, the minimum value is 546 satoshis; 294 satoshis for SegWit transactions.

As [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md) only defines SegWit HTLC then the former value is to be considered and will be referred as `min_sat`.

`OP_RETURN` outputs are not spendable and hence SHOULD not be assigned any Bitcoin value, or it would be lost. 

Note that in the sections below, all Bitcoin value are indicative. Implementation SHOULD follow the recommendation.

The only hard rule is that spendable output MUST have at least `min_sat` Satoshis assigned.

### Deployment transaction

To construct the deployment transaction, the following information is needed:

| Variable | Description
|:--- |:--- |
| `funder_address` | Address owning the Omni Layer tokens that will fund the HTLC. |
| `funder_utxo` | UTXO locked by `funder_address` that owns some Bitcoin asset to allow the transaction to be valid as per Bitcoin consensus. |
| `initial_sat` | The amount of Satoshis initially owned by `funder_utxo` |  
| `fee_sat` | The amount of Satoshi the funder wishes to spend on mining fees |
| `htlc_address` | Address of the HTLC as described in [RFC-005](./RFC-005-SWAP-Bitcoin-basic.md) |
| `omni_data` | The Omni Layer data that transfers the Omni Layer token asset as described in [Omni Layer data](#omni-layer-data) |

The Bitcoin transaction MUST be constructed with the following inputs, outputs and values. The order MUST be as indicated.
The *assigned Bitcoin value* on the inputs is indicative:

| Inputs (assigned Bitcoin value) | Outputs (assigned Bitcoin value) |
|:--- |:---
| 1. `funder_utxo` (`initial_sat`) | 1. `htlc_address` (`min_sat`) |
| | 2. `funder_address` (`initial_sat - min_sat - fee_sat`) |
| | 3. `OP_RETURN omni_data` (`0`) 

### Redeem transaction
 
 To construct the redeem transaction, the following information is needed:
 
 | Variable | Description | 
 |:--- |:---|
 | `htlc_utxo` | UTXO locked by the HTLC, created with the deployment transaction |
 | `redeemer_address` | Address to which the Omni Layer token will be sent upon redeeming. |
 | `fee_address` | Address that owns some Bitcoin to allow the payment of the mining fee. CAN be the same than `redeemer_address` |
 | `init_sat` | Amount of Satoshis owned by `fee_address` |
 | `fee_sat` | The amount of Satoshi the redeemer wishes to spend on mining fees |
 | `omni_data` | The Omni Layer data that transfers the Omni Layer token asset as described in [Omni Layer data](#omni-layer-data) |
 
 The Bitcoin transaction MUST be constructed with the following inputs, outputs and values. The order MUST be as indicated.
 The *assigned Bitcoin value* on the inputs is indicative:
 
 | Inputs (assigned Bitcoin value) | Outputs (assigned Bitcoin value) |
 |:--- |:--- |
 | 1. `htlc_utxo` (`min_sat`)    | 1. `redeemer_address` (`min_sat`) |
 | 2. `fee_address` (`init_sat`) | 2. `fee_address` (`init_sat - fee_sat`) |
 |                               | 3. `OP_RETURN omni_data` (`0`) |
 
The redeemer MAY include output `2. fee_address` or MAY decide to transfer `init_sat` Bitcoin to `redeemer_address`. 
 
### Refund transaction

<!-- TODO -->

# Examples/Test vectors

## RFC003 SWAP REQUEST

<!-- TODO -->

## RFC003 SWAP RESPONSE

<!-- TODO -->
