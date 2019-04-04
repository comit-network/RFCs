# BAM! - Bidirectional Application Messaging

- RFC-Number: 001
- Status: Draft
- Discussion-issue: [#17](https://github.com/comit-network/RFCs/issues/17)
- Created on: 04 Sep. 2018

**Table of contents**
- [Description](#description)
- [Motivation & Requirements](#motivation--requirements)
- [Evaluation of existing protocols](#evaluation-of-existing-protocols)
    - [HTTP(2)](#http2)
    - [WebSockets](#websockets)
    - [A profile for BEEP](#a-profile-for-beep)
    - [gRPC](#grpc)
- [Introducing **BAM!**](#introducing-bam)
    - [Frames](#frames)
        - [Type](#type)
        - [Id](#id)
    - [Different frame types](#different-frame-types)
        - [Notification](#notification)
        - [Error](#error)
            - [Structure](#structure)
            - [Possible error types](#possible-error-types)
        - [Request / Response](#request--response)
            - [Structure](#structure-1)
            - [Optional fields](#optional-fields)
            - [Type](#type-1)
            - [Headers](#headers)
    - [Headers](#headers-1)
    - [JSON Encoding](#json-encoding)
        - [Frames](#frames-1)
        - [Encoding of the individual frame types](#encoding-of-the-individual-frame-types)
            - [Request](#request)
            - [Response](#response)
        - [Header](#header)
            - [Representation](#representation)
            - [Default values](#default-values)
            - [Allowed values](#allowed-values)
            - [Compact representation](#compact-representation)
        - [Naming conventions](#naming-conventions)
    - [Connection errors / failure cases](#connection-errors--failure-cases)
        - [Malformed response](#malformed-response)
- [References](#references)

## Description

This RFC describes BAM!: **B**i-directional **A**pplication **M**essaging.
It is a domain-agnostic and self-descriptive protocol and potentially supports multiple encodings.

## Motivation & Requirements 

In COMIT, nodes need to exchange all kinds of information in a peer-to-peer manner.
As COMIT is an inherently distributed system, the network will eventually become heterogeneous, with an array of implementations, varying in languages and versions.
Given that, we need a transport protocol that fulfills the following requirements:

- **Different kinds of messages:** The protocol needs to support requests (response expected) and one-way transmissions (no response).
- **Self-descriptive messages:** Because of the heterogeneous nature, the protocol needs to allow the introduction of new features without breaking older versions.
Self-descriptive messages allow backward compatibility because older implementations can still parse newer messages and continue even if they don't fully understand its content. (Spoiler: There are also ways to force backward incompatibility.)
- **Dynamic/Polymorphic messages:** Similar to HTTP, the messages sent to the other party don't always take the exact same form although they convey the same meaning.
GET requests for example always describe the intent of retrieving a resource.
The concrete structure of two GET requests might differ though.
Similar to that, we need to send messages between COMIT nodes that communicate a certain intent (like initiate an atomic swap) but might take different structure (for example, different information  needs to be exchanged based on the ledger).
- **Peer-to-peer:** In a communication between two nodes, not always the same one initiates, e.g.
both nodes need to be able to actively send messages to each other.
Contrary to this requirement, HTTP for example demands that the client always initiates the connection/communication.
In addition, it should be possible for nodes to work *behind* a NAT.
This means, nodes should only use *one* physical connection to talk in both directions.

## Evaluation of existing protocols

We looked at several existing application protocols.
In particular, we evaluated the following:

- HTTP(2)
- WebSockets
- A profile for BEEP (RFC 3080)
- gRPC and protobuf

### HTTP(2)

HTTP by definition is a client-server protocol.
Unfortunately, this almost rules it out right from the beginning.
With HTTP2 though, the protocol introduces so-called `Frames` which, among other things, allow the often cited PUSH-functionality of HTTP2. One way to use HTTP2 would be to modify the protocol slightly, so that each party could send `Frames` at any time: Means also the "server" could send request frames to the client and vice versa.

We decided against this approach because we feared it might falsely suggest compatibility with HTTP2 software, although this compatibility is not given.
Using HTTP (1 or 2) the intended way doesn't work because it only supports uni-directional communication.

### WebSockets

WebSockets are the Web's solution for bidirectional communication.
Although it would support the peer-to-peer requirement, WebSockets have a very low abstraction level, as they don't specify anything about the actual message that is sent over.
Instead, the websocket spec mostly deals with things that are related to the Web (what a surprise!) like protection of shared infrastructure through masking of the actual payload or upgrades of HTTP connection to a web socket connection.

While WebSockets could have been a solution, they actually don't provide any benefit over plain sockets for us.
In order to keep the complexity low, we decided against it.

### A profile for BEEP

BEEP is a fully bidirectional application protocol and looks very much like what we needed at first sight.
It covers a wide range of requirements and has many features.
Unfortunately, it never seemed to get traction and thus, library support is very limited.
For Rust, we would have had to write our own implementation.

Although BEEP would fulfill all the requirements, there is no implementation for it.
We could create and implementation but that would be quite a lot of work because BEEP covers much more than we need.

### gRPC

Whilst gRPC is a popular way of defining messaging between applications, it doesn't really fit our requirements: It doesn't work in a peer-to-peer context as it is centered around the idea of Client-Server.
The messages are neither self-descriptive nor dynamic: In order to parse a message, a party needs to know the message format upfront.
This is usually achieved through compiling `protobuf` files into source code.
The messages are also not dynamic as their structure needs to be known at compile-time.

Given that, we decided against gRPC.

## Introducing **BAM!**

Given the above evaluation, we are left with defining our own protocol that implements the stated requirements.
It incorporates ideas from several other protocols and combines them for our usecase.
In particular, it borrows ideas from the following protocols:

- BEEP
- HTTP2
- Lightning Network

The main concept in this protocol is called a `FRAME`.

### Frames

Frames identify themselves through a `type` and an `id` and carry a `payload`.

#### Type

The `type` defines how the `payload` of a frame is encoded.
Types also define the certain semantics of a frame, for example, whether an implementation should wait for a response to this frame or not.

The following types are allowed:

- REQUEST
- RESPONSE
- ERROR

The following types are reserved for future use:

- NOTIFICATION

#### Id

The `id` of a frame allows nodes to associate frames with each other.
Each node MUST keep track of the `id` it used on its own.
Nodes SHOULD start with the value `1` for this id.
The id MUST fit into an unsigned 32-bit integer.
This means the maximum allowed value is `4294967295`.

Ids MUST NOT be reused for subsequent frames.

A node should only assign `id`s to frames that are out-going and self-initiated, for example `REQUEST` frames.
All frames that act as a reply to another frame MUST use the appriopriate `id` to establish the necessary context.
For example, a `RESPONSE` frame MUST use the `id` of the `REQUEST` frame it refers to.

Ids MAY be skipped but MUST be ascending.

### Different frame types

#### Notification

> `NOTIFICATION` is a reserved type and may be specified at a later point.

#### Error

The `ERROR` frame is used to communicate failures between nodes on the lowest level of this protocol.
It is purely informational and is meant to fascilitate debugging.
An `ERROR` frame can not be sent pro-actively.
It's `id` MUST refer to a previously received frame, like a `REQUEST` frame.

##### Structure

`ERROR` frames carry the following payload:

- type

A machine-friendly identifier for this type of error

- message

A human-readable message giving more details about the error.
Implementations MAY use this for logging purposes.

##### Possible error types

The following types of `ERROR` frames are defined:

- `unknown-frame-type`
- `malformed-frame`
- `unknown-request-type`
- `unknown-mandatory-header`

#### Request / Response

A request is a type of message that implies an answer.
Nodes MUST be prepared to receive a frame of type `RESPONSE` with the id used in the `REQUEST` frame.
Alternatively to a `RESPONSE` frame, the other party MAY also send an `ERROR` frame.

##### Structure

A frame of type `REQUEST` carries the following payload:

- type
- headers
- body

whereas a frame of type `RESPONSE` looks like this:

- headers
- body

##### Optional fields

With the exception of `type`, all of these are optional and MAY be omitted if they are empty and the underlying encoding allows this without introducing ambiguity.
For example, the JSON encoding can easily handle that whereas a binary encoding may have a difficult time to omit certain fields.

##### Type

The field `type` in a request defines the semantics of the given request.
Defining a particular request type usually comes with defining the headers which are usable within this request.

##### Headers

`Headers` and `Body` are supposed to be used by an application protocol defined on top of `BAM`.
Application protocols can be described by defining `REQUEST` types and with them, the semantics of certain headers and the body of a `REQUEST`.
Similar to HTTP, application protocols MAY include some kind of 'Content-Type' in the headers in order to describe the encoding of the body.

In addition, headers also encode compatibility information.
Each header is available in two variants:

- MUST understand
- MAY ignore

If a node receives a header in the `MUST understand` variant in a `REQUEST` and it does not understand it, it MUST reject the request with an `ERROR` frame of type `unknown-mandatory-header`.
Headers encoded as `MAY ignore` are ok to be not understood.
Nodes may simply ignore them as if they were not there.

To avoid more complexity through additional messages, the spec doesn't define a concept of acknowledging successful processing of a `RESPONSE` to the sender.
Thus, there is no way of signaling to the sender of a `RESPONSE` whether or not it was properly understood.
Therefore, careful thought should be put into the design of and use of the `MUST understand` variant of a header to make this failure case as rare as possible.
In particular, nodes should never send a `RESPONSE` that contains a `MUST understand` header without them having confidence that the receiving node will understand it.
Usually, this can be derived from the `REQUEST` that is sent by a node.

### Headers

A header is a key-value structure.
Its actual structure is to be defined by an encoding of this protocol.
This section just describes the semantics that need to be encoded in a header.

A header's key acts as its identifier.
A header's value MUST encode the following information:

1. A flag to indicate the variant of this header

    Headers appear in two variants: MUST understand and MAY ignore.
This allows nodes with different versions of a particular protocol to enforce rejection of a message that contains a header that the other node does not understand.
It also facilitates incremental rollout of features.
At first, a header can be declared as `MAY ignore`.
At some point, an implementation might demand that another node understands a particular header by only sending the `MUST understand` variant.

2. A value: The actual value that is associated with the header

    Every header MUST have a value.
    It functions as the *identity* for the given header.

3. Parameters

    Parameters are additional data that depend on a header's value.
    A parameter for a given header value may be optional or mandatory.
    Optional parameters may be omitted.
    If the header value specifies no mandatory parameters, then the parameters section may be completely left out and implementations should treat this as if the empty set of parameters was sent.

Splitting headers up into `value` and `parameters` was done for the following reasons:

1. Having a single `value` facilitates comparison of header values: A node can determine whether or not they understand a particular header just by looking at the `value`.
If they don't understand the `value`, they also cannot understand its `parameters`.
2. It is more consistent: By having `value` and `parameters`, the general structure of a header is always the same, independent of the actual data.
This allows for easier implementation in statically-typed languages.
3. Without a dedicated `parameters` field, there could be a clash with a parameter named `value`, if they would just be next to the original `value` field.

### JSON Encoding

This section defines a text-based encoding of the above concepts.
Later RFCs might define a binary encoding in order to increase efficiency.

Nodes MUST use UTF-8 for the actual character encoding. (JSON technically requires that but just to be sure, it is stated here again.)

A protocol needs to somehow encode, where messages start and where they end.
In the JSON encoding of BAM, this is solved through newlines.
Thus, each message MUST be on a single line.
All examples in the following sections are **pretty-printed** over several lines.
This is just for increased readability of the RFC.
As mentioned, the JSON documents are actually sent over the wire **without** newlines.

#### Frames

Each frame is encoded as a JSON-object with the following schema:

```json
{
  "$id": "https://comit.network/transport-protocol/frame.json",
  "type": "object",
  "definitions": {},
  "$schema": "http://json-schema.org/draft-07/schema#",
  "properties": {
    "type": {
      "$id": "/properties/type",
      "type": "string",
      "title": "The frame type",
      "default": "",
      "examples": [
        "REQUEST"
      ]
    },
    "id": {
      "$id": "/properties/id",
      "type": "integer",
      "title": "The frame id",
      "default": 0,
      "examples": [
        0
      ],
      "minimum" : 0,
      "maximum" : 4294967295
    },
    "payload": {
      "$id": "/properties/payload",
      "type": "object"
    }
  }
}
```

Example:

```json
{
    "type": "REQUEST",
    "id": 10,
    "payload": {}
}
```

The field `payload` holds the data for all the different frames and looks different for every type.
Implementations SHOULD deserialize it into a generic data structure so that they can handle all types of frames.

#### Encoding of the individual frame types

##### Request

For the `REQUEST` frame, the `payload` looks like this:

```json
{
    "type": "...",
    "headers": {},
    "body": {},
}
```

##### Response

The `payload` of a `RESPONSE` frame shares the same encoding as the `REQUEST` frame.
As noted above, responses don't have a `type`.

```json
{
    "headers": {},
    "body": {},
}
```

#### Header

This section defines how headers are encoded in the JSON-based text encoding. 

##### Representation

Let's start off with an example:

```json
"_alpha_ledger" : {
    "value": "Bitcoin",
    "parameters": {
        "network": "mainnet"
    }
}
```

In the above example:
- `alpha_ledger` is the header-key.
- The underscore in the beginning denotes that this header MAY be ignored if not understood.
Respectively, if the key does_not start with an underscore, the header is mandatory and MUST be understood by the node.
- `value` is the header-value (see [Headers - Requirement 2](#headers))
- `parameters` encode the parameters of the header.

##### Default values

Implementations MUST NOT fail if they receive a header without a `parameters` field but rather default to an empty object (which is equivalent to no parameters).

##### Allowed values

The `value` of a header MAY be any valid JSON-value.
This includes `object`s and `list`s.

##### Compact representation

Sometimes, headers only carry one particular value.
For example:

```json
"swap_protocol": {
    "value": "COMIT-RFC-003"
}
```

In cases like these, where there are no parameters, implementations can choose to use the compact representation which looks like this:

```json
"swap_protocol": "COMIT-RFC-003"
```

Implementations MUST be able to process compact representations.
They MUST treat them identical to the version with only a `value` field.

In the compact representation, the `value` of a header MUST NOT be an `object`.

The following is therefore invalid:

```json
"invalid_header": {
    "some_key": "foobar"
}
```

If a header needs an `object` to express its value, you should resort to the default representation to make it unambigous:

```json
"valid_header": {
    "value": {
        "some_key": "foobar"
    }
}
```

#### Naming conventions

- Frame types should use all caps convention.
For example, `REQUEST`.
- Headers should use snake case convention.
For example, `alpha_ledger`.
- `REQUEST` types should use all caps convention as well.
For example: `SWAP`.

### Connection errors / failure cases

The following section describes the behaviour in certain failure scenarios.

#### Malformed response

It can happen that a counterparty sends a malformed response or that the response is not deserializable due to some other cause.
For requests, an `ERROR` frame with type `malformed-frame` can be sent back to the other party.
However, there are no response messages for responses.
In this case, implementations should treat this as a *temporary* failure by logging the incident and ignoring the malformed response.
Implementations should be able to receive further messages on the connection.

## References

- [1] : https://tools.ietf.org/html/rfc3117#section-3.3
