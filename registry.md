# Registry

**Table of contents**
- [Description](#description)
- [FRAME types](#frame-types)
    - [REQUEST / RESPONSE](#request--response)
        - [SWAP](#swap)
- [Ledgers](#ledgers)
    - [`bitcoin` Parameters](#bitcoin-parameters)
        - [Bitcoin Networks](#bitcoin-networks)
    - [`ethereum` Parameters](#ethereum-parameters)
        - [Ethereum Networks](#ethereum-networks)
- [Assets](#assets)
    - [`bitcoin` Parameters](#bitcoin-parameters-1)
    - [`ether` Parameters](#ether-parameters)
    - [`erc20` Parameters](#erc20-parameters)
    - [`omni` Parameters](#omni-parameters)
- [SWAP Protocols](#swap-protocols)
- [SWAP decline reasons](#swap-decline-reasons)
- [Identities](#identities)
- [Hash Functions](#hash-functions)

## Description

This registry defines all types used across COMIT RFCs.
This registry may be expanded with new RFCs.

## FRAME types

This section explains all possible FRAME `type`s.

### REQUEST / RESPONSE

Frames of type `REQUEST` have a `type` field.
The following `type`s are defined:

#### SWAP

Introduced in [RFC-002](RFC-002-SWAP.adoc).

A SWAP request and the according response allow for the following headers to appear:

- `alpha_ledger`
- `beta_ledger`
- `alpha_asset`
- `beta_asset`
- `protocol`
- `decision`

Please refer to [RFC-002](RFC-002-SWAP.adoc) for the exact definition of those headers.

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

| Value     | Reference                               | Description                   |
| :-------- | --------------------------------------- | ----------------------------- |
| `bitcoin` | [RFC-004](./RFC-004-Bitcoin.md)         | Native Bitcoin network asset  |
| `ether`   | [RFC-006](./RFC-006-Ethereum.md)        | Native Ethereum network asset |
| `erc20`   | [RFC-008](./RFC-008-ERC20.md)           | ERC20 token                   |
| `omni`    | [RFC-010](./RFC-010-SWAP-Omni-Layer.md) | Omni Layer token              |

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

### `omni` Parameters

| Parameter     | Value Type | Description                                       |
| :------------ | ---------- | ------------------------------------------------- |
| `quantity`    | `u64`      | The amount of Omni Layer assets to be transferred |
| `property_id` | number     | The property id of the Omni Layer asset           |

## SWAP Protocols

The following is a list of protocols defined in COMIT RFCs for use in the `protocol` header of a SWAP message (i.e. a REQUEST message with type SWAP).

| Value           | Reference                          | Description            |
| :-------------- | ---------------------------------- | ---------------------- |
| `comit-rfc-003` | [RFC-003](RFC-003-SWAP-Basic.adoc) | Basic HTLC Atomic Swap |

## SWAP decline reasons

The following is a list of response bodies that receivers of a SWAP REQUEST may choose to send back to the sender.
The value of the `decision` header MUST be set to `declined`.

| `reason`               | Reference                          | Description                                                                                                                                                              |
| :--------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `unsatisfactory-rate`  | [RFC-002](RFC-002-SWAP.adoc)       | The rate of `alpha_asset` to `beta_asset` is not satisfactory to the receiver.                                                                                           |
| `protocol-unsupported` | [RFC-002](RFC-002-SWAP.adoc)       | The protocol specified in the `protocol` header is not known to the receiving party.                                                                                     |
| `unknown-ledger`       | [RFC-002](RFC-002-SWAP.adoc)       | A ledger referenced by the sending party is unknown to the receiving party.                                                                                              |
| `unknown-asset`        | [RFC-002](RFC-002-SWAP.adoc)       | An asset referenced by the sending party is unknown to the receiving party.                                                                                              |
| `timeouts-too-tight`   | [RFC-003](RFC-003-SWAP-Basic.adoc) | This indicates to the sender that the difference between `alpha_expiry` and `beta_expiry` is too small and the receiver may accept the swap if they are given more time. |

## Identities

[RFC003](RFC-003-SWAP-Basic.adoc#identity) requires that each ledger has an associated identity:

| Ledger   | Identity Name | Encoding                 | Reference                                  | Description             |
| :------- | :------------ | :----------------------- | :----------------------------------------- | ----------------------- |
| Bitcoin  | `pubkey`      | `hex-encoded-bytes (33)` | [RFC-004](./RFC-005-SWAP-Basic-Bitcoin.md) | A compressed public key |
| Ethereum | `address`     | `0x` prefixed address    | [RFC-007](./RFC-007-SWAP-Basic-Ether.md)   | An Ethereum Address     |

## Hash Functions

The following is a list of cryptographic hash functions for use within COMIT protocols:

| Name      | Reference                                                      |
| :-------- | :------------------------------------------------------------- |
| `SHA-256` | [IETF RFC463](https://tools.ietf.org/html/rfc4634#section-4.1) |
