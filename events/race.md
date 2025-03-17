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

### Properties
- `fast` [boolean]: Racer's fastest lap
- `streak` [boolean]: Indicates if the racer is on a streak
- `valid` [boolean]: Valid gate pass
- `transponder` [string]: The transponder ID of the racer
- `gate` [string]: The id of the gate the racer passed
- `gate_type` [string start|checkpoint|finish]: The type of gate the racer passed

```json
{"cmd":"event","evt":"race","type":"racer_passed_gate","fast":false,"streak":true,"lap":true,"transponder":"123","gate_id":"1","gate_type":"start",[...protocol properties]}
```


## Race Standings
`Occurs When`: An update to the standings has occurred.

The position of the racers must be calculated based on the race type.

### Properties
- `name` [string]: Racer's name
- `laps` [integer]: Number of laps completed
- `fast` [integer]: Fastest lap time (in milliseconds)
- `elapsed` [integer]: Total elapsed time (in milliseconds)
- `status` [string active|dnf|dq]: Racer’s status (active, did not finish, disqualified)
- `id` [string]: Unique identifier for the racer’s standing entry
- `transponder` [string]: Transponder ID, which may change

```json
{"cmd":"event","evt":"race","type":"standings_update","name":"Fred Huffington","laps":1,"fast":5555,"elapsed":20000,"status":"active","id":"123","transponder":"123",[...protocol properties]}
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
