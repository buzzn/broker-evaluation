# Broker API
Each call returns the HTTP code `200` or an HTTP error code with an optional error object: `{"errorName": string, "errorDescription": string}`.
Attention: Most calls require a logged-in user! The session token is sent via the according authorization header (exceptions: `/set-password`, `/community-global-challenge`).
If the session has expired, each request will result in a `401` error code and an according error object.
We use POSIX time for all date/time values.

## User login
### Login
`POST /login`

Parameters:
- `user` (string): The account's mail address.
- `password` (string): The account's password.

Returns:

`200 {"sessionToken": string}` if username exists and password is correct or `401 {errorObject}` if username does not exist or password is incorrect.
- `sessionToken` (string): The session token.

### Logout
`GET /logout`

Invalidates the user's session.

Parameters: none

Returns: 

`200 {}` if the user was logged in and the session was invalidated.

## Hints
### Get all hints
`GET /hints`

Gets all the hints how to save power available for the user.

Parameters: none

Returns: 

`200 {"icon": string, "headline": string, "description": string}[]`

### Mark hint as read
`POST /hint/mark-read/$id`

Marks the given hint as read by the user.

Parameters: 
- `id` (int): The id of the read hint.

Returns:
 
`202` if the hint exists and was marked read or `404 {ErrorObject}` if the hint does not exist.

## Challenges
### List available challenges
`GET /challenges`

Gets all the challenges available for the user.

Parameters: none

Returns: `200 {"id": int, "name": string, "description": string, "total": int, "succeeded": int}[]` or `204` if there is no challenge.
- *id*: The challenge's id.
- *name*: The challenge's name.
- *description*: The challenge's description.
- *total*: Counts how often this challenge has been run by the user.
- *succeeded*: Counts how often this challenge has been completed successfully by the user. Always < total.

### Start a challenge
`GET /challenges/start/$id`

Starts the specified challenge.

Parameters: 
- `id` (int): The id of the challenge to start.

Returns: `200 {}` or `404 {ErrorObject}` if the `id` is not valid.

### Get status of running challenge(s)
`GET /challenges/status`

Shows the current running challenge.

Parameters: none

Returns: `200 {"id": int, "progress": float, "expiresAt": int, "status": "running" | "failed" | "succeeded"}` | `204 {}` or `204 {}` if no challenge is running.
- *id*: The id of the running challenge.
- *progress*: A value indicating the distance to the target, usually in kWh.
- *expiresAt*: Seconds left until the challenge ends.
- *status*: Indicates the challenge's state, i.e. running, failed or succeeded.

Example with one challenge running: `200 {"id": 1, "progress": 0.8, "expiresAt": 72032, "status": "running"}`

### Quit a challenge
`POST /challenges/quit`

Quits a challenge. This can be used to abort a running challenge or
acknowledge the failure or success of an ended challenge.

Parameters: 
- `id` (int): The id of the challenge to quit.

Returns: `200` if the challenge was quit, `404` if no such challenge is running, has failed or succeeded recently.

## Consumption history
### User
`GET /individual-consumption-history?begin=$begin`

Shows the history of power consumption from the given starting point.

Parameters:
- `begin` (int): Start time of consumption as unix timestamp. Default is today at 00:00:00 am.

Returns: `200 {"energy": string => int, "power": string => int}` or `206 {}` if there is no history. 
- *energy*: A dictionary (JSON object) with the first and the latest meter reading (μWh/10) of the day mapped to their timestamps.
- *power*: A dictionary (JSON object) with average power consumptions (mW) of 15 minute timeframes mapped to their timestamps.

Example: `200 {'energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000},
'power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0}}`

### Group
`GET /group-consumption-history`

Shows the history of power consumption.

Parameters: none

Returns: `200 
{ 
    "consumed_energy": string => int, 
    "consumed_power": string => int,
    "group_users": list,
    "produced_first_meter_energy": string => int, 
    "produced_first_meter_power": string => int,
    "produced_second_meter_energy": string => int,
    "produced_second_meter_power": string => int
}` 
or `206 {}` if there is no history.
- *consumed_energy*: A dictionary (JSON object) with the first and the latest meter reading (μWh/10) of the day mapped to their timestamps.
- *consumed_power*: A dictionary (JSON object) with average power consumptions (mW) of 15 minute timeframes mapped to their timestamps.
- *group_users*: A dictionary (JSON objects) with the first and the latest meter reading (μWh/10) of the day and average power consumption values (mW) of 15 minutes timeframes of each group user mapped to their timestamps.
- *produced_first_meter_energy*: A dictionary (JSON object) with the first and the latest meter reading (μWh/10) of the day mapped to their timestamps.
- *produced_first_meter_power*: A dictionary (JSON object) with average power productions (mW) of 15 minute timeframes mapped to their timestamps.
- *produced_second_meter_energy*: A dictionary (JSON object) with the first and the latest meter reading (μWh/10) of the day mapped to their timestamps.
- *produced_second_meter_power*: A dictionary (JSON object) with average power productions (mW) of 15 minute timeframes mapped to their timestamps.

