# Bitcoin Ledger in SWAP Messages

- RFC-Number: 004
- Status: Draft
- Discussion-issue: -
- Created on: 1 Mar. 2019

# Table of Contents
<!-- markdown-toc start -->

- [Bitcoin Ledger in SWAP Messages](#bitcoin-ledger-in-swap-messages)
- [Table of contents](#table-of-contents)
    - [Description](#description)
    - [The Bitcoin Ledger](#the-bitcoin-ledger)
        - [`network`](#network)
    - [The Bitcoin Asset](#the-bitcoin-asset)
        - [`quantity`](#quantity)
    - [Registry extension](#registry-extension)
        - [The Bitcoin Ledger](#the-bitcoin-ledger-1)
            - [Bitcoin Networks](#bitcoin-networks)
        - [The Bitcoin Asset](#the-bitcoin-asset-1)
- [Examples](#examples)

<!-- markdown-toc end -->

## Description

This RFC specifies how the Bitcoin blockchain and its native asset Bitcoin are described in RFC002 SWAP messages.
Bitcoin refers specifically to [Bitcoin Core](https://github.com/bitcoin/bitcoin/) and not any blockchains derived from it.
As required by RFC002, this RFC defines the *ledger* value `bitcoin` and its associated parameters as well as the *asset* `bitcoin`.
This in combination with subsequent RFCs that define concrete execution steps for particular SWAP protocols enable users to execute and negotiate Bitcoin swaps.

## The Bitcoin Ledger

To specify Bitcoin in one of the RFC002 ledger headers (`alpha_ledger` or `beta_ledger`) use the value `bitcoin` with the following parameter:

### `network`

The `network` parameter describes whether you are intending to do a SWAP with *real* Bitcoin on the `mainnet` or are doing a test SWAP on the `testnet` or a shared `regtest` node.
The `network` parameter is mandatory.

All private test networks like [btcd](https://github.com/btcsuite/btcd)'s `simnet` should be specified as `regtest`.

## The Bitcoin Asset

Bitcoin is also the name of the Bitcoin blockchain's native asset.
This RFC only contains protocol definitions for the native Bitcoin asset.
Non-native assets like [colored coins](https://en.bitcoin.it/wiki/Colored_Coins) may be defined in subsequent RFCs.

Although Bitcoin is nominally the native asset, ownership of Bitcoin is really the ability to unlock *unspent transaction outputs* (UTXOs) whose value is measured in *satoshi*.
1 Bitcoin is simply the name for 100,000,000 satoshi.

To specify Bitcoin in one of the RFC002 asset headers (`alpha_asset` or `beta_asset`) use the value `bitcoin` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of satoshi that the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u64` (note that in the JSON encoding a `u64` is encoded as decimal string like `"100000000"`).

## Registry extension

### The Bitcoin Ledger

This RFC extends the registry [Ledgers section](./registry#ledgers) with the `bitcoin` ledger:

| Value     | Description             |
|:----------|-------------------------|
| `bitcoin` | Bitcoin Core Blockchain |


And defines the `network` parameter for it:

| Parameter | Value Type                                  | Description                              |
|:----------|---------------------------------------------|------------------------------------------|
| `network` | see [Bitcoin Networks](./#bitcoin-networks) | The particular blockchain network to use |

#### Bitcoin Networks

And adds a Bitcoin Networks section describing the possible values for this header:

| Value     | Description          |
|:----------|:---------------------|
| `regtest` | Bitcoin-core regtest |
| `testnet` | Bitcoin testnet      |
| `mainnet` | Bitcoin mainnet      |

### The Bitcoin Asset

This RFC extends the registry [Assets section](./registry#assets) with the `bitcoin` asset:

| Value     | Description                  |
|:----------|:-----------------------------|
| `bitcoin` | Native Bitcoin network asset |

And defines the `quantity` parameter for it:

| Key        | Value Type | Description         |
|:-----------|------------|---------------------|
| `quantity` | `u64`      | Quantity in satoshi |

# Examples

The following shows a generic JSON encoded SWAP request (with irrelevant fields filled with `...`) with Bitcoin as the `alpha_ledger` and 1 Bitcoin as the `alpha_asset`:

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
