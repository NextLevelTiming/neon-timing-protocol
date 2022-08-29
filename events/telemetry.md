# Telemetry Events

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---


Telemetry events can be used to monitor changes of various meta data. Such as track temperature and humidity.


## Track Telemetry
`Occurs When`: Track telemetry data is updated.


## Additional Properties
- `temperature` [float]: The temperature measured in celcius
- `humidity` [float]: The humidity measured in percent

```json
{"cmd":"event","evt":"telemetry","type":"track","temperature":50.5,"humidity":20.5,[...protocol properties]}
```