Example: 
```javascript
200  {
        'consumed_energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000},
        'consumed_power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0},
        'group_users': {'1': {'energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000}, 
                              'power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0}},
                        '2': {'energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000},
                              'power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0}}
                        ...},
        'produced_first_meter_energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000},
        'produced_first_meter_power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0},
        'produced_second_meter_energy': {'2020-01-15 00:00:04': 2180256872214000, '2020-01-15 00:47:10': 2180256872214000},
        'produced_second_meter_power': {'2020-01-15 00:15:00': 224550.0, '2020-01-15 00:30:00': 232000.0, '2020-01-15 00:45:00': 227630.0}
}
```

## Disaggregation 
### User
`GET /individual-disaggregation?begin=$begin`

Shows the power curve disaggregation from the given starting point.

Parameters: 
- `begin` (int): Start time of disaggregation as unix timestamp. Default is two days ago at 00:00:00 am.

Returns: `200 {string => {string => int}}` or `206 {}` if there is no history.

Example: `200` `{"2020-01-15 10:01:10": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

### Group 
`GET /group-disaggregation?begin=$begin`

Shows the power curve disaggregation from the given starting point.

Parameters: 
- `begin` (int): Start time of disaggregation as unix timestamp. Default is two days ago at 00:00:00 am.

Returns: `200 {string => {string => int}}` or `206 {}` if there is no history. 

Example: `200 {"2020-01-15 10:01:10": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

## Profile
### List profile
`GET /profile`

Gets the user's profile.

Parameters: none

Returns: `200 {"id": int, "name": string, "firstName": string, "nick": string, "flatSize": float, "inhabitants": int, "groupAddress": string, "mail": string, "avatar": string, "registration_date": string, "baseline_state": string}`
- *id*: The user's id.
- *name*: The user's name.
- *firstName*: The user's first name.
- *nick*: The user's nick name.
- *groupAddress*: The user's address.
- *mail*: The user's mail address.
- *flatSize*: The user's flat size in m^2.
- *inhabitants*: The number of people living in the flat.
- *avatar*: The user's avatar encoded in base64.
- *meterId*: The id of the user's meter.
- *registration_date*: The user's registration date (year-month-day hour:minute:second.microseconds)
- *baseline_state*: The user's baseline sate ("READY", "WAITING_FOR_DATA", "NO_READINGS_AVAILABLE")

### Update  profile
`PUT /profile`

Sets the user's profile stats.

Parameters:
- `flatSize` (optional): The user's flat size in m^2.
- `inhabitants` (optional): The number of people living in the flat.
- `nick` (optional): The user's nick name.
- `avatar` (optional): The user's avatar encoded in base64.

Returns: `200` if the validation passed or `400 {ErrorObject}` if the validation failed.

### Set password
`POST /set-password`

Sets an initial password. Does not require the user to be logged in.

Parameters:
- `password` (string): Password to set.
- `token` (string): User token to identify the correct one. Not confused with the session token.

Returns:
`200` if the password passes the validation or `400 {ErrorObject}` if the  password fails the validation or the token is invalid.

### Update password
`PUT /update-password`

Sets a new password.

Parameter:
- `password` (string): Password to set.

Returns: `200` if the new password passes the validation or `400 {ErrorObject}` if the new password fails the validation.

## Group profile pictures
`GET /assets/group-profile-pictures`

Shows the profile pictures of the user's group.

Parameters: none

Returns: `200 {"id": int, "avatar": string}[]` or `400 {ErrorObject}`/`206 {ErrorObject}` if group member query/avatar query fails.
- `id`: The user's id.
- `avatar`: The user's avatar encoded in base64.

## Global challenge
### Individual global challenge
`GET /individual-global-challenge`

Shows the individual saving prognosis for today in µWh/10 and the individual baseline in µWh/10.

Parameters: none

Returns: `200 {"saving": {string => float}, "baseline": int}` or `206 {}` if there is no history.

Example: `200 {"saving": {"2020-01-15 10:01:10": 3148577026610.7812}, "baseline": 53011346257574}`

### Community global challenge
`GET /community-global-challenge`

Shows the community saving prognosis for today in µWh/10.

Parameters: none

Returns: `200 {string => float}` or `206 {}` if there is no history.

Example: `200 {"2020-02-13 16:20:21": 85184267259376.5}`

## Per capita consumption
`GET /per-capita-consumption`

Shows the annualized moving average of the user's per capita consumption for today in kWh.

Parameters: none

Returns: `200 {string => int}` or `206 {}` if is no value.

Example: `200 {'2020-03-10 17:08:24': 396932}`

## Live data via websocket
`GET /live`

Parameters:
- `meter_id` (string): The user's meter id.

```javascript
{
    "date": date,
    "group_production": int,
    "group_users": [
	{
	    "id": string, 
	    "meter_id": string,
	    "consumption": int, 
	    "power": int
	},
	...
    ]
}
```
Each object represents one reading of the database.
- *date*: Time of the reading.
- *group_production*: The overall group production in µWh/10.
- *group_users*: ID, meter ID, consumption and power (Leistung) of each user in the group. 
*consumption* is given in µWh/10, *power* (Leistung) in µW. 
