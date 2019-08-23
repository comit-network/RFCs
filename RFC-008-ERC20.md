# The ERC20 Asset

- RFC-Number: 008
- Status: Draft
- Discussion-issue: [#45](https://github.com/comit-network/RFCs/issues/45)
- Created on: 2019-03-12

**Table of Contents**
- [Description](#description)
- [The ERC20 Asset](#the-erc20-asset)
    - [`quantity`](#quantity)
    - [`token_contract`](#token_contract)
- [Registry extension](#registry-extension)

## Description

This RFC defines how Ethereum based [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) tokens should be described in Asset type headers.

The Asset type header was introduced in [RFC002](RFC-002-SWAP.adoc) to describe assets being exchanged in a COMIT SWAP protocol.

## The ERC20 Asset

The ERC20 token standard was introduced in [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md).
Token contracts record how much of the token is owned by particular addresses.

Each ERC20 token contract defines its name.
Just as with the native Ether asset, the nominal token quantity is measured in a smaller denomination which we will call *token wei*.
The relationship between the nominal asset and the token wei is defined per contract.
For the purposes of this RFC we ignore nominal amounts completely and simply refer to quantities of token wei.

To describe ERC20 asset in an Asset type header specify `erc20` as the value along with the following parameters:

### `quantity`

The `quantity` parameter describes the quantity of token wei the asset represents.
The `quantity` parameter is mandatory.
Its value MUST be a `u256` encoded as a decimal string, e.g. `"1000000000000000000"`.

### `token_contract`

The `token_contract` parameter specifies which token contract and therefore which token the asset header is referring to.
The `token_contract` parameter is mandatory.
The address MUST be encoded as this 20-byte hex string prefixed by `0x` (as is standard in the Ethereum ecosystem).
Furthermore, implementations MUST also accept [EIP50](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md) mixed case addresses and MAY verify the checksum.

The address MUST be a *contract address* which complies with [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md).
Implementations MAY use fixed list of contract addresses that are known to be compliant.

## Registry extension

This RFC extends the registry's [Assets table](./registry.md#assets) with the `erc20` Asset:

| Value   | Description |
|:--------|-------------|
| `erc20` | ERC20 token |

And defines its parameters:

| Parameter        | Value Type            | Description                                                                 |
|:-----------------|-----------------------|-----------------------------------------------------------------------------|
| `quantity`       | `u256`                | The ERC20 contract value to be transferred (not the decimal token quantity) |
| `token_contract` | `0x` prefixed address | The address of the ERC20 contract                                           |

# Examples

The following shows an example [RFC002](RFC-002-SWAP.adoc) SWAP REQUEST with Ethereum as the `alpha_ledger` and 1 [PAY token](https://etherscan.io/token/0xB97048628DB6B661D4C2aA833e95Dbe1A905B280) as the `alpha_asset`.
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
      "value": "erc20",
      "parameters": {
          "quantity": "1000000000000000000",
          "token_contract": "0xb97048628db6b661d4c2aa833e95dbe1a905b280"
      }
    },
    "beta_asset": {...},
    "protocol": "...",
  },
  "body": {...},
}

```
