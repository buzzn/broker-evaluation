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

## Consumption usage history
### User
`GET /individual-consumption-history?begin=$begin`

Shows the history of power consumption from the given starting point.

Parameters:
- `begin` (float): Start time of consumption as unix timestamp. Default is today at 0:00.

Returns: `200 {'energy': string => int, 'power': string => int}` or `206 {}` if there is no history. 
- *power*: A dictionary (JSON object) where each power consumption (mW) is mapped to its point in time.
- *energy*: A dictionary (JSON object) where each meter reading (μWh) is mapped to its point in time.

Example: `200 {'energy': {'2020-01-15 10:00:04': 2180256872214000, '2020-01-15 10:01:10': 2180256872214000, ...},
'power': {'2020-01-15 10:00:04': 0, '2020-01-15 10:01:10': 0, ...}}`

### Group
`GET /group-consumption-history?begin=$begin`

Shows the history of power consumption from the given starting point.

Parameters:
- `begin` (float): Start time of consumption as unix timestamp. Default is today at 0:00.

Returns: `200 {"consumed": string => int, "produced_first_meter": string => int, "produced_second_meter": string => int}` or `206` `{}` if there is no history.
- *consumed*: A dictionary (JSON object) where each power consumption (mW) is mapped to its point in time.
- *produced_first_meter*: A dictionary (JSON object) where each power production (mW) is mapped to its point in time.
- *produced_second_meter*: A dictionary (JSON object) where each power production (mW) is mapped to its point in time.

Example: `200 {"consumed": {"2020-01-15 10:01:04": 45322, "2020-01-15 10:01:10": 45352, "2020-01-15 10:01:16": 45422, "2020-01-15 10:01:20": 45522, ...}, 
"produced_first_meter": {"2020-01-15 10:01:04": 45322, "2020-01-15 10:01:10": 45352, "2020-01-15 10:01:16": 45422, "2020-01-15 10:01:20": 45522, ...}, 
"produced_second_meter": {"2020-01-15 10:01:04": 45322, "2020-01-15 10:01:10": 45352, "2020-01-15 10:01:16": 45422, "2020-01-15 10:01:20": 45522, ...}}`

## Disaggregation 
### User
`GET /individual-disaggregation?begin=$begin`

Shows the power curve disaggregation from the given starting point.

Parameters: 
- `begin` (float): Start time of disaggregation as unix timestamp. Default is 48h back in time. 

Returns: `200 {string => {string => int}}` or `206 {}` if there is no history.

Example: `200` `{"2020-01-15 10:01:10": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

### Group 
`GET /group-disaggregation?begin=$begin`

Shows the power curve disaggregation from the given starting point.

Parameters: 
- `begin` (float): Start time of disaggregation as unix timestamp. Default is 48h back in time. 

Returns: `200 {string => {string => int}}` or `206 {}` if there is no history. 

Example: `200 {"2020-01-15 10:01:10": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

## Profile
### List profile
`GET /profile`

Gets the user's profile.

Parameters: none

Returns: `200 {"id": int, "name": string, "firstName": string, "nick": string, "flatSize": float, "inhabitants": int, "groupAddress": string, "mail": string, "avatar": string}`
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

Shows the individual saving prognosis for today in µWh and the individual baseline value in µWh.

Parameters: none

Returns: `200 {"saving": {string => float}, "baseline": {string => int}}` or `206 {}` if there is no history.

Example: `200 {"saving": {"2020-01-15 10:01:10": 3148577026610.7812}, "baseline": {"2020-02-21 09:40:04": 53011346257574}}`

### Community global challenge
`GET /community-global-challenge`

Shows the community saving prognosis for today in µWh.

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
    "group_consumption": int,
    "group_production": int,
    "group_users": {
	"id": string, 
	"meter_id": string,
	"consumption": int, 
	"power": int,
	"self_sufficiency": float} [],
}
```
Each object represents one reading of the database.
- *date*: Time of the reading.
- *group_consumption*: The overall group consumption in µWh.
- *group_production*: The overall group production in µWh.
- *group_users*: ID, meter ID, consumption, power (Leistung) and self-sufficiency of each user in the group. 
*consumption* is given in µWh, *power* (Leistung) in µW. 
*self_sufficiency* returns a value between 0 and 1 where smaller values are considered to be better.
