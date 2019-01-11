# Registry for common types used across COMIT RFCs

## Description

This registry defines common types used across COMIT RFCs.
This registry may be expanded when adding support to new ledgers and assets.

## Ledger

### Bitcoin

`value`: `bitcoin`
<!-- TODO: Issue needed as currently `Bitcoin` -->

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

## Asset

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

`parameters.quantity`: the amount without a decimal; integer in a string format; e.g. 9000 PAY Tokens: `"9000000000000000000000"`, knowing that the PAY smart contract defines 18 decimals for its token.
