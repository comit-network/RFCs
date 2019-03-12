# Table of contents

<!-- markdown-toc start -->
- [Registry](#registry)
    - [Description](#description)
    - [Ledgers](#ledgers)
        - [`bitcoin` Parameters](#bitcoin-parameters)
            - [Bitcoin Networks](#bitcoin-networks)
        - [`ethereum` Parameters](#ethereum-parameters)
            - [Ethereum Networks](#ethereum-networks)
    - [Assets](#assets)
        - [`bitcoin` Parameters](#bitcoin-parameters-1)
        - [`ether` Parameters](#ether-parameters)
        - [`erc20` Parameters](#erc20-parameters)
    - [Identities](#identities)
    - [Protocols](#protocols)
    - [Hash Functions](#hash-functions)
    - [Headers](#headers)
<!-- markdown-toc end -->

# Registry

## Description

This registry defines all types used across COMIT RFCs.
This registry may be expanded with new RFCs.

## Ledgers

The following is a list of possible values a `Ledger` type header can take:

| Value      | Reference                            | Description                            |
|:-----------|--------------------------------------|----------------------------------------|
| `bitcoin`  | [RFC-004](./RFC-004-SWAP-Bitcoin.md) | The Bitcoin Core family of blockchains |
| `ethereum` | TBD                                  | The Ethereum family of blockchains     |


And the possible parameters they each may have:

### `bitcoin` Parameters

| Parameter | Value Type                                  | Description                              |
|:----------|---------------------------------------------|------------------------------------------|
| `network` | see [Bitcoin Networks](#bitcoin-networks) | The particular blockchain netowrk to use |

#### Bitcoin Networks

| Value     | Description                          |
|:----------|:-------------------------------------|
| `regtest` | Private Bitcoin Core regtest network |
| `testnet` | Bitcoin Core testnet                 |
| `mainnet` | Bitcoin Core mainnet                 |


### `ethereum` Parameters

| Parameter | Value Type                                    | Description                              |
|:----------|-----------------------------------------------|------------------------------------------|
| `network` | see [Ethereum Networks](#ethereum-networks) | The particular blockchain netowrk to use |


#### Ethereum Networks

| Value     | Description                      | Network Id |
|:----------|:---------------------------------|------------|
| `regtest` | Private Ethereum regtest network | N/A        |
| `ropsten` | Ropsten testnet                  | 3          |
| `mainnet` | Ethereum mainnet                 | 1          |

## Assets

The following is a list of possible values an `Asset` type header can take:


| Value     | Reference                            | Description                   |
|:----------|--------------------------------------|-------------------------------|
| `bitcoin` | [RFC-004](./RFC-004-SWAP-Bitcoin.md) | Native Bitcoin network asset  |
| `ether`   | TBD                                  | Native Ethereum network asset |
| `erc20`   | TBD                                  | ERC20 token                   |

And the possible parameters they each may have:

### `bitcoin` Parameters

| Parameter  | Value Type | Description         |
|:-----------|------------|---------------------|
| `quantity` | `u64`      | Quantity in satoshi |

### `ether` Parameters

| Parameter  | Description     | Value Type |
|:-----------|-----------------|------------|
| `quantity` | Quantity in wei | `u256`     |


### `erc20` Parameters

| Parameter  | Value Type | Description                                                                 |
|:-----------|------------|-----------------------------------------------------------------------------|
| `quantity` | `u256`     | The ERC20 contract value to be transferred (not the decimal token quantity) |
| `address`  | TBD        | The address of the ERC20 contract                                           |


## Identities

[RFC003](./RFC-003-SWAP-basic.md#identity) requires that each ledger has an associated identity:

| Ledger   | Identity Name | JSON Encoding            | Reference                            | Description                                                                           |
|:---------|:--------------|:-------------------------|:-------------------------------------|---------------------------------------------------------------------------------------|
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | [RFC-005](./RFC-004-SWAP-Bitcoin-basic.md) | The result of applying SHA256 and then RIPEMD160 to a SECP256k1 compressed public key |
| Ethereum | `address`     | TBD                      | TBD                                  | An Ethereum address                                                                   |

## Protocols

The following is a list of protocols defined in COMIT RFCs for use in the `protocol` header of a SWAP message.

| Name                   | Reference                       |
|:----------------------- |:-------------------------------- |
| Basic HTLC Atomic Swap | [RFC-003](./RFC-003-SWAP-basic) |


## Hash Functions

The following is a list of cryptographic hash functions for use within COMIT protocols:


| Name    | Reference  |
|:------- |:----------- |
| `SHA-256`| [IETF RFC463](https://tools.ietf.org/html/rfc4634#section-4.1) |

## Headers

| Name           | Reference                                    | `value`                                                                                                                                                                                                                                                                                                                                                                          |
|:---------------|:---------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `alpha_ledger` | [RFC-002](./RFC-002-SWAP.md#alpha_ledger)    | See [Ledgers](#ledgers)                                                                                                                                                                                                                                                                                                                                                          |
| `beta_ledger`  | [RFC-002](./RFC-002-SWAP.md#beta_ledger)     | See [Ledgers](#ledgers)                                                                                                                                                                                                                                                                                                                                                          |
| `alpha_asset ` | [RFC-002](./RFC-002-SWAP.md#alpha_asset)     | See [Assets](#assets)                                                                                                                                                                                                                                                                                                                                                            |
| `beta_asset`   | [RFC-002](./RFC-002-SWAP.md#beta_asset)      | See [Assets](#assets)                                                                                                                                                                                                                                                                                                                                                            |
| `protocol`     | [RFC-002](./RFC-002-SWAP.md#protocol)        | See [Protocols](#protocols)                                                                                                                                                                                                                                                                                                                                                      |
| `reason`       | [RFC-002](./RFC-002-SWAP.md#reason-optional) | [`unsatisfactory-rate`](./RFC-002-SWAP.md#reason-optional), [`unsatisfactory-quantity`](./RFC-002-SWAP.md#reason-optional), [`protocol-unsupported`](./RFC-002-SWAP.md#reason-optional), [`unsupported-ledger`](./RFC-002-SWAP.md#reason-optional), [`unavailable-asset`](./RFC-002-SWAP.md#reason-optional), [`timeouts-too-tight`](./RFC-003-SWAP-basic.md#timeouts-too-tight) |
|                |                                              |                                                                                                                                                                                                                                                                                                                                                                                  |
