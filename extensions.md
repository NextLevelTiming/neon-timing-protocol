# Protocol Extensions

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---

To avoid future incompatability, extensions may not be included in the final version of protocol. It is best to integrate directly into the base protocol.


# Commands

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
