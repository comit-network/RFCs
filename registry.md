# Registry

**Table of contents**
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
- [Protocols](#protocols)
- [Negotiation Errors](#negotiation-errors)
  - [Details of the `timeouts-too-tight` error](#details-of-the-timeouts-too-tight-error)
- [Identities](#identities)
- [Hash Functions](#hash-functions)
- [Headers](#headers)

## Description

This registry defines all types used across COMIT RFCs. This registry may be expanded with new RFCs.

## Ledgers

The following is a list of possible values a `Ledger` type header can take:

| Value      | Reference                        | Description                                         |
| :--------- | -------------------------------- | --------------------------------------------------- |
| `bitcoin`  | [RFC-004](./RFC-004-Bitcoin.md)  | The Bitcoin Core family of blockchains              |
| `ethereum` | [RFC-006](./RFC-006-Ethereum.md) | A blockchain following the Ethereum consensus rules |

And the possible parameters they each may have:

### `bitcoin` Parameters

| Parameter | Value Type                                | Description                              |
| :-------- | ----------------------------------------- | ---------------------------------------- |
| `network` | see [Bitcoin Networks](#bitcoin-networks) | The particular blockchain network to use |

#### Bitcoin Networks

| Value     | Description                          |
| :-------- | :----------------------------------- |
| `regtest` | Private Bitcoin Core regtest network |
| `testnet` | Bitcoin Core testnet                 |
| `mainnet` | Bitcoin Core mainnet                 |

### `ethereum` Parameters

| Parameter | Value Type                                  | Description                              |
| :-------- | ------------------------------------------- | ---------------------------------------- |
| `network` | see [Ethereum Networks](#ethereum-networks) | The particular blockchain network to use |

#### Ethereum Networks

| Value     | Description                          |
| :-------- | :----------------------------------- |
| `regtest` | Private Ethereum development network |
| `ropsten` | Ropsten testnet                      |
| `mainnet` | Ethereum mainnet                     |

## Assets

The following is a list of possible values an `Asset` type header can take:

| Value     | Reference                        | Description                   |
| :-------- | -------------------------------- | ----------------------------- |
| `bitcoin` | [RFC-004](./RFC-004-Bitcoin.md)  | Native Bitcoin network asset  |
| `ether`   | [RFC-006](./RFC-006-Ethereum.md) | Native Ethereum network asset |
| `erc20`   | [RFC-008](./RFC-008-ERC20.md)    | ERC20 token                   |

And the possible parameters they each may have:

### `bitcoin` Parameters

| Parameter  | Value Type | Description         |
| :--------- | ---------- | ------------------- |
| `quantity` | `u64`      | Quantity in satoshi |

### `ether` Parameters

| Parameter  | Value Type | Description     |
| :--------- | ---------- | --------------- |
| `quantity` | `u256`     | Quantity in wei |

### `erc20` Parameters

| Parameter  | Value Type            | Description                                                                 |
| :--------- | --------------------- | --------------------------------------------------------------------------- |
| `quantity` | `u256`                | The ERC20 contract value to be transferred (not the decimal token quantity) |
| `address`  | `0x` prefixed address | The address of the ERC20 contract                                           |

## Protocols

The following is a list of protocols defined in COMIT RFCs for use in the `protocol` header of a SWAP message.

| Value           | Reference                          | Description            |
| :-------------- | ---------------------------------- | ---------------------- |
| `comit-rfc-003` | [RFC-003](./RFC-003-SWAP-Basic.md) | Basic HTLC Atomic Swap |

## Negotiation Errors

The following is a list of errors that receivers of a SWAP REQUEST may choose to send back to the sender. In order to send a negotiation error, the value of the `negotiation_result` header MUST be set to "failed".

| `reason`               | Reference                          | Description                                                                                                                                                              | `details`                                           |
| :--------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| `unsatisfactory-rate`  | [RFC-002](./RFC-002-SWAP.md)       | The rate of `alpha_asset` to `beta_asset` is not satisfactory to the receiver.                                                                                           | None                                                |
| `protocol-unsupported` | [RFC-002](./RFC-002-SWAP.md)       | The protocol specified in the `protocol` header is not known to the receiving party.                                                                                     | None                                                |
| `unknown-ledger`       | [RFC-002](./RFC-002-SWAP.md)       | A ledger referenced by the sending party is unknown to the receiving party.                                                                                              | None                                                |
| `unknown-asset`        | [RFC-002](./RFC-002-SWAP.md)       | An asset referenced by the sending party is unknown to the receiving party.                                                                                              | None                                                |
| `timeouts-too-tight`   | [RFC-003](./RFC-003-SWAP-Basic.md) | This indicates to the sender that the difference between `alpha_expiry` and `beta_expiry` is too small and the receiver may accept the swap if they are given more time. | [Details](#details-of-the-timeouts-too-tight-error) |

### Details of the `timeouts-too-tight` error

| property name | type   | description                                                                         |
| :------------ | :----- | :---------------------------------------------------------------------------------- |
| min_time      | number | The minimum time difference between the HLTCs in seconds that the receiver requires |

## Identities

[RFC003](./RFC-003-SWAP-Basic.md#identity) requires that each ledger has an associated identity:

| Ledger   | Identity Name | JSON Encoding            | Reference                                  | Description                                                                           |
| :------- | :------------ | :----------------------- | :----------------------------------------- | ------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | [RFC-004](./RFC-005-SWAP-Basic-Bitcoin.md) | The result of applying SHA256 and then RIPEMD160 to a SECP256k1 compressed public key |
| Ethereum | `address`     | `0x` prefixed address    | [RFC-007](./RFC-007-SWAP-Basic-Ether.md)   | An Ethereum Address                                                                   |

## Hash Functions

The following is a list of cryptographic hash functions for use within COMIT protocols:

| Name      | Reference                                                      |
| :-------- | :------------------------------------------------------------- |
| `SHA-256` | [IETF RFC463](https://tools.ietf.org/html/rfc4634#section-4.1) |

## Headers

| Name                 | Reference                    | `value`                     |
| :------------------- | :--------------------------- | :-------------------------- |
| `alpha_ledger`       | [RFC-002](./RFC-002-SWAP.md) | See [Ledgers](#ledgers)     |
| `beta_ledger`        | [RFC-002](./RFC-002-SWAP.md) | See [Ledgers](#ledgers)     |
| `alpha_asset`        | [RFC-002](./RFC-002-SWAP.md) | See [Assets](#assets)       |
| `beta_asset`         | [RFC-002](./RFC-002-SWAP.md) | See [Assets](#assets)       |
| `protocol`           | [RFC-002](./RFC-002-SWAP.md) | See [Protocols](#protocols) |
| `negotiation_result` | [RFC-002](./RFC-002-SWAP.md) | `"successful" OR "failed"`  |
