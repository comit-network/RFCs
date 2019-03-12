# Ethereum Basic HTLC Atomic Swap

## Description

This RFC specifies how the Ethereum blockchain and its native asset Ether are described in [RFC002](./RFC-002-SWAP.md) SWAP messages.
*Ethereum* refers specifically to the blockchain endorsed by [The Ethereum Foundation](https://www.ethereum.org/foundation) and its associated testnets but not any other blockchains related to or derived from it.
As required by [RFC002](./RFC-002-SWAP.md), this RFC defines the *ledger* value `ethereum` and as well as the *asset* `ether`.

## The Ethereum Ledger

To specify Ethereum in one of the [RFC002](./RFC-002-SWAP.md) ledger headers (`alpha_ledger` or `beta_ledger`) use the value `ethereum` with the following parameter:

### `network`

The `network` parameter describes whether you are intending to do a SWAP with *real* Ether on the `mainnet` or are doing a test SWAP on the `testnet` or a shared `regtest` node.
The `network` parameter is mandatory.

See [Ethereum Networks](#ethereum-networks) in the registry extension for possible value for `network` introduced by this RFC.

## The Ether Asset

Ether is native asset of the Ethereum blockchain.
This RFC only contains protocol definitions for the Ether asset.
Non-native assets like [ERC20](https://theethereum.wiki/w/index.php/ERC20_Token_Standard) tokens may be defined in subsequent RFCs.

Although Ether is nominally the native asset, amounts of Ether are counted in wei.
1 Ether =  1,000,000,000,000,000,000 (10¹⁸) wei.

To specify Ether as one of the [RFC002](./RFC-002-SWAP.md) asset headers (`alpha_asset` or `beta_asset`) use the value `ether` with the following parameter:

### `quantity`

The `quantity` parameter describes the quantity of wei that the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u256` (note that in the JSON encoding a `u256` is encoded as decimal string like `"1000000000000000000"`).


## Registry extension

### The Ethereum Ledger

This RFC extends the registry [Ledgers section](./registry.md#ledgers) with the `ethereum` ledger:

| Value      | Description                        |
|:-----------|------------------------------------|
| `ethereum` | The Ethereum family of blockchains |

And defines the `network` parameter for it:

| Parameter | Value Type                                  | Description                              |
|:----------|---------------------------------------------|------------------------------------------|
| `network` | see [Ethereum Networks](#ethereum-networks) | The particular blockchain network to use |


#### Ethereum Networks

And adds a Ethereum Networks section describing the possible values for this parameter:

| Value     | Description                      | Network Id |
|:----------|:---------------------------------|------------|
| `regtest` | Private Ethereum regtest network | N/A        |
| `ropsten` | Ropsten testnet                  | 3          |
| `mainnet` | Ethereum mainnet                 | 1          |


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
