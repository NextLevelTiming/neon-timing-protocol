# Neon Timing Protocol

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---


The purpose of this protocol is to enable **real time** communication of race data between multiple participating clients.
Participating clients can be a range of devices, such as: timing decoders, scoring displays, lights, race software, and telemetry sensors.

Supporting clients may enable enhanced broadcasting and connectivity features. This may include enabling compatibility
for hardware that is not compatible with the Neon Timing Protocol.

Clients **may** support one or more connected clients. If more than one connected client is supported, the host **must**
rebroadcast events received.

- [Todo](#todo)
- [Communication Protocols](#communication-protocols)
- [Messages](#messages)
- [Protocol Commands](#protocol-commands)
  - [Handshake](#handshake)
  - [Events](#events)


---


# Todo

> There are a few things left to finish before we recommend using the Neon Timing Protocol for use.

- [ ] Get feedback from community
- [ ] Communication protocol examples and projects implementing the protocol
  - [ ] Serial
  - [ ] Socket.io
- [ ] Add support for hub communication: client -> hub -> (host) race computer
    - The current issue is that things like `handshake_ack` won't make it back to the originating device
- [ ] Finalize handshake and event commands
- [ ] Ensure the 200 character message limit is possible with the defined protocol. Increase as needed.
- [ ] Host vs client? Is there P2P communication or is it all client to host?
- [ ] Pick a date to solidify version 1 of the protocol


---


# Communication Protocols
The Neon Timing Protocol is primarily a message protocol. Communication protocols are used to deliver messages between
clients. A client **may** support more than one communication protocol.

It is recommended that clients follow the communication protocol guidelines. This is not strictly necessarry.

Newer, faster, easier communication protocols are being developed every day. We may adopt and make recommendations for
implementation of these communication protocols. Ideally message data being sent between clients will remain unchanged
when adding support for newer communication protocols.

## Serial
Serial is a very common protocol used when developing hardware. It is easy to implment and most computers are capable
of sending and receiving it through the use of USB COM ports. It also has low latency as there is generally a wire
between the two clients.


- Example client: TODO
- Port Settings
    - Baud Rate: 115200
    - Data Bits: 8
    - Parity: None
    - Stop Bits: 1
- Message Delimiter: Each message must be followed by a newline `\n` character.
- Authentication: When connected over serial authentication is granted automatically.

## WebSocket: Socket.io
[Socket.io](https://socket.io/) is a less common protocol. It sits on top of a more common WebSocket protocol but has a
few extra ease of use features. WebSockets are incredibly powerful. They can be used to communicate over a network,
including the internet. Latency is very dependent on the network connectivity. The Neon Timing Protocol attempts to
reduce timing errors induced by latency.

- Example client: TODO
- Transport: Websocket only. No upgrades.
- Namespace: neon-timing
- Authentication:
  - A `token` query string parameter can be sent when connecting for authentication
  - /neon-timing?token=abc-123
- Emitting events: socket.emit('client_event', message);
- Receiving events: socket.on('host_event', message => {});


---


# Messages
Messages must be in [JSON](https://www.json.org/) format. This increases readablity and defines concrete structure for the messages.
The trade-offs made for processing JSON were not made lightly. We feel there are enough low powered devices that are
capable of handling this format effeciently.

Messages between clients are done through a set of commands. Some of these commands may have additional properties.

Invalid messages should not cause failure of the connection unless it is a matter of security or stability of the client.

Messages **should** be as compact as possible. The message character limit is 200 characters. Limiting message length
ensures performance is maintained and allows for low compute clients.


## Message Format
Each message **must** include the at minimum the following set of properties to be considered valid.
Commands **may** include additional properties beyond this minimum set.

- `cmd` [string]: A command defined in the Neon Timing Protocol.
- `protocol` [string]: For this version of the Neon Timing Protocol this value is **NT1**.
- `time` [integer]: Device time (in milliseconds) when the message was sent.
- `did` [string]: A unique device identifier. This should not change between connections. Max 16 characters.

```json
{"cmd":"example_command","protocol":"NT1","time":6066,"did":"DEMO-1234567890A"}
```

Further in this documentation you will see these referenced as `protocol properties`.
```json
{"cmd":"example_command",[...protocol properties]}
```


# Protocol Commands
Commands are messages that **may** cause the receiving client to execute actions.

Commands can also convey that an event happened. Events are not expected to ellicit any particular response from clients.

Most commands are optional to implement. The `handshake` commands **must** be implmented.

## Command Extensions
Clients are not limited to the list of commands provided by the Neon Timing protocol. If a client implements a custom
command a namespace should be used in the `cmd` property value to ensure future Neon Timing protocol commands do not conflict.

These commands **may** define their own properties but **must** include the required message properties.

Example custom command with `x_` as a namespace:
```json
{"cmd":"x_add_points","protocol":"NT1","time":6066,"did":"DEMO-1234567890A","x_points":3}
```

Clients are also allowed to add additional properties to commands. These addiontal properties should use a namespace.
Additional properties **should** be safely ignored by clients not using them.

Special care should be taken not to alter the original intent of the command, but instead add functionality in such a
way the original purpose is not influenced.

Example command with a custom property of `x_animated`.
```json
{"cmd":"event","evt":"flag","type":"clear","protocol":"NT1","time":6066,"did":"DEMO-1234567890A","x_animated":true}
```

## Handshake
All clients **must** implement the commands `handshake_init` and `handshake_ack`.

The handshake commands serve two purposes:
- During initial connection to determine client compatibility.
- At any time during a connection to synchronize clocks between clients.


### Initial Connection
The `handshake_init` command **must** be sent on initial connection by the hosting client and **may** be sent by the client.
After receiving the `handshake_init` the client **should** respond with `handshake_ack` within one-second or be subject
to connection termination.

Once a client has received a `handshake_ack` it can begin sending and receiving other Neon Timing Protocol commands.


### Handshaking to Synchronize Clocks
The handshake can be used to fetch a clients time and understand latency in the connection. It is not
intended to update or reset the time of clients. While events are intended to be sent and received in real-time it
**is not guaranteed**. Clients processing event messages that require accurate timing, such as transponder and race data,
should synchronize time with the client that is emitting these events.

The suggested pattern for syncing clocks is
- Send the `handshake_init` message and record the `time` as `local_init_time`
- Wait for the `handshake_ack` message response
- Verify the `init_time` matches the previously recorded `local_init_time`
- Get the `time` from the `handshake_ack` response and record it as `remote_ack_time`
- Process time sensitive messages by converting message time to local time
    - local_init_time + incoming_message.time - remote_ack_time
    - Additionally compensate for latency if required.

The Neon Timing Protocol assumes that clients do not update their clock at any point during the connection. You can at any
point initiate a handshake to synchronize time. I suggest synchronizing time during the initial handshake and
at the start of each race.


### Events Property
`handshake_init` and `handshake_ack` **must** contain array of `events` supported by the client.

- *: Support for receiving all event groups.
- race: Support for receiving race events.
- flag: Support for receiving flag events.
- gate: Support for receiving gate events.

Events are described more in the `event` command documentation.


### Handshake Initialization
Directly after connecting to another Neon Timing Protocol client, both clients must send a `handshake_init`.

- `device` [string]: Device name or description
- `events` [array of strings]: A list of events supported by the client.

```json
{"cmd":"handshake_init","device":"Satellite","protocol":"NT1","time":6066,"did":"DEMO-1234567890A"}
```


### Handshake Acknowledgement
Directly after receiving a `handshake_init` command a client **must** respond with `handshake_ack`.

The `handshake_ack` command **must** only be used in response to a `handshake_init` command.

- `device` [string]: Device name or description.
- `init_time` [integer]: The `time` from the `handshake_init` command being responded to.
    - Used by the sender to validate which `handshake_init` is being responded to.
    - Used by the sender to synchronize the clocks.

```json
{"cmd":"handshake_ack","init_time":1652239294011,"device":"Hub","protocol":"NT1","time":6066,"did":"DEMO-1234567890A"}
```


## Event Command
Clients **should** not send events to clients that do not support the event groups specified during the handshake. However, the receiving client **must** gracefully ignore events it cannot support.

Clients processing events **must** must insure both `evt` and `type` properties match the associated event. `type` is not guarenteed to be unique between all event groups.

- `evt` [string]: The event group.
- `type` [string]: The event type.

```json
{"cmd":"event","evt":"flag","type":"red",[...protocol properties]}
```


### Event Groups
- [Race Events](/events/race.md)
- [Flag Events](/events/flag.md)
- [Gate Events](/events/gate.md)


## Telemetry
**Todo**: Telemetry data...

```json
{"cmd":"telemetry","type":["temperature","humidity"],"temperature":50.5,"humidity":20.5,"protocol":"NT1","time":2401,"did":"2509507082"}
```
