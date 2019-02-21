# Bitcoin Definitions

As per [Bitcoin Core](https://github.com/bitcoin/bitcoin/) concensus.

Introduced for RFC-003 (should RFC-003 be part of the title?)

## Ledger definition

Taken from registry

| Name     | Description            | Reference |  `value`   | `parameters` key | `parameters` value |`parameters` description         |
|:---      |:---                    |:---       |:---        |:---              |:---                |:---                             |
| Bitcoin  | Bitcoin-core network   | TBD       | `bitcoin`  | `network`        |                    | The network on which to operate |
|          |                        |           |            |                  | `regtest`          | Bitcoin-core regtest            |
|          |                        |           |            |                  | `testnet`          | Bitcoin testnet                 |
|          |                        |           |            |                  | `mainnet`          | Bitcoin mainnet                 |

## Asset definition

Taken from registry

| Name           | Description                   | Reference | `value`   | `parameters` key | `parameters` value type | `parameters` description |
|:---            |:----                          |:---       |:---       |:---              |:---                     |:---                      |
| Bitcoin        | Native Bitcoin network asset  | TBD       | `bitcoin` | `quantity`       | integer in Json string  | Amount in satoshi        |

## Identities

Taken from registry

| Ledger   | Identity Name | JSON Encoding            | Reference | Description                                                                       |
|:----     |:-------      |:-------------            |:--------- | --------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | TBD       | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 public key |

## RFC-003

Mention that it should be p2wsh.
Do we want p2sh(p2wsh) support?
Do we want p2sh native support?

### HTLC

Definition of the HTLC in OP_CODEs and hex.
Describes how to generate the contract from hex (ie, what bytes contain what)

### Redeem
Describe redeem script

### Refund
Describe refund script and any caveat/things to watch out

### Timeouts

Do we want to discuss timeouts?

