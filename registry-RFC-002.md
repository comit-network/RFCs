# Registry for RFC002 SWAP Message Types

<!-- toc -->

- [Description](#description)
- [Reason](#reason)
  * [Decline reasons](#decline-reasons)
    + [Rate is unsatisfactory](#rate-is-unsatisfactory)
    + [Quantity is unsatisfactory](#quantity-is-unsatisfactory)
  * [Reject reasons](#reject-reasons)
    + [Unsupported protocol](#unsupported-protocol)
    + [Unsupported ledger combination](#unsupported-ledger-combination)
    + [Unavailable asset](#unavailable-asset)

<!-- tocstop -->

## Description

This registry defines the possible values used to encode SWAP message types as defined in [RFC-002](./RFC-002-SWAP.md).

Refer to [RFC-002](./RFC-002-SWAP.md) for the full definition of the message.

## Reason

### Decline reasons
The following reasons must be accompanied with a `RE20` status.

#### Rate is unsatisfactory
The Receiver rejects the exchange rate and may accept a swap request with a different rate.
`value`: `rate-unsatisfactory`
##### `parameters`
Hints are optional.
If present, expected hints are `alpha_asset` and `beta_asset`. Both or none of them must be included.

#### Quantity is unsatisfactory
The Receiver declines the offered asset quantity and may accept the request if a different asset quantity is requested.
`value`: `quantity-unsatisfactory`
##### `parameters`
Hints are optional.
If present, expected hints are `alpha_asset` and `beta_asset`. Both or none of them must be included.

### Reject reasons
The following reasons must be accompanied with a `RE21` status.

#### Unsupported protocol
The Receiver does not support the requested protocol.
`value`: `protocol-unsupported`
##### `parameters`
Hints are optional.
If present, expected hint is `protocol`.

#### Unsupported ledger combination
The Receiver does not support the requested ledger combination.
`value`: `unsupported-ledger`
##### `parameters`
No hint is supported.

#### Unavailable asset
The Receiver does not have the given asset or enough of the given asset quantity.
`value`: `unavailable-asset`
##### `parameters`
*Please note that hinting on this rejection may lead to privacy concerns (exposure of available liquidity)*.
Hints are optional.
If present, expected hints are `alpha_asset` and `beta_asset`. Both or none of them must be included.
