# Registry for RFC002 SWAP Message Types

<!-- toc -->

- [Description](#description)
- [`alpha_ledger/beta_ledger`](#alpha_ledgerbeta_ledger)
  * [Bitcoin](#bitcoin)
  * [Ethereum](#ethereum)
- [`alpha_asset/beta_asset`](#alpha_assetbeta_asset)
  * [Bitcoin](#bitcoin-1)
  * [Ethereum](#ethereum-1)
    + [Ether](#ether)
    + [ERC20 token](#erc20-token)
- [`reason`](#reason)
  * [Rate is unsatisfying](#rate-is-unsatisfying)
    + [`parameters`](#parameters)
  * [Exceeding quantity](#exceeding-quantity)
    + [`parameters`](#parameters-1)
  * [Unsupported swap protocol](#unsupported-swap-protocol)
    + [`parameters`](#parameters-2)
  * [Unsupported ledger combination](#unsupported-ledger-combination)
    + [`parameters`](#parameters-3)

<!-- tocstop -->

## Description

This registry defines the possible values used to encode SWAP message types as defined in [RFC-002](./RFC-002-SWAP.md).
This registry may be expanded when extending [RFC-002](./RFC-002-SWAP.md) to new ledgers and assets.

Refer to [RFC-002](./RFC-002-SWAP.md) for the full definition of the headers.

This RFC contains the definition of possible values for the following `BAM!` headers:
- [`alpha_asset`](#alpha_assetbeta_asset)
- [`alpha_ledger`](#alpha_ledgerbeta_ledger)
- [`beta_asset`](#alpha_assetbeta_asset)
- [`beta_ledger`](#alpha_ledgerbeta_ledger)

## `alpha_ledger/beta_ledger`

### Bitcoin

`value`: `bitcoin`

`parameters.network`:
- `regtest`: Bitcoin-core regtest
- `testnet`: Bitcoin testnet
- `mainnet`: Bitcoin mainnet <!-- TODO: issue to be opened as it's currently "Bitcoin" because of rust bitcoin -->

### Ethereum

`value`: `ethereum`

`parameters.network`:
- `regtest`: Local dev network <!-- TODO: Issue needed as not supported -->
- `ropsten`: Ropsten testnet (network id 3)
- `mainnet`: Ethereum mainnet (network id 1)

## `alpha_asset/beta_asset`

### Bitcoin

`value`: `bitcoin`

`parameters.quantity`: in satoshis; integer in string format; e.g. 1 Bitcoin: `"100000000"`.

### Ethereum

#### Ether

`value`: `ether`

`parameters.quantity`: in wei; integer in string format; e.g. 1 Ether: `"1000000000000000000"`.

#### ERC20 token

`value`: `erc20`

`parameters.address`: the hex address of the smart contract defining the given ERC20 token including `0x` prefix, e.g. `0xB97048628DB6B661D4C2aA833e95Dbe1A905B280`.

`parameters.quantity`: the amount without a decimal; integer in a string format; e.g. 9000 PAY Tokens: `"9000000000000000000000"`, knowing that the PAY token contract defines 18 decimals.

## `reason`
### Rate is unsatisfying
The receiver rejects the exchange rate and may accept a swap request with a different rate.
`value`: `rate-declined`
#### `parameters`
Hints are optional.
If present, expected hints are `alpha_asset` and `beta_asset`. Both or none of them must be included.

### Exceeding quantity
The receiver cannot (liquidity) or does not want (risk) to proceed with such asset quantity and may accept the request if a lower asset quantity was requested.
`value`: `exceeding-quantity`
#### `parameters`
*Please note that hinting on this scenario may lead to privacy concerns (exposure of available liquidity)*.
Hints are optional.
If present, expected hints are `alpha_asset` and `beta_asset`. Both or none of them must be included.

### Unsupported protocol
The receiver does not support the requested protocol
`value`: `protocol-unsupported`
#### `parameters`
Hints are optional.
If present, expected hint is `protocol`. if the receiver may accept the asset exchange under a different protocol.

### Unsupported ledger combination
The receiver does not support the requested ledger combination.
`value`: `unsupported-ledger`
#### `parameters`
No hint is supported.
