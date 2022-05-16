# Neon Timing Protocol

> This protocol is currently under development. We are looking for feedback.
> Breaking changes to this protocol may still occur causing your clients to no longer be compatible.

The purpose of this protocol is to enable **real time** communication of race data between multiple participating clients.
Participating clients can be a range of devices, such as: timing decoders, scoring displays, lights, and race software.

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
- [ ] Support satellite -> hub -> race computer communication
    - The current issue is that things like `handshake_ack` won't make it back to the originating device
- [ ] Consider converting "race time over" event type back to "race last lap"
- [ ] Finalize handshake and event commands
- [ ] Finish documentation


---


# Communication Protocols


## Serial
- Port Settings
    - Baud Rate: 115200
    - Data Bits: 8
    - Parity: None
    - Stop Bits: 1
- Message Delimiter: Each message must be followed by a newline `\n` character.
- Authentication: When connected over serial authentication is granted automatically.

## Work in progress: Socket.io
- Transport: Websocket only. No upgrades.
- Namespace: neon-timing
- Todo: Auth
- Todo: Why socket.io
- Todo: Emitting events
- Todo: Receiving events


---


# Messages
Messages must be in JSON format.

Messages between clients are done through a set of commands. Some of these commands have additional arguments.

Invalid messages should not cause failure of the connection unless it is a matter of security or stability of the client.

Messages should be as compact as possible. The message character limit is 200 characters.


## Message Format
Each message **must** include the following set of properties to be considered valid.

Commands **may** have additional properties.

- `cmd` [string]: A command from the Neon Timing Protocol.
- `protocol` [string]: For this version of the Neon Timing Protocol this value is **NT1**.
- `time` [integer]: Device time (in milliseconds) when the message was sent.
- `did` [string]: A unique device identifier. This should not change between connections. Max 16 characters.

```json
{"cmd":"example_command","protocol":"NT1","time":6066,"did":"DEMO-1234567890A"}
```


# Protocol Commands

## Handshake
All clients **must** implement the commands `handshake_init` and `handshake_ack`.

The handshake commands serve two purposes:
- During initial connection to determine client compatibility.
- At any time during a connection to synchronize clocks between clients.


### Initial Connection
The `handshake_init` command **must** be sent on initial connection by both clients.
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


### Events Argument
`handshake_init` and `handshake_ack` **must** contain array of `events` supported by the client.

- *: Support for receiving all event types.
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
{"type":"handshake_ack","init_time":1652239294011,"device":"Hub","protocol":"NT1","time":6066,"did":"DEMO-1234567890A"}
```


## Events
Clients **should** not send events to clients that do not support the event types. However, the receiving client
**must** gracefully ignore event types it cannot support.


## Race Events
```json
{"cmd":"event","evt":"race","type":"racer_passed_gate","fast":false,"streak_laps":0,"lap":true,"transponder":"123"}
```
```json
{"cmd":"event","evt":"race","type":"race_staging"}
```
```json
{"cmd":"event","evt":"race","type":"countdown_started"}
```
```json
{"cmd":"event","evt":"race","type":"countdown_end_delay_started"}
```
```json
{"cmd":"event","evt":"race","type":"race_started"}
```
```json
{"cmd":"event","evt":"race","type":"race_time_over"}
```
```json
{"cmd":"event","evt":"race","type":"race_completed"}
```

## Flag Events
```json
{"cmd":"event","evt":"flag","type":"red"}
```
```json
{"cmd":"event","evt":"flag","type":"green"}
```
```json
{"cmd":"event","evt":"flag","type":"white"}
```
```json
{"cmd":"event","evt":"flag","type":"clear"}
```

## Gate Events
```json
{"cmd":"event","evt":"race","type":"transponder_passed_gate","transponder":"1234567890ABCDEF"}
```

