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
    - [Alpha and Beta](#alpha-and-beta)
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
    - [The type `NegotiationError` and the initial set of possible errors](#the-type-negotiationerror-and-the-initial-set-of-possible-errors)
        - [`unsatisfactory-rate`](#unsatisfactory-rate)
            - [`details`](#details)
        - [`protocol-unsupported`](#protocol-unsupported)
            - [`details`](#details-1)
        - [`unknown-ledger`](#unknown-ledger)
            - [`details`](#details-2)
        - [`unknown-asset`](#unknown-asset)
            - [`details`](#details-3)
    - [Headers](#headers-2)

## Introduction

This RFC serves two purposes:

1. Introduce terminology used to describe the exchange of assets.
2. Define a REQUEST/RESPONSE message pair for two parties to negotiate such an exchange via `BAM!`.

## Terminology

To describe the elements of an exchange, we use the terms `Ledger`, `Asset`, `Alpha` and `Beta`.

### Ledger

A ledger is anything that records ownership and allows transferal of that ownership.

### Asset

An asset is anything whose ownership can be transferred on a [ledger](#ledger).

### Alpha and Beta

The terms **alpha** and **beta** are used to unambiguously identify assets and ledgers in an exchange.
Such an exchange can thus be described as:
- Transferring the ownership of the **alpha-asset** on the **alpha-ledger** from party A to party B.
- Transferring the ownership of the **beta-asset** on the **beta-ledger** from party B to party A.

This terminology has several advantages:

1. It does not imply an order (compared to a terminology like **first**/**second**).
2. It is not subjective to any of the parties (compared to terminology like **source**/**target**, **incoming**/**outgoing** or **local**/**remote**).

## BAM! messages

The messages to negotiate such an exchange consist of a `REQUEST` frame and a corresponding `RESPONSE` frame.

### SWAP REQUEST frame

#### Definition

A SWAP REQUEST message is a `FRAME` of type `REQUEST`.
[As per definition](./RFC-001-BAM.md#type-1) in `BAM!`, a `REQUEST` `FRAME` has a `type` that defines its semantics.
For the SWAP REQUEST message, this type is `SWAP`.

#### Headers

To express all the information for an exchange, a SWAP REQUEST MUST include the following headers:

- `alpha_ledger`: describes the ledger the alpha-asset is tracked on
- `beta_ledger`: describes the ledger the beta-asset is tracked on
- `alpha_asset`: describes the asset whose ownership will be transferred on the `alpha_ledger`
- `beta_asset`: describes the asset whose ownership will be transferred on the `beta_ledger`
- `protocol`: describes the protocol that is used to transfer the ownership of the assets

This RFC only defines these headers.
The actual definition of ledgers, assets and protocols is not subject to this RFC.

#### Body

The body of a SWAP REQUEST depends on the chosen `protocol`.
It is therefore subject to RFCs which define such protocols to also define which information is contained in the `body` of a SWAP REQUEST/RESPONSE frame.

### SWAP RESPONSE frame

#### Definition

A frame of type `REQUEST` implies a `RESPONSE`.
We will refer to the response of a SWAP REQUEST as SWAP RESPONSE.

A SWAP RESPONSE contains the result of the negotiation initiated through the SWAP REQUEST.

#### Headers

The response MUST contain the following header:

- `negotiation_result`

The `negotiation_result` header is defined as follows:

- `value`: `successful` OR `failed`.
- `parameters`: None

#### Body

The content of the `body` depends on the values of the headers.
The following diagram illustrates the process:

<!--
@startuml

title Parse SWAP RESPONSE body

start

:inspect `negotiation_result` header;

if (negotiation_result) then (successful)
  :inspect `protocol` header;
  :parse body according to RFC of protocol;
else (failed)
  :parse body as `NegotiationError`;
endif

stop

@enduml
-->
![](https://www.plantuml.com/plantuml/img/PP0nRiCm34LtdkAFzXMI9KNWZAaH3nrhLQ8I0QfeKFJGsqS9K1X8HeBlVppoKCsfhR-Po99bnkYqCgQlZn6NOHe_pzE07mb_H4-IQ9TANTWRvi9NiUGiIVbMhcks6JTsWNLFb2AwTw27tRYWgwltN6jSSq_0Lhcec7Z9Mr7RBa-bXmISzw8XbIjCS3aT8H7_cJrnRbmNNSeS-jTanNpUV0PLqRb5IaZnSPiiH8SsjLVS0G00)

A `NegotiationError` is an object with a two properties:
  - `reason`: a string acting as the identifier for the `NegotiationError`.
  - `details`: an object containing more information about the error, depending on the given `reason`.

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

A section "Ledgers" is added to the registry which tracks all currently defined ledger types.
Subsequent RFCs can refer to this type if they want to define a particular ledger.

### The type `Asset`

A section "Assets" is added to the registry which tracks all currently defined asset types.
Subsequent RFCs can refer to this type if they want to define a particular asset.

### The type `SwapProtocol`

A section "Protocols" is added to the registry which tracks all currently defined protocols.
Subsequent RFCs can refer to this type if they want to define a particular swap protocol.

### The type `NegotiationError` and the initial set of possible errors

A section "Negotiation Errors" is added to the registry which tracks all currently defined errors.
Subsequent RFCs can refer to this type if they want to define a particular negotiation error.

The following `NegotiationError`s are added to the list.
Each heading represents the `reason` of the `NegotiationError`

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
Note that different networks of the same blockchain are different ledgers!
Bitcoin Testnet is a different ledger than Bitcoin Mainnet.

##### `details`

TBD <!-- List known ledgers in details -->

#### `unknown-asset`

An asset referenced by the sending party is unknown to the receiving party.

##### `details`

TBD <!-- List known assets in details -->


### Headers

| Header name          | Value type                 |
| -------------------- | -------------------------- |
| `alpha_ledger`       | `Ledger`                   |
| `betaledger`         | `Ledger`                   |
| `alpha_asset`        | `Asset`                    |
| `beta_asset`         | `Asset`                    |
| `protocol`           | `SwapProtocol`             |
| `negotiation_result` | `"successful" OR "failed"` |
