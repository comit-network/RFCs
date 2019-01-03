# SWAP Message Types

<!-- toc -->

- [Description](#description)
- [Status](#status)
- [Assumptions](#assumptions)
- [Terminology](#terminology)
  * [Alice & Bob](#alice--bob)
  * [Ledger](#ledger)
  * [Asset](#asset)
  * [Alpha Ledger (recip. Asset)](#alpha-ledger-recip-asset)
  * [Beta Ledger (recip. Asset)](#beta-ledger-recip-asset)
  * [Identity](#identity)
  * [Swap Protocol](#swap-protocol)
- [SWAP REQUEST](#swap-request)
  * [Purpose](#purpose)
  * [Definition](#definition)
    + [Format](#format)
    + [`alpha_ledger/beta_ledger`](#alpha_ledgerbeta_ledger)
    + [`alpha_asset/beta_asset`](#alpha_assetbeta_asset)
    + [`swap_protocol`](#swap_protocol)
    + [`body`](#body)
- [SWAP RESPONSE](#swap-response)
  * [Purpose](#purpose-1)
  * [Definition](#definition-1)
    + [Format](#format-3)
    + [Decline with Reason Format](#decline-with-reason-format)
    + [`Status`](#status)
    + [`Reason`](#reason)

<!-- tocstop -->

## Description

This RFC defines the `SWAP` message types for `BAM` (see [RFC-001](./RFC-001-BAM.md)), using JSON encoding.
This set of message types facilitates the execution of trustless[¹], cross-ledger atomic swaps of assets as described in [RFC-003](wip).

This RFC is part of COMIT, an open protocol facilitating trustless cross-blockchain applications.


## Status

```
RFC-Number: 002
Status: Draft
```

## Assumptions

The messages are assumed to be transported using the BAM! protocol as defined in [RFC-001](./RFC-001-BAM.md).

## Terminology

### Alice & Bob

Participants of the atomic swap. Alice and Bob intend to exchange two assets via an atomic swap. Alice and Bob may refer to both the users and the COMIT nodes proceeding with the swap.

### Ledger

A distributed record (e.g. a blockchain) which tracks asset ownerships and transactions. For example, the Bitcoin or Ethereum networks

This RFC assumes that a ledger has basic properties described as part of [this section](#terminology).

### Asset

A digital asset. Its ownership can be tracked on a [ledger](#ledger).

### Alpha Ledger (recip. Asset)

The ledger on which (recip. the asset that) Alice sells and Bob buys[²].

### Beta Ledger (recip. Asset)

The ledger on which (recip. the asset that) Alice buys and Bob sells.

### Identity

An identifier to which the ownership of a given asset can be transferred. 

### Swap Protocol

A protocol that defines the steps, transactions and communications needed to proceed with an atomic swap.

[RFC-003](wip) is an available swap protocol. However, it is not limited to [RFC-003](wip) and this RFC is expected to be the building block for other swap protocols.

## SWAP REQUEST

### Purpose

For Alice to request Bob to proceed with an atomic swap.
This message contains all the information that Alice needs to provide for both parties to proceed with the swap.

### Definition
```
BAM! type: REQUEST
```
#### Format
```
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": AlphaLedger,
    "beta_ledger": BetaLedger,
    "alpha_asset": AlphaAsset,
    "beta_asset": BetaAsset,
    "swap_protocol": SwapProtocol.
    },
  "body": Body, 
}
```

#### `alpha_ledger/beta_ledger`
The *Alpha Ledger*/*Beta Ledger* for this swap. As defined in the [terminology](#terminology) section.

##### Format
```
{
  "value": LedgerName,
  "parameters": { "network": Network }
}
```
##### `value`
`LedgerName`: the name of the ledger in ASCII (case insensitive). <!-- TODO: Issue needed as currently case sensitive -->
Examples: `Bitcoin`, `Ethereum`, `Lightning`

##### `parameters`
###### `network`

`Network`: the target network.
Examples:
* for Bitcoin: `mainnet`<!-- TODO: Issue needed as not supported. rust-bitcoin uses "bitcoin" -->, `testnet`, `regtest`
* for Ethereum: <!-- TODO: Issue needed as not supported -->`mainnet`, `kovan`, `ropsten`, `regtest`

##### Sample
```json
{
  "value": "Bitcoin",
  "parameters": { "network": "regtest" }
}
```

#### `alpha_asset/beta_asset`
The *Alpha Asset*/*Beta Asset* for this swap. As defined in the [terminology](#asset).

The `parameters`' value depends on the kind of the described asset. Native assets SHOULD only have a `quantity` integer parameter in the smallest unit, supported by the *Ledger*.

Esoteric assets such as tokens, sub-coins or collectibles MAY need further parameters to be described.

##### Format
###### Native Asset
```
{
  "value": AssetName,
  "parameters": { "quantity": Quantity },
}
```

###### Contract-based Token
```
{
  "value": AssetName,
  "parameters": {
    "address": ContractAddress,
    "quantity": Amount
  },
}
```

##### `value`
`AssetName`: the capitalized name of the asset in ASCII.
Examples: `Bitcoin`, `Ether`, `ERC20`.

##### `parameters`
###### `quantity`

`Quantity`: the integer amount in smallest unit of the asset; Wei for Ether, Satoshi for Bitcoin.
Example: `"4200000000000000000"` for 4.2 Ether.
`"9000000000000000000000"` for 9000 PAY Token, assuming that the PAY token contract has 18 decimals.

###### `address`
`ContractAddress`: where relevant, the address of the token contract.

###### Samples
1 BTC.
```json
{
  "value": "Bitcoin",
  "parameters": { "quantity": "100000000" }
}
```
9000 PAY tokens.
```json
{
  "value": "ERC20",
  "parameters": {
    "address": "0xB97048628DB6B661D4C2aA833e95Dbe1A905B280",
    "quantity": "9000000000000000000000"
  }
}
```

#### `swap_protocol`
The protocol used to proceed with the swap. Defined in subsequent RFCs.

##### `value`
Currently defined protocols
- [RFC-003](wip)<!-- TODO: Add descriptive name of the protocol. To be done with #7 -->: `COMIT-RFC-003`

##### Sample
RFC-003 HTLC based protocol.
```json
{
  "value": "COMIT-RFC-003",
  "parameters": {},  
}
```

#### `body`

The body is defined in the RFC of the given `SwapProtocol`.

## SWAP RESPONSE

### Purpose

For Bob to accept or decline a **SWAP REQUEST** previously made by Alice.
If Bob accepts the request, this message contains all the information that Bob needs to provide for both parties to proceed with the swap.

### Definition
```
BAM! type: RESPONSE
```
#### Format
```
{
  "status": Status
  "body": Body,
}
```
Valid for both accept and decline responses.

#### Decline with Reason Format
```
{
  "status": "SE20",
  "headers": {
    "reason": Reason
  }
}
```
<!-- TODO: Open issue to make `reason` lower case -->

#### `Status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Each swap protocol MAY define their own statuses for 40 and above.

* `OK20`: Swap request is accepted
* `RE00`: Receiver (Bob) Internal Error (as per [RFC-001](./RFC-001-BAM.md#status-code-families))<!-- TODO: open an issue because we incorrectly use SE00 in the code -->
* `SE20`: Swap declined
* `SE21`: Swap rejected due to unsupported ledger combination
* `SE22`: Swap rejected due to unsupported swap protocol

#### `Reason`
The reason for which a swap was declined.
Bob MAY use this field to tip Alice on why the swap was declined.

* `bad-rate`: proposed rate for the swap is too low <!-- TODO: fix the reasons in code -->
* `liquidity-shortage`: not enough liquidity to proceed with swap

More reasons MAY be defined by a given swap protocol specification.

---

###### ¹ trustless: as in no one (counterpart or third party) has to be trusted.
[¹]:#-trustless-as-in-no-one-counterpart-or-third-party-has-to-be-trusted
###### ² In case of [RFC-003](wip), alpha ledger is the first ledger on which an action is needed, hence the name.
[²]:#-in-case-of-rfc-003-this-the-first-ledger-on-which-an-action-is-needed-hence-the-name
