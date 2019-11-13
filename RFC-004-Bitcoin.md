# The Bitcoin Ledger

- RFC-Number: 004
- Status: Draft
- Created on: 1 Mar. 2019

**Table of Contents**
- [Description](#description)
- [The Bitcoin Ledger](#the-bitcoin-ledger)
    - [`network`](#network)
- [The Bitcoin Asset](#the-bitcoin-asset)
    - [`quantity`](#quantity)
- [Registry extension](#registry-extension)
    - [The Bitcoin Ledger](#the-bitcoin-ledger-1)
        - [Bitcoin Networks](#bitcoin-networks)
    - [The Bitcoin Asset](#the-bitcoin-asset-1)
- [SWAP Request](#swap-request)

## Description

This RFC specifies how the Bitcoin blockchain and its native asset Bitcoin are described in Ledger and Asset type headers within the COMIT protocol.
Bitcoin refers specifically to [Bitcoin Core](https://github.com/bitcoin/bitcoin/) and not any blockchains derived from it.
The Ledger and Asset type headers were introduced in [RFC002](RFC-002-SWAP.adoc) to describe assets being exchanged in a COMIT SWAP protocol.

## The Bitcoin Ledger

To specify Bitcoin in a Ledger type header use the value `bitcoin` with the following parameter:

### `network`

The `network` parameter describes whether you are intending to do a SWAP with *real* Bitcoin on the `mainnet` or are doing a test SWAP on the `testnet` or a shared `regtest` node.
The `network` parameter is mandatory.

All private test networks like [btcd](https://github.com/btcsuite/btcd)'s `simnet` should be specified as `regtest`.

## The Bitcoin Asset

Bitcoin is also the name of the Bitcoin blockchain's native asset.
This RFC only contains protocol definitions for the native Bitcoin asset.
Non-native assets like [colored coins](https://en.bitcoin.it/wiki/Colored_Coins) may be defined in subsequent RFCs.

Although Bitcoin is nominally the native asset, ownership of Bitcoin is really the ability to unlock *unspent transaction outputs* (UTXOs) whose value is measured in *satoshi*.
1 Bitcoin is simply the name for 100,000,000 (10<sup>8</sup>) satoshi.

To specify Bitcoin in an Asset type header use the value `bitcoin` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of satoshi that the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u64` encoded as a decimal string, e.g. `"100000000"`

## Registry extension

### The Bitcoin Ledger

This RFC extends the registry [Ledgers section](./registry.md#ledgers) with the `bitcoin` ledger:

| Value     | Description             |
|:----------|-------------------------|
| `bitcoin` | Bitcoin Core Blockchain |


And defines the `network` parameter for it:

| Parameter | Value Type                                  | Description                              |
|:----------|---------------------------------------------|------------------------------------------|
| `network` | see [Bitcoin Networks](#bitcoin-networks) | The particular blockchain network to use |

#### Bitcoin Networks

And adds a Bitcoin Networks section describing the possible values for this parameter:

| Value     | Description                          |
|:----------|:-------------------------------------|
| `regtest` | Private Bitcoin Core regtest network |
| `testnet` | Bitcoin Core testnet                 |
| `mainnet` | Bitcoin Core mainnet                 |

### The Bitcoin Asset

This RFC extends the registry [Assets section](./registry.md#assets) with the `bitcoin` asset:

| Value     | Description                  |
|:----------|:-----------------------------|
| `bitcoin` | Native Bitcoin network asset |

And defines the `quantity` parameter for it:

| Key        | Value Type | Description         |
|:-----------|------------|---------------------|
| `quantity` | `u64`      | Quantity in satoshi |

# Examples

## SWAP Request

The following shows an example [RFC002](RFC-002-SWAP.adoc) SWAP REQUEST with Bitcoin as the `alpha_ledger` and 1 Bitcoin as the `alpha_asset`.
Fields that are outside of the scope of this RFC are filled with `...`.

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
      "value": "bitcoin",
      "parameters": { "quantity": "100000000" }
    },
    "beta_asset": {...},
    "protocol": "...",
  },
  "body": {...},
}
```
