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
- [`protocol`](#protocol)

## Status

- RFC-Number: 002
- Status: Draft
- Discussion issue: [#18](https://github.com/comit-network/RFCs/issues/18)

## Terminology

### Ledger

A ledger is anything that records ownership and allows transferal of that ownership.

Refer to the [registry](./registry.md#ledgers) for the definition of supported ledgers.

### Asset

An asset is anything whose ownership can be transferred on a [ledger](#ledger).

Refer to the [registry](./registry.md#assets) for the definition of supported assets.

## SWAP REQUEST

### Definition
The SWAP REQUEST message is a `FRAME` of type `REQUEST`.
[As per definition](./RFC-001-BAM.md#type-1) in `BAM!`, a `REQUEST` `FRAME` has a `type` that defines its semantics.
For the SWAP REQUEST message, this type is `SWAP`.

### `headers`

#### `alpha_ledger`
```
Type: Ledger 
```

The ledger on which the Sender sells and the Receiver buys[²].

#### `beta_ledger`
```
Type: Ledger
```
The ledger on which the Sender buys and the Receiver sells.

#### `alpha_asset`
```
Type: Asset
```

The asset that the Sender sells and the Receiver buys.

#### `beta_asset`

The asset that the Sender buys and the Receiver sells.

#### `protocol`
A protocol that defines the steps, transactions and communications needed to proceed with an asset exchange.

The exact value of this header will be defined in subsequent swap protocol RFCs.

### `body`
Refer to the RFC of the given swap protocol.

## SWAP RESPONSE

### Definition
The SWAP RESPONSE message is a `FRAME` of type `RESPONSE`.

### `status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Protocols may define the meaning of statuses 40 and above.

* `OK20`: Accepted
* `RE20`: Declined - the Receiver voluntarily turned down the swap request
* `RE21`: Rejected - the Receiver is not able to proceed with the swap request

### `headers`
#### `reason` (optional)
The reason why the Receiver Declined or Rejected the swap request.

Reason's `parameters` are hints on which request headers, if changed, may lead a subsequent request to be accepted.

##### `value`

###### Decline reasons
The following reasons must be accompanied with a `RE20` status.

| Value                     | Description     | Hints Conditions | Supported Parameters |
|:----                      |:---             |:---              |:---                  |
| `unsatisfactory-rate`     | The Receiver rejects the exchange rate and may accept a swap request with a different rate.                             | optional, both or none | `alpha_asset`, `beta_asset` |
| `unsatisfactory-quantity` | The Receiver declines the offered asset quantity and may accept the request if a different asset quantity is requested. | optional, both or none | `alpha_asset`, `beta_asset` | 

###### Reject reasons
The following reasons must be accompanied with a `RE21` status.

| Value                  | Description | Hints Conditions | Supported Parameters |
|:---                    |:---         |:----             |:---                  |
| `protocol-unsupported` | The Receiver does not support the requested protocol.                             | optional | `protocol` |
| `unsupported-ledger`   | The Receiver does not support the requested ledger combination.                   | none | |
| `unavailable-asset`    | The Receiver does not have the given asset or enough of the given asset quantity. | optional, both or none | `alpha_asset`, `beta_asset` |

### `body`

Refer to the RFC of the given swap protocol.

# Examples

## SWAP REQUEST

### 1 Bitcoin for 21 Ether on testnet using RFC-003

```json
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

```json
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
      "parameters": {
        "quantity": "42000000000000000000",
        "address": "0xB97048628DB6B661D4C2aA833e95Dbe1A905B280"
      }
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

```json
{
  "status": "OK20",
  "headers": {},
  "body": {},
}
```

### Request declined due to non-beneficial rate with hints

```json
{
  "status": "RE20",
  "headers": {
    "reason": {
      "value": "rate-declined",
      "parameters": {
        "alpha_asset": {
          "value": "erc20",
          "parameters": {
            "quantity": "42500000000000000000",
            "address": "0xB97048628DB6B661D4C2aA833e95Dbe1A905B280"
          }
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

```json
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
