# The Omni Layer Token Asset

- RFC-Number: 010
- Status: Draft
- Discussion-issue: -
- Created on: 27 Mar. 2019

## Table of Contents

<!-- toc -->

  * [Description](#description)
  * [The Omni Layer Assets](#the-omni-layer-assets)
    + [`quantity`](#quantity)
    + [`property_id`](#property_id)
  * [Registry extension](#registry-extension)
- [Examples](#examples)

<!-- tocstop -->

## Description

This RFC specifies how Bitcoin based Omni Layer Assets MUST be described in [RFC002](./RFC-002-SWAP.md) SWAP messages.
OmniLayer refers specifically to [Omni Protocol](https://github.com/OmniLayer/spec).
As required by [RFC002](./RFC-002-SWAP.md), this RFC defines the *asset* value `omni_layer`.
This in combination with subsequent RFCs that define concrete execution steps for particular SWAP protocols enable users to negotiate and execute Omni Layer Token swaps.

## The Omni Layer Assets

The Omni Protocol defines several _cryptocurrencies_, also commonly referred as _coins_ or _tokens_.
It also allow the definition of new tokens by the mean of a Bitcoin transaction containing specific data.

This asset MUST be used on the Bitcoin Ledger, as defined in [RFC-004](./RFC-004-SWAP-Bitcoin.md).
Subsequent RFCs MAY introduce the support of other ledgers for the Omni Layer token Asset.

While *Omni* or *OmniCoin* holds a special place among the tokens as being predefined and having special properties in the Omni Protocol, it is not different than other Omni Layer tokens in the context of the COMIT protocol family.

Each Omni Layer token defines its name.
Omni Layer tokens are uniquely identified by a *property id*.
Notable tokens have the following property ids:
- Omni Coin (predefined): `1`
- Test Omni Coin (predefined): `2`
- TetherUS: `31`

For a full list of pre-defined coins refer to [Omni Layer Specs](https://github.com/OmniLayer/spec).
 
To specify an Omni Layer Token in one of the [RFC002](./RFC-002-SWAP.md) asset headers (`alpha_asset` or `beta_asset`) use the value `omni_layer` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of tokens that the asset represents.
The `quantity` parameter is mandatory.

A given Omni Layer token is defined as *divisible* or *indivisible* at creation.
To cater for these two types of token, the [Omni Protocol- Number of coins field](https://github.com/OmniLayer/spec#field-number-of-coins) defines the following behaviour in Omni Layer transactions:
- If the token is **indivisible** then the `quantity` value is the exact number of tokens: `1` represents one token.
- If the token is **divisible** then the `quantity` value is to be divided by 100,000,000: `100,000,000` represents one token.

The `quantity` value will be inserted in the appropriate Omni Layer transactions, hence the same behaviour is adopted.

Its value MUST be a `u64` (note that in the JSON encoding a `u64` is encoded as decimal string like `"100000000"`).

### `property_id`

The `property_id` parameter specifies which property id and therefore which token the asset header is referring to.
The `property_id` parameter is mandatory.

Note that property id is assigned by the Omni Protocol at the creation of a token, it is simply the next available property id.

The property id MUST be of an existing token on the target Bitcoin ledger.


## Registry extension

This RFC extends the registry's [Assets table](./registry.md#assets) with the `omni_layer` Asset:

| Value        | Description      |
:---           |---               |
| `omni_layer` | Omni Layer token |

And defines its parameters:

| Parameter        | Value Type | Description                                            |
|:-----------------|------------|--------------------------------------------------------|
| `quantity`       | `u64`      | The number of coins as the eponymous Omni Layer field. |
| `property_id`    | number     | The property id of the Omni Layer token                |


# Examples

The following shows an example [RFC002](./RFC-002-SWAP.md) JSON encoded SWAP REQUEST with Bitcoin as the `alpha_ledger` and 1 [Omni Layer token TetherUS](https://www.omniexplorer.info/asset/31) as the `alpha_asset`.
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
      "value": "omni_layer",
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
