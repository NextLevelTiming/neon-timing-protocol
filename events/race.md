# Race Events

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---


Race events describe events that occur during a live race.


## Race Staging
`Occurs When`: A race has been initialized but is not yet started.

Indicates a closing of the track. Racers should prepare for the start of the race.

```json
{"cmd":"event","evt":"race","type":"race_staging",[...protocol properties]}
```


## Countdown Started
`Occurs When`: A race countdown clock has started.

Indicates to the racers that the race will start imminently.

```json
{"cmd":"event","evt":"race","type":"countdown_started",[...protocol properties]}
```


## Countdown End Delay Started
`Occurs When`: A race countdown clock has started an end delay cycle.

Indicates the start of a random time delay between `countdown_started` and `race_started`.

```json
{"cmd":"event","evt":"race","type":"countdown_end_delay_started",[...protocol properties]}
```


## Race Started
`Occurs When`: The race has started.

Indicates that racers should start racing.

```json
{"cmd":"event","evt":"race","type":"race_started",[...protocol properties]}
```


## Racer Passed Gate
`Occurs When`: The racer has passed a gate.

Indicates the racer has passed a gate, possibly completed a lap, 

- TODO: Rename this or break it into multiple events? `racer_completed_lap` and `racer_passed_gate`
- TODO: This doesn't indicate the gate, should it?

```json
{"cmd":"event","evt":"race","type":"racer_passed_gate","fast":false,"streak_laps":0,"lap":true,"transponder":"123",[...protocol properties]}
```


## Race Time Over
`Occurs When`: A race time limit as been reached.

Indicates the race has a time limit and the racers should complete the race on their next lap.

- TODO: Consider converting "race time over" event type back to "race last lap" (this doesn't make sense at the moment but I wrote it down so I will think about this later)
- TODO: What about staggered starts? The race may not be over on their next lap... Maybe we should not throw this event for staggered races.

```json
{"cmd":"event","evt":"race","type":"race_time_over",[...protocol properties]}
```


## Race Completed
`Occurs When`: A race has been completed.

Indicates that all the racers have completed the race or the race was stopped due to an error.

```json
{"cmd":"event","evt":"race","type":"race_completed",[...protocol properties]}
```
