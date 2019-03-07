# Bitcoin Ledger in SWAP Messages

- RFC-Number: 004
- Status: Draft
- Discussion-issue: -
- Created on: 1 Mar. 2019

# Table of contents

<!-- toc -->

- [Description](#description)
- [Motivation](#motivation)
- [Content](#content)
  - [Ledger definition](#ledger-definition)
    - [Parameters](#parameters)
  - [Asset definition](#asset-definition)
- [Registry extension](#registry-extension)

<!-- tocstop -->

## Description

This RFC specifies the how the Bitcoin blockchain and its native asset Bitcoin can described in RFC002 SWAP messages.
Bitcoin refers specifically to the Bitcoin Core  [Bitcoin Core](https://github.com/bitcoin/bitcoin/) and not any blockchains derived from it.
As required by RFC002, this RFC defines the *ledger* value `bitcoin` and its associated parameters as well as the *asset* `bitcoin`.
This in combination with subsequent RFCs that define concrete execution steps for particular SWAP protocols enable users to execute and negotiate Bitcoin swaps.

## The Bitcoin Ledger

To specify Bitcoin in one of the RFC002 ledger headers (`alpha_ledger` or `beta_ledger`) use the value `bitcoin` with the following parameter:

### `network`

The `network` parameter describes whether you are intending to do a SWAP with *real* Bitcoin on the mainnet or are doing a test SWAP on the testnet or a shared regtest node.
The `network` parameter is mandatory and can take the following values:

| value     | description                                              |
|:----------|:---------------------------------------------------------|
| `regtest` | Bitcoin Core regtest or other local/private test network |
| `testnet` | Bitcoin Core testnet                                     |
| `mainnet` | Bitcoin Core mainnet                                     |


All private test networks like [btcd](https://github.com/btcsuite/btcd)'s `simnet` should be specified as `regtest`.

## The Bitcoin Asset

Bitcoin is also the name of the Bitcoin blockchain's native asset.
This RFC only pertains to the native Bitcoin asset.
Other non-native assets will be defined in subsequent RFCs.

Although Bitcoin is nominally the native asset, ownership of Bitcoin is really the ability to unlock *unspent transaction outputs* whose value is measured in *satoshi*.
1 Bitcoin is simply the name for 100,000,000 satoshi.

To specify the Bitcoin in one of the RFC002 asset headers (`alpha_asset` or `beta_asset`) use the value `bitcoin` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of satoshi that the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u64` (note that in the JSON encoding a `u64` is encoded as decimal string like `"100000000"`).

## Registry extension

- Addition of [Bitcoin Ledger](./registry.md#ledgers)
- Addition of [Bitcoin Asset](./registry.md#assets)

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
