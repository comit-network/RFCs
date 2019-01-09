# SWAP Message Types

<!-- toc -->

  * [Description](#description)
  * [Status](#status)
  * [Terminology](#terminology)
    + [Ledger](#ledger)
    + [Asset](#asset)
  * [SWAP REQUEST](#swap-request)
    + [Purpose](#purpose)
    + [Definition](#definition)
    + [`headers`](#headers)
      - [`alpha_ledger`](#alpha_ledger)
      - [`beta_ledger`](#beta_ledger)
      - [`alpha_asset`](#alpha_asset)
      - [`beta_asset`](#beta_asset)
      - [`protocol`](#protocol)
    + [`body`](#body)
  * [SWAP RESPONSE](#swap-response)
    + [Purpose](#purpose-1)
    + [Definition](#definition-1)
    + [`status`](#status)
    + [`headers`](#headers-1)
      - [`reason`](#reason)
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

Its ownership can be tracked on a [ledger](#ledger).

## SWAP REQUEST

### Purpose

A message for requesting the exchange of two assets.

### Definition
```
BAM! type: REQUEST
```

### `headers`

#### `alpha_ledger`

The ledger on which the Sender sells and the Receiver buys[¹].

##### `value`
The name of the ledger in ASCII (lower case). <!-- TODO: Issue needed as currently case sensitive -->

##### `parameters`

Refer to the [registry](./registry-RFC-002.md#alpha_ledgerbeta_ledger).

#### `beta_ledger`

The ledger on which the Sender buys and the Receiver sells.

##### `value`
The name of the ledger in ASCII (lower case). <!-- TODO: Issue needed as currently case sensitive -->

##### `parameters`

Refer to the [registry](./registry-RFC-002.md#alpha_ledgerbeta_ledger).

#### `alpha_asset`

The asset that the Sender sells and the Receiver buys[¹].

The `parameters`' value depends on the kind of the described asset. Native assets SHOULD only have a `quantity` integer parameter in the smallest unit, supported by the *Ledger*.
See the [registry](./registry-RFC-002.md#alpha_assetbeta_asset) for more details.

##### `value`
The name of the asset in ASCII (lower case).

##### `parameters`
Refer to the [registry](./registry-RFC-002.md#alpha_ledgerbeta_ledger).

#### `beta_asset`

The asset that the Sender buys and the Receiver sells.

The `parameters`' value depends on the kind of the described asset. Native assets SHOULD only have a `quantity` integer parameter in the smallest unit, supported by the *Ledger*.
See the [registry](./registry-RFC-002.md#alpha_assetbeta_asset) for more details.

##### `value`
The name of the asset in ASCII (lower case).

##### `parameters`
Refer to the [registry](./registry-RFC-002.md#alpha_ledgerbeta_ledger).

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

### Purpose

For the Receiver to accept or decline a **SWAP REQUEST**.

### Definition
```
BAM! type: RESPONSE
```

### `status`

See [RFC-001](./RFC-001-BAM.md#status-code-families) for more details, including the definition of statuses `XX00-19`.

RFC-002 reserves statuses 20 to 39 across all families.
Each protocol MAY define their own statuses for 40 and above.

* `OK20`: Accepted
* `RE00`: Receiver Internal Error (as per [RFC-001](./RFC-001-BAM.md#status-code-families))<!-- TODO: Open issue because we incorrectly use SE00 in the code -->
* `RE20`: Declined - the swap request was not beneficial for the receiver
* `RE21`: Rejected - the receiver is not able to proceed with the swap request

### `headers`
#### `reason`

Optional reason why the request was declined.

##### `value`
A human readable reason. In Lowercase ascii, hyphen separated.

See the [registry](./registry-RFC-002.md) for possible values.
A protocol may define further available reasons.

##### `parameters`
Parameters are optionals hints on what inputs, if changed, may lead the request to be accepted.

The aim is for the request receiver to hint the sender on the values that it would accept for the swap.
The following hints are supported:
- `alpha_asset`
- `beta_asset`
- `protocol`

Their format is as defined in the [SWAP REQUEST - `headers`](#headers) section.

A protocol may define further available hints.
<!-- TODO: Open issue to remove reason and fix statuses -->

### `body`

The body is defined in the RFC of the given `protocol`.

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
                     "parameters: { "quantity": "100000000" }
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
      "parameters: { "quantity": "42000000000000000000" }
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
  "status": "OK20"
  "headers": {},
  "body": {},
}
```

### Request declined due to non-beneficial rate with hints

```json
{
  "status": "RE20"
  "headers": {
    "reason": {
      "value": "rate-declined",
      "parameters": {
        "alpha_asset": {
          "value": "erc20",
          "parameters: { "quantity": "42000000000000000000" }
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
  "status": "RE21"
  "headers": {},
  "body": {},
}
```

---

###### ¹ trustless: as in no one (counterpart or third party) has to be trusted.
[¹]:#-trustless-as-in-no-one-counterpart-or-third-party-has-to-be-trusted
###### ² In case of [RFC-003](wip), alpha ledger is the first ledger on which an action is needed, hence the name.
[²]:#-in-case-of-rfc-003-this-the-first-ledger-on-which-an-action-is-needed-hence-the-name
