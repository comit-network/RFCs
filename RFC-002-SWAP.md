# SWAP Message Types

<!-- toc -->

- [Description](#description)
- [Status](#status)
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
    + [`status`](#status)
    + [`body`](#body-1)

<!-- tocstop -->

## Description

This RFC defines the `SWAP` message types for `BAM!` (see [RFC-001](./RFC-001-BAM.md)), using JSON encoding.
This set of message types facilitates the execution of trustless[¹], cross-ledger atomic swaps of assets as described in [RFC-003](wip).

This RFC is part of COMIT, an open protocol facilitating trustless cross-blockchain applications.

This RFC contains the definition of the following `BAM!` headers:
- [`alpha_asset`](#alpha_assetbeta_asset)
- [`alpha_ledger`](#alpha_ledgerbeta_ledger)
- [`beta_asset`](#alpha_assetbeta_asset)
- [`beta_ledger`](#alpha_ledgerbeta_ledger)
- [`status`](#status)
- [`swap_protocol`](#swap_protocol)

## Status

```
RFC-Number: 002
Status: Draft
```

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

[RFC-003](wip) is an available swap protocol. However, it is not limited to [RFC-003](wip) and this RFC is expected to be the building block for several swap protocols.

## SWAP REQUEST

### Purpose

A message for requesting an atomic swap between two assets.
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
`LedgerName`: the name of the ledger in ASCII (lower case). <!-- TODO: Issue needed as currently case sensitive -->
Example: `bitcoin`.

##### `parameters`
###### `network`

`Network`: the target network.
Example: `mainnet`.

Refer to the [registry](./registry-RFC-002.md#alpha_ledgerbeta_ledger) for the definition of all possible network values.

##### Sample
```json
{
  "value": "bitcoin",
  "parameters": { "network": "regtest" }
}
```

#### `alpha_asset/beta_asset`
The *Alpha Asset*/*Beta Asset* for this swap. As defined in the [terminology](#asset).

The `parameters`' value depends on the kind of the described asset. Native assets SHOULD only have a `quantity` integer parameter in the smallest unit, supported by the *Ledger*.
See the [registry](./registry-RFC-002.md#alpha_assetbeta_asset) for more details.

##### Format
```
{
  "value": AssetName,
  "parameters": { "quantity": Quantity },
}
```

##### `value`
`AssetName`: the name of the asset in ASCII (lower case).
Example: `bitcoin`.

##### `parameters`
See the [registry](./registry-RFC-002.md#alpha_assetbeta_asset) for a full definition of all parameters.

`Quantity`: the amount in the smallest unit of the asset.

###### Samples
1 BTC.
```json
{
  "value": "bitcoin",
  "parameters": { "quantity": "100000000" }
}
```

#### `swap_protocol`
The protocol used to proceed with the swap. Defined in subsequent RFCs.

##### `value`
Refer to the RFC of the given `SwapProtocol`.

##### Sample
RFC-003 HTLC based protocol.
```json
{
  "value": "comit-rfc-003",
  "parameters": {},  
}
```
<!-- TODO: Open issue to lowercase it -->

#### `body`

Refer to the RFC of the given `SwapProtocol`.

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

#### `status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Each swap protocol MAY define their own statuses for 40 and above.

* `OK20`: Swap request is accepted
* `RE00`: Receiver (Bob) Internal Error (as per [RFC-001](./RFC-001-BAM.md#status-code-families))<!-- TODO: Open issue because we incorrectly use SE00 in the code -->
* `RE20`: Swap declined (no reason provided)
* `RE21`: Swap declined due to proposed rate.
* `RE22`: Swap declined due to lack of liquidity
* `RE31`: Swap rejected due to unsupported ledger combination
* `RE32`: Swap rejected due to unsupported swap protocol

<!-- TODO: Open issue to remove reason and fix statuses -->

#### `body`

The body is defined in the RFC of the given `SwapProtocol`.

---

###### ¹ trustless: as in no one (counterpart or third party) has to be trusted.
[¹]:#-trustless-as-in-no-one-counterpart-or-third-party-has-to-be-trusted
###### ² In case of [RFC-003](wip), alpha ledger is the first ledger on which an action is needed, hence the name.
[²]:#-in-case-of-rfc-003-this-the-first-ledger-on-which-an-action-is-needed-hence-the-name
