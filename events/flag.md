# Flag Events

**This protocol is currently under development. We are looking for feedback.**

**_Warning: Breaking changes to this protocol may still occur causing your clients to no longer be compatible._**


---


**Todo: Flesh out flag events**


## Red Flag
```json
{"cmd":"event","evt":"flag","type":"red",[...protocol properties]}
```


## Green Flag
```json
{"cmd":"event","evt":"flag","type":"green","protocol":"NT1","time":6066,"did":"DEMO-1234567890A",[...protocol properties]}
```


## White Flag
```json
{"cmd":"event","evt":"flag","type":"white","protocol":"NT1","time":6066,"did":"DEMO-1234567890A",[...protocol properties]}
```


## Clear
```json
{"cmd":"event","evt":"flag","type":"clear",[...protocol properties]}
```
