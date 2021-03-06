# Identify v1.0.0

The identify protocol is used to exchange basic information with other peers
in the network, including addresses, public keys, and capabilities.

There are two variations of the identify protocol, `identify` and `identify/push`.

### `identify`

The `identify` protocol has the protocol id `/ipfs/id/1.0.0`, and it is used
to query remote peers for their information.

The protocol works by opening a stream to the remote peer you want to query, using
`/ipfs/id/1.0.0` as the protocol id string. The peer being identified responds by returning
an `Identify` message and closes the stream.

### `identify/push`

The `identify/push` protocol has the protocol id `/ipfs/id/push/1.0.0`, and it is used
to inform known peers about changes that occur at runtime.

When a peer's basic information changes, for example, because they've obtained a new
public listen address, they can use `identify/push` to inform others about the new
information.

The push variant works by opening a stream to each remote peer you want to update, using
`/ipfs/id/push/1.0.0` as the protocol id string. When the remote peer accepts the stream,
the local peer will send an `Identify` message and close the stream.

Upon recieving the pushed `Identify` message, the remote peer should update their local
metadata repository with the information from the message. Note that missing fields
should be ignored, as peers may choose to send partial updates containing only the fields
whose values have changed.

## The Identify Message

```protobuf
message Identify {
  optional string protocolVersion = 5;
  optional string agentVersion = 6;
  optional bytes publicKey = 1;
  repeated bytes listenAddrs = 2;
  optional bytes observedAddr = 4;
  repeated string protocols = 3;
}
```

### protocolVersion

The protocol version identifies the family of protocols used by the peer.
The current protocol version is `ipfs/0.1.0`; if the protocol major or minor
version does not match the protocol used by the initiating peer, then the connection
is considered unusable and the peer must close the connection.

### agentVersion

This is a free-form string, identifying the implementation of the peer.
The usual format is `agent-name/version`, where `agent-name` is
the name of the program or library and `version` is its semantic version.

### publicKey

This is the public key of the peer, marshalled in binary form as specicfied
in [peer-ids](../peer-ids).


### listenAddrs

These are the addresses on which the peer is listening as multi-addresses.

### observedAddr

This is the connection source address of the stream initiating peer as observed by the peer
being identified; it is a multi-address. The initiator can use this address to infer
the existence of a NAT and its public address.

For example, in the case of a TCP/IP transport the observed addresses will be of the form
`/ip4/x.x.x.x/tcp/xx`. In the case of a circuit relay connection, the observed address will
be of the form `/p2p/QmRelay/p2p-circuit`. In the case of onion transport, there is no
observable source address.

### protocols

This is a list of protocols supported by the peer.
