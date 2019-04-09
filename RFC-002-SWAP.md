# SWAP Message Types

- RFC-Number: 002
- Status: Draft
- Discussion issue: [#18](https://github.com/comit-network/RFCs/issues/18)
- Created on: 28 Dec. 2018

**Table of contents**
- [Introduction](#introduction)
- [Terminology](#terminology)
    - [Ledger](#ledger)
    - [Asset](#asset)
    - [Alpha/Beta](#alphabeta)
- [BAM! messages](#bam-messages)
    - [SWAP REQUEST frame](#swap-request-frame)
        - [Definition](#definition)
        - [Headers](#headers)
        - [Body](#body)
    - [SWAP RESPONSE frame](#swap-response-frame)
        - [Definition](#definition-1)
        - [Headers](#headers-1)
        - [Body](#body-1)
- [Examples](#examples)
    - [SWAP REQUEST frame](#swap-request-frame-1)
    - [SWAP RESPONSE frame](#swap-response-frame-1)
        - [Successful negotiation](#successful-negotiation)
        - [Failed negotiation](#failed-negotiation)
- [Registry extensions](#registry-extensions)
    - [The type `Ledger`](#the-type-ledger)
    - [The type `Asset`](#the-type-asset)
    - [The type `SwapProtocol`](#the-type-swapprotocol)
    - [The type `NegotiationError`](#the-type-negotiationerror)
    - [Headers](#headers-2)
    - [The following `NegotiationError`s](#the-following-negotiationerrors)
        - [`unsatisfactory-rate`](#unsatisfactory-rate)
            - [`details`](#details)
        - [`protocol-unsupported`](#protocol-unsupported)
            - [`details`](#details-1)
        - [`unknown-ledger`](#unknown-ledger)
            - [`details`](#details-2)
        - [`unknown-asset`](#unknown-asset)
            - [`details`](#details-3)

## Introduction

This RFC introduces defines two things:

1. A terminology for describing the exchange of assets
2. A REQUEST/RESPONSE pair of messages for `BAM!` to negotiate such an exchange between two parties

## Terminology

To describe the elements of an exchange, we are using the terms `Ledger`, `Asset` and `Alpha/Beta`.

### Ledger

A ledger is anything that records ownership and allows transferal of that ownership.

### Asset

An asset is anything whose ownership can be transferred on a [ledger](#ledger).

### Alpha/Beta

The terminology of **alpha** and **beta** is used to unambiguously refer to the assets and ledgers in an exchange.
Such an exchange can thus be described as
- transferring the ownership of the **alpha-asset** on the **alpha-ledger** from party A to party B
- transferring the ownership of the **beta-asset** on the **beta-ledger** from party B to party A

This terminology has several advantages:

1. it does not imply an order (compared to a terminology like **first**/**second**)
2. it is not subjective to any of the parties (compared to a terminology like **source**/**target**, **incoming**/**outgoing** or **local**/**remote**)

## BAM! messages

The messages to negotiate such an exchange consist of a `REQUEST` frame and an according `RESPONSE` frame.

### SWAP REQUEST frame

#### Definition

A SWAP REQUEST message is a `FRAME` of type `REQUEST`.
[As per definition](./RFC-001-BAM.md#type-1) in `BAM!`, a `REQUEST` `FRAME` has a `type` that defines its semantics.
For the SWAP REQUEST message, this type is `SWAP`.

#### Headers

To express all the information for an exchange, a SWAP REQUEST MUST include the following headers:

- `alpha_ledger`: describes the ledger the alpha-asset is tracked on
- `beta_ledger`: describes the ledger the beta-asset is tracked on
- `alpha_asset`: describes the asset who's ownership is transferred on the alpha-ledger
- `beta_asset`: describes the asset who's ownership is transferred on the beta-ledger
- `protocol`: describes the protocol that is used to transfer the ownership of the assets

This RFC only defines these headers.
The actual definition of ledgers, assets and protocols is not subject to this RFC.

#### Body

The body of a SWAP REQUEST depends on the chosen `protocol`.
It is therefore subject to RFCs which define such protocols to also define, which information is contained in the `body` of a SWAP REQUEST/RESPONSE frame.

### SWAP RESPONSE frame

#### Definition

A frame of type `REQUEST` implies a `RESPONSE`.
We will refer to the response of a `SWAP REQUEST` as SWAP RESPONSE.

The SWAP RESPONSE serves two purposes:

1. It contains the result of the negotiation initiated through the SWAP REQUEST
2. It allows the receiving party to communicate information 

#### Headers

The response MUST contain the following header:

- `negotiation_result`

The `negotiation_result` header is defined as follows:

- `value`: `successful` OR `failed`.
- `parameters`: None

#### Body

Together with the the value of the `protocol` header, the `negotiation_result` header determines the `body` of a SWAP RESPONSE.

In the case of a `successful` negotiation, the `body` of the response will contain the necessary information for the swap to start.
This entirely depends on the chosen `protocol` and is thereby subject to the `protocol`s RFC to define the `body`.
In the case of a `failed` negotiation, the `body` of the response MAY contain an object of type `NegotiationError`.
A `NegotiationError` is an object with a two properties:

- `reason`: A string, acting as the identifier for this `NegotiationError`.
- `details`: An object containing more information about the error, depending on the given `reason`.

See the [Registry extensions](#registry-extensions)-section for examples of `NegotiationError`s.

## Examples

This section contains examples of SWAP REQUEST/RESPONSE frames.
Elements not relevant for this RFC or which are subject to later definition are filled in with "...".

### SWAP REQUEST frame

```json
{
    "type": "REQUEST",
    "id": 0,
    "payload": {
        "type": "SWAP",
        "headers": {
            "alpha_ledger": {
                "value": "...",
                "parameters": { ... }
            },
            "beta_ledger": {
                "value": "...",
                "parameters": { ... }
            },
            "alpha_asset": {
                "value": "...",
                "parameters": { ... }
            },
            "beta_asset": {
                "value": "...",
                "parameters": { ... }
            },
            "protocol": "...",
        },
        "body": { ... },
    } 
}
```

### SWAP RESPONSE frame

#### Successful negotiation

```json
{
    "type": "RESPONSE",
    "id": 0,
    "payload": {
        "headers": {
            "negotiation_result": "successful"
        },
        "body": { ... },
    }
}
```

#### Failed negotiation

```json
{
    "type": "RESPONSE",
    "id": 0,
    "payload": {
        "headers": {
            "negotiation_result": "failed"
        },
        "body": { ... },
    }
}
```

## Registry extensions

This RFC extends the registry with the following elements:

### The type `Ledger`

Subsequent RFCs can refer to this type if they want to define a particular ledger.
A section "Ledgers" is added to the registry which tracks all currently defined ledgers.

### The type `Asset`

Subsequent RFCs can refer to this type if they want to define a particular asset.
A section "Assets" is added to the registry which tracks all currently defined assets.

### The type `SwapProtocol`

Subsequent RFCs can refer to this type if they want to define a particular swap protocol.
A section "Protocols" is added to the registry which tracks all currently defined protocols.

### The type `NegotiationError`

Subsequent RFCs can refer to this type if they want to define a particular negotiation error.
A section "Negotiation Errors" is added to the registry which tracks all currently defined errors.

### Headers

| Header name          | Value                      |
|----------------------|----------------------------|
| `alpha_ledger`       | `Ledger`                   |
| `betaledger`         | `Ledger`                   |
| `alpha_asset`        | `Asset`                    |
| `beta_asset`         | `Asset`                    |
| `protocol`           | `SwapProtocol`             |
| `negotiation_result` | `"successful" | "failed"`  |

### The following `NegotiationError`s

In the following section, each heading is a `reason`.

#### `unsatisfactory-rate`

The rate of `alpha_asset` to `beta_asset` is not satisfactory to the receiver.

##### `details`

TBD 

#### `protocol-unsupported`

The protocol specified in the `protocol` header is not known to the receiving party.

##### `details`

TBD <!-- List known protocols in details -->

#### `unknown-ledger`

A ledger referenced by the sending party is unknown to the receiving party.
Note that different networks of the same blockchain are different ledger!
Bitcoin Testnet is a different ledger than Bitcoin Mainnet.

##### `details`

TBD <!-- List known ledgers in details -->

#### `unknown-asset`

An asset referenced by the sending party is unknown to the receiving party.

##### `details`

TBD <!-- List known assets in details -->
