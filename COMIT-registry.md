# Registry for common types used across COMIT RFCs

## Description

This registry defines all types used across COMIT RFCs.
This registry may be expanded with new RFCs.

## Ledgers

<!-- TODO: issue to be opened as `value` for Bitcoin mainnet is currently "Bitcoin" because of rust bitcoin -->
<!-- TODO: Issue needed as currently ledger `value`s are currently parameterized -->
<!-- TODO: Issue needed as Ethereum networks are supported -->
<!-- TODO: Parameters to be moved in Bitcoin/Ethereum RFC -->
| Name     | Description            | Reference |  `value`   | `parameters` key | `parameters` value |`parameters` description         |
|:---      |:---                    |:---       |:---        |:---              |:---                |:---                             |
| Bitcoin  | Bitcoin-core network   | TBD       | `bitcoin`  | `network`        |                    | The network on which to operate |
|          |                        |           |            |                  | `regtest`          | Bitcoin-core regtest            |
|          |                        |           |            |                  | `testnet`          | Bitcoin testnet                 |
|          |                        |           |            |                  | `mainnet`          | Bitcoin mainnet                 |
| Ethereum | Ethereum network       | TBD       | `ethereum` | `network`        |                    | The network on which to operate |
|          |                        |           |            |                  | `regtest`          | Local dev network               |
|          |                        |           |            |                  | `ropsten`          | Ropsten testnet (network id 3)  |
|          |                        |           |            |                  | `mainnet`          | Ethereum mainnet (network id 1) |

## Assets
<!-- TODO: Parameters to be moved in Bitcoin/Ethereum RFC -->
| Name           | Description                   | Reference | `value`   | `parameters` key | `parameters` value type | `parameters` description |
|:---            |:----                          |:---       |:---       |:---              |:---                     |:---                      |
| Bitcoin        | Native Bitcoin network asset  | TBD       | `bitcoin` | `quantity`       | integer in Json string  | Amount in satoshi        |
| Ether          | Native Ethereum network asset | TBD       | `ether`   | `quantity`       | integer in Json string  | Amount in wei            |
| ERC-20 Token   | As defined by [ERC-20 standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) | TBD | `erc20` | `address` | hex string including `0x` prefix | The hex address of the smart contract defining the given token |
|                 |                              |           |           |  `quantity`      |  integer in Json string  | The token amount without the decimal, e.g. 9000 PAY Tokens: `"9000000000000000000000"`, knowing that the PAY smart contract defines 18 decimals for its token |

## Protocols

TODO

## Headers

| Name           | Reference                                 | `value`
|:---            |:---                                       |:---                                    
| `alpha_ledger` | [RFC-002](./RFC-002-SWAP.md#alpha_ledger) | See [Ledgers](#ledgers)
| `beta_ledger`  | [RFC-002](./RFC-002-SWAP.md#beta_ledger)  | See [Ledgers](#ledgers)
| `alpha_asset ` | [RFC-002](./RFC-002-SWAP.md#alpha_asset)  | See [Assets](#assets)
| `beta_asset`   | [RFC-002](./RFC-002-SWAP.md#beta_asset)   | See [Assets](#assets)
| `protocol`     | [RFC-002](./RFC-002-SWAP.md#protocol)     | See [Protocols](#protocols)
| `reason`       | [RFC-002](./RFC-002-SWAP.md#reason-optional)       | [`unsatisfactory-rate`](./RFC-002-SWAP.md#reason-optional), [`unsatisfactory-quantity`](./RFC-002-SWAP.md#reason-optional), [`protocol-unsupported`](./RFC-002-SWAP.md#reason-optional), [`unsupported-ledger`](./RFC-002-SWAP.md#reason-optional), [`unavailable-asset`](./RFC-002-SWAP.md#reason-optional)
