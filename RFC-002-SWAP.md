# SWAP Message Types

<!-- toc -->

  * [Introduction](#introduction)
  * [Status](#status)
  * [Terminology](#terminology)
    + [Ledger](#ledger)
    + [Asset](#asset)
  * [SWAP REQUEST](#swap-request)
    + [Definition](#definition)
    + [`headers`](#headers)
      - [`alpha_ledger`](#alpha_ledger)
      - [`beta_ledger`](#beta_ledger)
      - [`alpha_asset`](#alpha_asset)
      - [`beta_asset`](#beta_asset)
      - [`protocol`](#protocol)
    + [`body`](#body)
  * [SWAP RESPONSE](#swap-response)
    + [Definition](#definition-1)
    + [`status`](#status)
    + [`headers`](#headers-1)
      - [`reason` (optional)](#reason-optional)
    + [`body`](#body-1)
- [Examples](#examples)
  * [SWAP REQUEST](#swap-request-1)
    + [1 Bitcoin for 21 Ether on testnet using RFC-003](#1-bitcoin-for-21-ether-on-testnet-using-rfc-003)
    + [42 ERC20 PAY tokens for 10,000 Satoshis on mainnnet using RFC-003](#42-erc20-pay-tokens-for-10000-satoshis-on-mainnnet-using-rfc-003)
  * [SWAP RESPONSE](#swap-response-1)
    + [Request accepted](#request-accepted)
    + [Request declined due to non-beneficial rate with hints](#request-declined-due-to-non-beneficial-rate-with-hints)
    + [Request rejected without a reason](#request-rejected-without-a-reason)

<!-- tocstop -->

## Introduction

This RFC defines the `SWAP` message types for `BAM!` (see [RFC-001](./RFC-001-BAM.md)), using JSON encoding.
This set of message types facilitates the negotiation of an exchange of assets and the protocol parameters to execute it.

The SWAP REQUEST contains an asset exchange proposal and the Sender information to realise the exchange.

The SWAP RESPONSE allows the rejection or acceptance of the exchange and may contains the Receiver information to realise the exchange. 

This RFC is part of COMIT, an open protocol facilitating trustless[¹] cross-blockchain applications.

This RFC contains the definition of the following `BAM!` headers:
- [`alpha_asset`](#alpha_asset)
- [`alpha_ledger`](#alpha_ledger)
- [`beta_asset`](#beta_asset)
- [`beta_ledger`](#beta_ledger)
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

An asset is anything whose ownership can be transferred on a [ledger](#ledger).

## SWAP REQUEST

### Definition
```
BAM! type: REQUEST
```

### `headers`

#### `alpha_ledger`

The ledger on which the Sender sells and the Receiver buys[²].

##### `value`
The name of the ledger in ASCII (lower case). <!-- TODO: Issue needed as currently case sensitive -->

##### `parameters`

Refer to the [registry](./registry-RFC-002.md#ledger).

#### `beta_ledger`

The ledger on which the Sender buys and the Receiver sells.

##### `value`
The name of the ledger in ASCII (lower case). <!-- TODO: Issue needed as currently case sensitive -->

##### `parameters`

Refer to the [registry](./registry-RFC-002.md#ledger).

#### `alpha_asset`

The asset that the Sender sells and the Receiver buys.

##### `value`
The name of the asset in ASCII (lower case).

##### `parameters`
The `parameters` depend on the asset. Native assets should only have a `quantity` integer parameter, expressed in the smallest unit, supported by its corresponding *Ledger*.
See the [registry](./registry-RFC-002.md#asset) for exact description.

#### `beta_asset`

The asset that the Sender buys and the Receiver sells.

##### `value`
The name of the asset in ASCII (lower case).

##### `parameters`
The `parameters` depend on the asset. Native assets should only have a `quantity` integer parameter, expressed in the smallest unit, supported by its corresponding *Ledger*.
See the [registry](./registry-RFC-002.md#asset) for exact description.

#### `protocol`
<!-- TODO: Open issue to rename `swap_protocol` to `protocol` -->

A protocol that defines the steps, transactions and communications needed to proceed with an asset exchange.

The protocol itself is defined in subsequent RFCs.

##### `value`
Refer to the RFC of the given `protocol`.

<!-- TODO: Open issue to lowercase it -->

### `body`

The body is defined by the RFC of the given `protocol`.

## SWAP RESPONSE

### Definition
```
BAM! type: RESPONSE
```

### `status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Each protocol may define their own statuses for 40 and above.

* `OK20`: Accepted
* `RE20`: Declined - the swap request was not beneficial for the receiver
* `RE21`: Rejected - the receiver is not able to proceed with the swap request

### `headers`
#### `reason` (optional)
The reason why the receiver Declined or Rejected the swap request.

##### `value`
A human readable reason. In lower case hyphen separated ASCII.

See the [registry](./registry-RFC-002.md#reason) for possible values.
A given swap protocol may define further available reasons.

##### `parameters` (optional)
Parameters are hints on which request headers, if changed, may lead a request to be accepted.

This allows the Receiver to hint the Sender on an exchange it may accept.

The following hints are supported:
- `alpha_asset`
- `beta_asset`
- `protocol`

Their format is as defined in the [SWAP REQUEST - `headers`](#headers) section.

A protocol may define further available hints.
<!-- TODO: Open issue to fix reason and statuses -->

### `body`

The body is defined in the RFC of the given `protocol`.

# Examples

## SWAP REQUEST

### 1 Bitcoin for 21 Ether on testnet using RFC-003

```json5
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "bitcoin",
      "parameters": { "network": "regtest" }
    },
    "beta_ledger": {
      "value": "ethereum",
      "parameters": { "network": "ropsten" }
    },
    "alpha_asset": {
      "value": "bitcoin",
      "parameters": { "quantity": "100000000" }
    },
    "beta_asset": {
      "value": "ether",
      "parameters": { "quantity": "21000000000000000000" }
    },
    "protocol": "comit-rfc-003",
  },
  "body": {},
}
```

### 42 ERC20 PAY tokens for 10,000 Satoshis on mainnnet using RFC-003

```json5
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "ethereum",
      "parameters": { "network": "mainnet" }
    },
    "beta_ledger": {
      "value": "bitcoin",
      "parameters": { "network": "mainnet" }
    },
    "alpha_asset": {
      "value": "erc20",
      "parameters": { "quantity": "42000000000000000000" }
    },
    "beta_asset": {
      "value": "bitcoin",
      "parameters": { "quantity": "10000" }
    },
    "protocol": "comit-rfc-003",
  },
  "body": {},
}
```

## SWAP RESPONSE

### Request accepted

```json5
{
  "status": "OK20",
  "headers": {},
  "body": {},
}
```

### Request declined due to non-beneficial rate with hints

```json5
{
  "status": "RE20",
  "headers": {
    "reason": {
      "value": "rate-declined",
      "parameters": {
        "alpha_asset": {
          "value": "erc20",
          "parameters": { "quantity": "42000000000000000000" }
        },
        "beta_asset": {
          "value": "bitcoin",
          "parameters": { "quantity": "10000" }
        },
      },
    },
  },
  "body": {},
}
```

### Request rejected without a reason

```json5
{
  "status": "RE21",
  "headers": {},
  "body": {},
}
```

---

###### ¹ trustless: as in no one (counterpart or third party) has to be trusted.
[¹]:#-trustless-as-in-no-one-counterpart-or-third-party-has-to-be-trusted
###### ² In case of [RFC-003](wip), alpha ledger is the first ledger on which an action is needed, hence the name.
[²]:#-in-case-of-rfc-003-this-the-first-ledger-on-which-an-action-is-needed-hence-the-name
