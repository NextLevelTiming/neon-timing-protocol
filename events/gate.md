# Gate Events

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---


Gates, also known as sectors or checkpoints, serve to mark areas of the track such as the start and finish.


## Transponder Passed Gate
`Occurs When`: A transponder has passed the gate.

Indicates that a transponder has passed a gates boundry.

Reasonable internal device rejection should occur before broadcasting `transponder_passed_gate`. Do not depend on the receiving clients to debounce or accumalate signal hits to generate a valid event. Future events **may** cover this scenario.

## Additional Properties
- `transponder` [string|reqired|min:1|max:16]: A transponder identifier

```json
{"cmd":"event","evt":"gate","type":"transponder_passed_gate","transponder":"1234567890ABCDEF",[...protocol properties]}
```
