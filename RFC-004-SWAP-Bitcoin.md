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

This RFC introduces Ledger & Asset definitions to be used in SWAP Messages for [Bitcoin Core](https://github.com/bitcoin/bitcoin/) network.

## Motivation

This RFC aims to define the necessary values and parameters to support Bitcoin Core in SWAP messages.
This, added to complimentary RFCs per SWAP protocol, allows the execution of SWAP involving assets defined on the Bitcoin Core Ledger.

## Content

In order to create SWAP messages that make use of the `bitcoin` ledger or the `bitcoin` asset, those need to be defined first.
The next sections define how the `bitcoin` ledger and `bitcoin` asset are encoded when used within a SWAP message.

### Ledger definition

This RFC's objective is to uniquely identify the Ledger on which the SWAP will operate.
It is a difficult task as soft and hard forks may happen and there is no guarantee that the conditions identifying today are valid tomorrow.

We define Bitcoin Core as the network and consensus whose standard implementation is [bitcoind](https://github.com/bitcoin/bitcoin/).

- Name: Bitcoin
- Description: Bitcoin Core Network
- `value`: `bitcoin`

#### Parameters

For security reason, to avoid mainnet asset (valuable) to be swapped with test assets (expendable), a network parameter is defined and must be present for the Bitcoin Ledger.

- Name: Network
- Description: the network on which to operate
- `parameters` key: `network` 

| `parameters` value |`parameters` description                                             |
|:---                |:---                                                                 |
| `regtest`          | Bitcoin Core regtest or other local/private test network            |
| `testnet`          | Bitcoin Core testnet                                                |
| `mainnet`          | Bitcoin Core mainnet                                                |

There are other known test network such as [btcd](https://github.com/btcsuite/btcd) simnet.
Such network should be represented as `regtest`.

### Asset definition

Only the native Bitcoin asset is defined in this RFC.
Other non-native assets should be defined in dedicated RFCs.

All quantities are encoded without a unit.
They are to be interpreted as satoshis to avoid ambiguity in parser implementations.

To avoid issues with lost precision during parsing, the actual amount is encoded as an integer inside a string.

| Name           | Description                   | `value`   | `parameters` key | `parameters` value type   | `parameters` description |
|:---            |:----                          |:---       |:---              |:---                       |:---                      |
| Bitcoin        | Native Bitcoin network asset  | `bitcoin` | `quantity`       | integer in a JSON string  | Amount in satoshi        |

## Registry extension

- Addition of [Bitcoin Ledger](./registry.md#ledgers)
- Addition of [Bitcoin Asset](./registry.md#assets)