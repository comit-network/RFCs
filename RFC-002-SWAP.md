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
    + [`AlphaLedger/BetaLedger`](#alphaledgerbetaledger)
    + [`AlphaAsset/BetaAsset`](#alphaassetbetaasset)
    + [`SwapProtocol`](#swapprotocol)
    + [`Body`](#body)
- [SWAP RESPONSE](#swap-response)
  * [Purpose](#purpose-1)
  * [Definition](#definition-1)
    + [Format](#format-2)
    + [Decline with Reason Format](#decline-with-reason-format)
    + [`Status`](#status)
    + [`Reason`](#reason)

<!-- tocstop -->

## Description

This RFC describes the SWAP type messages.
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

Typically, Bitcoin, Ether or an ERC-20 token.

### Alpha Ledger (recip. Asset)

The ledger on which (recip. the asset that) Alice sells and Bob buys[²].

### Beta Ledger (recip. Asset)

The ledger on which (recip. the asset that) Alice buys and Bob sells.

### Identity

An identifier to which the ownership of a given asset can be transferred. 

Typically, a Bitcoin public key or an Ethereum account.

### Swap Protocol

A protocol that defines the steps, transactions and communications needed to proceed with an atomic swap.


Typically, [RFC-003](wip). However, it is not limited to [RFC-003](wip) and this RFC is expected to be the building block for other swap protocols.

## SWAP REQUEST

### Purpose

For Alice to request Bob to proceed with an atomic swap.
This message contains all the information that Alice needs to provide for both parties to proceed with the swap.

A **SWAP RESPONSE**, to accept or decline the **SWAP REQUEST**, SHOULD be sent by Bob.

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

#### `AlphaLedger/BetaLedger`
The *Alpha Ledger*/*Beta Ledger* for this swap. As defined in the [terminology](#terminology) section.

##### Format
```
{
  "value": LedgerName,
  "parameters": { "network": Network }
}
```
`LedgerName`: the name of the ledger in ASCII (case insensitive). <!-- TODO: Issue needed as currently case sensitive -->
Examples: `Bitcoin`, `Ethereum`, `Lightning`

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

#### `AlphaAsset/BetaAsset`
The *Alpha Asset*/*Beta Asset* for this swap. As defined in the [terminology](#asset).

The `parameters`' value depends on the kind of the described asset. Native assets SHOULD only have a `quantity` integer parameter in the smallest unit, supported by the *Ledger*.

Esoteric assets such as tokens, sub-coins or collectibles MAY need further parameters to be described.

##### Native Asset Format
```
{
  "value": AssetName,
  "parameters": { "quantity": Amount },
}
```
`AssetName`: the capitalized name of the asset in ASCII.
Examples: `Bitcoin`, `Ether`

`Amount`: the amount in smallest unit of the asset; Wei for Ether, Satoshi for Bitcoin.
Example: `"4200000000000000000"` for 4.2 Ether.

###### Sample
1 BTC.
```json
{
  "value": "Bitcoin",
  "parameters": { "quantity": "100000000" }
}
```

##### Contract-based Token Format
```
{
  "value": AssetName,
  "parameters": {
    "address": ContractAddress,
    "quantity": Amount
},
}
```
`AssetName`: the capitalized name of the asset in ASCII.
Example: `ERC20`

`ContractAddress`: where relevant, the address of the token contract.

`Amount`: the integer amount in the smallest unit of the asset; in case of ERC20 tokens, the decimal point is ignored.
Example: `"9000000000000000000000"` for 9000 PAY Token, assuming that the PAY token contract has 18 decimals. 

###### Sample
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

#### `SwapProtocol`
The protocol used to proceed with the swap. Defined in subsequent RFCs.

##### Currently defined protocols
- [RFC-003](wip)<!-- TODO: Add descriptive name of the protocol. To be done with #7 -->: `COMIT-RFC-003`

##### Sample
RFC-003 HTLC based protocol.
```json
{
  "value": "COMIT-RFC-003",
  "parameters": {},  
}
```

#### `Body`

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
