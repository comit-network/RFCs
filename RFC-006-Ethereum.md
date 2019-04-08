# The Ethereum Ledger

- RFC-Number: 006
- Status: Draft
- Discussion-issue: [#47](https://github.com/comit-network/RFCs/issues/47)
- Created on: 2019-03-12

**Table of Contents**
- [Description](#description)
- [The Ethereum Ledger](#the-ethereum-ledger)
    - [`network`](#network)
- [The Ether Asset](#the-ether-asset)
    - [`quantity`](#quantity)
- [Registry extension](#registry-extension)
    - [The Ethereum Ledger](#the-ethereum-ledger-1)
        - [Ethereum Networks](#ethereum-networks)
- [Assets](#assets)

## Description

This RFC specifies how the Ethereum blockchain and its native asset Ether are described in Ledger and Asset type headers within the COMIT protocol.
*Ethereum* refers specifically to the blockchain endorsed by [The Ethereum Foundation](https://www.ethereum.org/foundation) and its associated testnets but not any other blockchains related to or derived from it.
The Ledger and Asset type headers were introduced in [RFC002](./RFC-002-SWAP.md) to describe assets being exchanged in a COMIT SWAP protocol.

## The Ethereum Ledger

To specify Ethereum in a Ledger type header use the value `ethereum` with the following parameter:

### `network`

The `network` parameter describes whether you are intending to do a SWAP with *real* Ether on the `mainnet` or are doing a test SWAP on a testnet or on a private development network.
The `network` parameter is mandatory.
The only testnet introduced by this RFC is `ropsten` but others may be added in subsequent RFCs.

If you are conducting a SWAP on a private development network use the value `regtest`.
It is expected that you and your counterparty know how to interpret this value in your given context.

See [Ethereum Networks](#ethereum-networks) in the registry extension for possible values for `network` introduced by this RFC.

## The Ether Asset

Ether is the native asset of the Ethereum blockchain.
This RFC only contains protocol definitions for the Ether asset.
Non-native assets like [ERC20](https://theethereum.wiki/w/index.php/ERC20_Token_Standard) tokens may be defined in subsequent RFCs.

Although Ether is nominally the native asset, amounts of Ether are counted in wei.
1 Ether =  1,000,000,000,000,000,000 (10<sup>18</sup>) wei.

To specify Ether in an Asset type header use the value `ether` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of wei that the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u256` (note that in the JSON encoding a `u256` is encoded as decimal string like `"1000000000000000000"`).


## Registry extension

### The Ethereum Ledger

This RFC extends the registry [Ledgers section](./registry.md#ledgers) with the `ethereum` ledger:

| Value      | Description                                         |
|:-----------|-----------------------------------------------------|
| `ethereum` | A blockchain following the Ethereum consensus rules |

And defines the `network` parameter for it:

| Parameter | Value Type                                  | Description                              |
|:----------|---------------------------------------------|------------------------------------------|
| `network` | see [Ethereum Networks](#ethereum-networks) | The particular blockchain network to use |


#### Ethereum Networks

And adds an Ethereum Networks section describing the possible values for this parameter:

| Value     | Description                          |
|:----------|:-------------------------------------|
| `regtest` | Private Ethereum development network |
| `ropsten` | Ropsten testnet                      |
| `mainnet` | Ethereum mainnet                     |


## Assets

This RFC extends the registry [Assets section](./registry.md#assets) with the `ethereum` asset:

| Value   | Description                   |
|:--------|:------------------------------|
| `ether` | Native Ethereum network asset |

And defines the `quantity` parameter for it:

| Key        | Value Type | Description     |
|:-----------|------------|-----------------|
| `quantity` | `u256`     | Quantity in wei |


# Examples

The following shows an example [RFC002](./RFC-002-SWAP.md) JSON encoded SWAP REQUEST with Ethereum as the `alpha_ledger` and 1 Ether as the `alpha_asset`.
Fields that are outside of the scope of this RFC are filled with `...`.

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
    "protocol": "...",
  },
  "body": {...},
}
```
