# Bitcoin Definitions

- RFC-Number: 004
- Status: Draft
- Discussion-issue: -
- Created on: 26 Feb. 2019

# Table of contents

- [Description](#description)
- [Motivation](#motivation)
- [Content](#content)
- [Registry extension](#registry-extension)

## Description

Introduces definitions to allow the execution of RFC-003 for Bitcoin asset on the [Bitcoin Core](https://github.com/bitcoin/bitcoin/) network.

TODO: Decide we if do 1 RFC that introduce general Bitcoin terms + RFC003 or 2 RFCs (one for each)

This includes the exact Bitcoin scripts to be used.

TODO: Do we want p2sh(p2wsh) support? (more wallets support it)

TODO: Do we want p2sh native support? (more wallets support it + more straightforward than p2sh(p2wsh))

TODO: based on both TODO above, decide to include version and/or BIPs to be enabled.

## Motivation

TODO: the motivation is already in the description, not sure this section is relevant.

## Content

### Ledger definition

- Name: Bitcoin
- Description: Bitcoin Core Network
- `value`: `bitcoin`

#### Parameters

| `parameters` key | `parameters` value |`parameters` description         |
|:---              |:---                |:---                             |
| `network`        |                    | The network on which to operate |
|                  | `regtest`          | Bitcoin-core regtest            |
|                  | `testnet`          | Bitcoin testnet                 |
|                  | `mainnet`          | Bitcoin mainnet                 |

### Asset definition

| Name           | Description                   | `value`   | `parameters` key | `parameters` value type | `parameters` description |
|:---            |:----                          |:---       |:---              |:---                     |:---                      |
| Bitcoin        | Native Bitcoin network asset  | `bitcoin` | `quantity`       | integer in Json string  | Amount in satoshi        |

### Identities

| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 compressed public key |

### HTLC

Definition of the HTLC in OP_CODEs and hex.
Describes how to generate the contract from hex (ie, what bytes contain what)

### Redeem
Describe redeem script

### Refund
Describe refund script and any caveat/things to watch out

### Timeouts

Do we want to discuss timeouts?

## Registry extension

A list of changes to the registry described in this RFC. For example, new protocols, hash functions etc.
