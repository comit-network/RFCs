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

To describe an Omni asset in an Asset type header specify `omni` as the value along with the following paramters:

### `quantity`

The parameter `quantity` describes the asset's amount.
The `quantity` parameter is mandatory.


A given Omni asset is defined as *divisible* or *indivisible* at creation.
To cater for these two types of assets, the following behaviour is defined in the Omni Layer protocol field [Number of coins field](https://github.com/OmniLayer/spec#field-number-of-coins):
- If the asset is **indivisible**, then the `quantity` value is the exact number of tokens, i.e. `1` represents one token.
- If the asset is **divisible**, then the `quantity` value represents a hundred millionth of a token, i.e. `100,000,000` represents one token.

The `quantity` value will be inserted in the appropriate Omni Layer transactions, hence the same behaviour is adopted.

Its value MUST be a `u64` (note that in JSON encoding a `u64` is encoded as decimal string like `"100000000"`).

### `property_id`

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
