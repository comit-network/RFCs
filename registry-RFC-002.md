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

<!-- tocstop -->

## Description

This registry defines the possible values used to encode SWAP message types as defined in [RFC-002](./RFC-002-SWAP.md).
This registry purpose may be expanded when extending [RFC-002](./RFC-002-SWAP.md) to new ledgers and assets.

Refer to [RFC-002](./RFC-002-SWAP.md) for the full definition of the headers.

This RFC contains the definition of the following `BAM!` headers:
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
