# SWAP Message Types

<!-- toc -->

- [Description](#description)
- [Status](#status)
- [Terminology](#terminology)
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
    + [`reason`](#reason)
    + [`body`](#body-1)

<!-- tocstop -->

## Description

This RFC defines the `SWAP` message types for `BAM!` (see [RFC-001](./RFC-001-BAM.md)), using JSON encoding.
This set of message types facilitates the execution cross-ledger atomic swaps of assets.

This RFC is part of COMIT, an open protocol facilitating trustless[¹] cross-blockchain applications.

This RFC contains the definition of the following `BAM!` headers:
- [`alpha_asset`](#alpha_assetbeta_asset)
- [`alpha_ledger`](#alpha_ledgerbeta_ledger)
- [`beta_asset`](#alpha_assetbeta_asset)
- [`beta_ledger`](#alpha_ledgerbeta_ledger)
- [`status`](#status)
- [`protocol`](#protocol)

## Status

```
RFC-Number: 002
Status: Draft
```

## Terminology

### Ledger

A record which tracks asset ownerships. For example, the Bitcoin or Ethereum networks.

### Asset

A digital asset. Its ownership can be tracked on a [ledger](#ledger).

## SWAP REQUEST

### Purpose

A message for requesting the exchange of two assets.

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
    "protocol": Protocol.
    },
  "body": Body, 
}
```

#### `alpha_ledger/beta_ledger`

The `AlphaLedger` is the ledger on which the Sender sells and the Receiver buys[¹].

The `BetaLedger` is the ledger on which the Sender buys and the Receiver sells.

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

The `AlphaAsset` is the asset that the Sender sells and the Receiver buys[¹].

The `BetaAsset` is the asset that the Sender buys and the Receiver sells.

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

#### `protocol`
<!-- TODO: Open issue to rename `swap_protocol` to `protocol` -->

A protocol that defines the steps, transactions and communications needed to proceed with an asset exchange.

The protocol itself is defined in subsequent RFCs.

##### `value`
Refers to the RFC of the given `Protocol`.

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

The `Body` is defined by the RFC of the given `Protocol`.

## SWAP RESPONSE

### Purpose

For the Receiver to accept or decline a **SWAP REQUEST**.

### Definition
```
BAM! type: RESPONSE
```
#### Format
```
{
  "status": Status,
  "reason": Reason,
  "body": Body,
}
```
Valid for both accept and decline responses.

#### `status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Each protocol MAY define their own statuses for 40 and above.

* `OK20`: Swap request is accepted
* `RE00`: Receiver Internal Error (as per [RFC-001](./RFC-001-BAM.md#status-code-families))<!-- TODO: Open issue because we incorrectly use SE00 in the code -->
* `RE20`: Swap declined

#### `reason`

Optional reason why the request was declined.

##### Format
```
{
  "value": ReasonText,
  "parameters": {
    RequestHeaderHints
  }
}
```
##### `value`
A human readable reason. In Lowercase ascii, hyphen separated.
Example: `rate-declined`

See the [registry](./registry-RFC-002.md) for possible values.
A protocol may define further available reasons.

##### `parameters`
Parameters are optionals hints on what inputs, if changed, may lead the request to be accepted.

The aim is for the request receiver to hint the sender on the values that it would accept for the swap.
The following hints are supported:
- `alpha_asset`
- `beta_asset`
- `protocol`

Their format is as defined in the [SWAP REQUEST](#swap-request) section.

A protocol may define further available hints.

<!-- TODO: Open issue to remove reason and fix statuses -->

#### `body`

The body is defined in the RFC of the given `Protocol`.

---

###### ¹ trustless: as in no one (counterpart or third party) has to be trusted.
[¹]:#-trustless-as-in-no-one-counterpart-or-third-party-has-to-be-trusted
###### ² In case of [RFC-003](wip), alpha ledger is the first ledger on which an action is needed, hence the name.
[²]:#-in-case-of-rfc-003-this-the-first-ledger-on-which-an-action-is-needed-hence-the-name
