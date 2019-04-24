# The Omni Layer Asset

- RFC-Number: 010
- Status: Draft
- Discussion-issue: -
- Created on: 27 Mar. 2019

**Table of Contents**

  * [Description](#description)
  * [The Omni Layer Assets](#the-omni-layer-assets)
    * [`quantity`](#quantity)
    * [`property_id`](#property_id)
  * [Registry extension](#registry-extension)
  * [Examples](#examples)

## Description

This RFC defines how Bitcoin-based Omni Layer Assets are described in Asset type headers.
Omni Layer refers specifically to the [Omni Layer Protocol](https://github.com/OmniLayer/spec).

The Asset type header was introduced in [RFC002](./RFC-002-SWAP.md) to describe assets being exchanged in a COMIT SWAP protocol.


## The Omni Layer Assets

The Omni Layer Protocol defines several _assets_, also commonly referred as _coins_ or _tokens_ (from now on referred as Omni _asset_).
It also allows the definition of new assets by the mean of a Bitcoin transaction containing specific data.

This asset MUST be used on the Bitcoin Ledger, as defined in [RFC-004](./RFC-004-SWAP-Bitcoin.md).
Subsequent RFCs MAY introduce the support of other ledgers for Omni Layer Assets.

While *Omni* or *OmniCoin* holds a special place among the assets as being predefined and having special properties in the Omni Layer Protocol, it is not different than other Omni assets in the context of the COMIT protocol family.

Each Omni asset defines its name.
Omni assets are uniquely identified by a *property id*.

For a list of pre-defined Omni assets refer to [Omni Layer Specs](https://github.com/OmniLayer/spec#field-currency-identifier).

### Ownership

In Omni Layer, tokens are not owned by a specific UTXO but by a specific address.
This can referred as an *account model*.

The *ownership* of Omni Layer tokens is the ability to spend an output that owns omni layer tokens.

Because of that not all Omni Layer tokens owned by a given public key have to be spent in one transaction.
The Bitcoin concept of *change address* does not apply to Omni Layer.

If a transaction spends only part of the Omni Layer tokens available, then the remainder is still owned by the original address.

Thus, Bitcoin transactions that deal with Omni Layer often *re-use* addresses by locking the Bitcoin change against the same public key than the input, instead of using a brand new public key using [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

Finally, using simple send, it seems that only the ownership of tokens of the **first input** are transferred.
The ownership is transferred to the one output which:
- is not already present in the inputs (in term of public key)
- is not an `OP_RETURN` output


### Dust

The reference implementation of the Bitcoin Core consensus, _bitcoind_ [enforces](https://github.com/bitcoin/bitcoin/blob/c536dfbcb00fb15963bf5d507b7017c241718bf6/src/policy/policy.cpp#L129) that spendable UTXO MUST own enough value to be spent.
If the value owned by a UTXO is less than the minimum fee for a typical transaction then it is what we commonly call [dust](https://github.com/bitcoin/bitcoin/blob/c536dfbcb00fb15963bf5d507b7017c241718bf6/src/policy/policy.cpp#L20). A transaction that creates such output is considered as *non-standard* by bitcoind and rejected under normal configuration.

For `bitcoind`, it is currently defined in term of `dustRelayFee` and it is set by default to `3000 satoshis-per-kilobytes`.
The minimum value is 546 satoshis for a normal transaction and 294 satoshis for SegWit transactions.

### Header encoding

To describe an Omni asset in an Asset type header specify `omni` as the value along with the following parameters:

#### `quantity`

The parameter `quantity` describes the asset's amount.
The `quantity` parameter is mandatory.


As per the Omni Layer Spec, assets are defined as *divisible* or *indivisible* at creation.
To cater for these two types of assets, the following behaviour is defined in the Omni Layer protocol field [Number of coins field](https://github.com/OmniLayer/spec#field-number-of-coins):
- If the asset is **indivisible**, then the `quantity` value is the exact number of tokens, i.e. `1` represents one token.
- If the asset is **divisible**, then the `quantity` value represents a hundred millionth of a token, i.e. `100,000,000` represents one token.

The `quantity` value will be inserted in the appropriate Omni Layer transactions, hence the same behaviour is adopted.

Its value MUST be a `u64` (note that in JSON encoding a `u64` is encoded as decimal string like `"100000000"`).

#### `property_id`

The `property_id` parameter specifies which property id and therefore which specific asset the asset header is referring to.
The `property_id` parameter is mandatory.

Note that property id is assigned by the Omni Layer Protocol at the creation of an asset - it is simply the next available property id.

The property id MUST be of an existing asset on the target Bitcoin ledger.


## Registry extension

This RFC extends the registry's [Assets table](./registry.md#assets) with the `omni` Asset:

| Value        | Description      |
:---           |---               |
| `omni`       | Omni Layer Asset |

And defines its parameters:

| Parameter        | Value Type | Description                                            |
|:-----------------|------------|--------------------------------------------------------|
| `quantity`       | `u64`      | The Omni Layer asset amount to be transferred          |
| `property_id`    | number     | The property id of the Omni Layer asset                |


## Examples

The following shows an example [RFC002](./RFC-002-SWAP.md) JSON encoded SWAP REQUEST with Bitcoin as the `alpha_ledger` and 1 [Omni Layer asset TetherUS](https://www.omniexplorer.info/asset/31) as the `alpha_asset`.
Fields that are outside of the scope of this RFC are filled with `...`.
Note: TetherUS is divisible.

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
          "property_id": 31
      }
    },
    "beta_asset": {...},
    "protocol": "...",
  },
  "body": {...},
}

```
