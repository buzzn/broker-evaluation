# Broker Api
Each call returns the HTTP error code `200` or a HTTP error code with a optional error object: `{"errorName": string, "errorDescription": string}`.
Attention: Each call requires a logged-in user! The session token is sent via the according Authorization header (Exception: `/set-password`).
If the session has expired, each request will result in a `401` and an according error object.
We use POSIX time for all date/time values.

## User login
### Login
`POST /login`
Parameters:
- `user` string The account's username.
- `password` string The account's password.

Returns:
`200` `{"sessionToken": string}` If username exists and password is correct.
    -  *sessionToken* The session token.
`401` `{errorObject}` If username does not exist or password is incorrect.

### Logout
`Get /logout`
Parameters: none

Invalidates the user's session.

Returns: `200` `{}` If the user was logged in and the session was invalidated.

## Hints
### Get all hints
`GET /hints`
Parameters: none

Gets all the hints how to save power available for the user.

Returns: `200` `{"icon": string, "headline": string, "description": string}[]`

### Mark hint read
`POST /hint/mark-read/$id`
Parameters: `id` int The id of the read hint.

Marks the given hint as read by the user.

Returns: `202` If the hint exists and was marked read.
        `404` `{ErrorObject}` If the hint does not exist.

## Challenges
### List available challenges
`GET /challenges`
Parameters: none

Gets all the challenges available for the user.

Returns: `200` `{"id": int, "name": string, "description": string, "total": int, "succeeded": int}[]` or `204` if there is no challenge.
- *id*: Challenge's id
- *name*: Challenge's name
- *description*: Challenge's description
- *total*: Counts how often this challenge has been run by the user.
- *succeeded*: Counts how often this challenge has been succeeded by the user. Always < total.

### Start a challenge
`GET /challenges/start/$id`
Parameters: `id` int The id of the challenge to start

Starts the specified challenge.

Returns:`200` `{}`
    or `404` `{ErrorObject}` If the `id` is not valid.

### Get status of running challenge(s)
`GET /challenges/status`
Parameters: none

Shows the current running challenge.

Returns:`200` `{"id": int, "progress": float, "expiresAt": int, "status": "running" | "failed" | "succeeded"}` | `204` `{}`
- *id*: The id of the running challenge.
- *progress*: A value indicating the distance to the target, usually in kWh
- *expiresAt*: Seconds left until the challenge ends.
- *status*: Indicates the challenge's state, running, failed or succeeded.
- Or `204` `{}` if no challenge is running.

Example one challenge running:`200` `{"id": 1, "progress": 0.8, "expiresAt": 72032, "status": "running"}`

### Quit a challenge.
`POST /challenges/quit`
Parameters: `id` int The id of the challenge to quit.

Quits a challenge. This can be used to abort a running challenge or
acknowledge the failure or success of a ended challenge.
Returns: `200` if the challenge was quitted
	 `404` if no such challenge is running, failed or succeeded recently

## Consumption usage history
### User
`GET /individual-consumption-history?begin=$begin&end=$end&tics=$tics`

Parameters:
- *begin*: `int` Start time of consumption. Default is today at 0:00.
- *end*: `int` End time of consumption. Default is $now.
- *tics*: `str` Time distance between readings with possible values 'raw', 'three_minutes', 'fifteen_minutes', 'one_hour', 'one_day', 'one_week', 'one_month', 'one_year'. Default is 'three_minutes'.

Shows the history of consumption of the given time interval.

Returns: `200` `string => int` A dictionary (JSON object) where each meter reading is mapped to its point in time.
         or `206` `{}` if there is no history.

Example: `200` `{"1574684336": 45322, 1574684346": 45352, 1574684356": 45422, 1574684366": 45522, ...}`

### Group
`GET /group-consumption-history?begin=$begin&end=$end&tics=$tics`

Parameters:
- *begin*: `int` Start time of consumption. Default is today at 0:00.
- *end*: `int` End time of consumption. Default is $now.
- *tics*: `str` Time distance between readings with possible values 'raw', 'three_minutes', 'fifteen_minutes', 'one_hour', 'one_day', 'one_week', 'one_month', 'one_year'. Default is 'three_minutes'. 

Shows the history of consumption of the given time interval.

Returns:
- `200` `{"consumed": string => int, "produced": string => int}` or `206` `{}` if there is no history.
- *consumed*: A dictionary (JSON object) where each meter reading is mapped to its point in time.
- *produced*: A dictionary (JSON object) where each meter reading is mapped to its point in time.

Example: `200 {"consumed": {"1574684336": 45322, 1574684346": 45352, 1574684356": 45422, 1574684366": 45522, ...}, "produced": {"1574684336": 45322, 1574684346": 45352, 1574684356": 45422, 1574684366": 45522, ...}}`

## Disaggregation 
### User
`GET /individual-disaggregation?begin=$begin&end=$end`

Parameters: 
- *begin*: `int` Start time of disaggregation. Default is 48h back in time. 
- *end*: `int` End time of disaggregation. Default is $now.

Shows the power curve disaggregation of the given time interval.

Returns: `200` `{string => {string => int}}` or `206` `{}` if there is no history.

Example: `200` `{"1574101800000": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

### Group 
`GET /group-disaggregation?begin=$begin&end=$end`

Parameters: 
- *begin*: `int` Start time of disaggregation. Default is 48h back in time. 
- *end*: `int` End time of disaggregation. Default is $now. 

Shows the power curve disaggregation of the given time interval.

Returns: `200` `{string => {string => int}}` or `206` `{}` if there is no history. 

Example: `200` `{"1574101800000": {"Waschmaschine-1": 0, "Spülmaschine-1": 0, "Durchlauferhitzer-2": 0, "Durchlauferhitzer-3": 0, "Grundlast-1": 2500000, "Durchlauferhitzer-1": 0}}`

## Profile
### List profile
`GET /profile`
Parameters: none

Gets the user's profile.
Returns: `200` `{"id": int, "name": string, "firstName": string, "nick": string, "flatSize": int, "flatPopulation": int, "groupAddress": string, "mail": string, "avatar": string}`
- *id*: The user's id.
- *name*: The user's name.
- *firstName*: The user's first name.
- *nick*: The user's nick name.
- *groupAddress*: The user's address.
- *mail*: The user's mail address.
- *flatSize*: The user's flat size in m^2
- *flatPopulation*: The number of people living in the flat.
- *avatar*: The user's avatar encoded in base64.

### Update  profile
`PUT /profile`
Parameters:
- *nick*: The user's nick name.
- *flatSize*: The user's flat size in m^2
- *flatPopulation*: The number of people living in the flat.
- *avatar*: The user's avatar encoded in base 64.

Sets the user's profile stats.

Returns:
- `200` If the validation passed or
- `400` `{ErrorObject}` {ErrorObject} If the validation failed

### Set password
`POST /set-password`
Parameters:
- *password*: `string` Password to set
- *token*: `string` User token to identify the correct one. Not confused with the session token.

Sets an initial password. Does not require the user to be logged in.

Returns:
- `200` if the password passes the validation
- `400` `{ErrorObject}` If the  password fails the validation or the token is invalid

### Update password
`PUT /update-password`
Parameter:
- *password*: `string` Password to set

Sets a new password.

Returns:
- `200` if the new password passes the validation
- `400` `{ErrorObject}` If the new passward fails the validation

## Community chart
`GET /hitlist`
Parameters: none

Shows the group's hitlist.

Returns:`200` `{"id": int, "avatar": string, "position": int, "name": string, "value": string}[]`
- *id*: `int` The user's id.
- *avatar*: The user's avatar encoded in base64.
- *icon*: `string` The group icon.
- *position*: `int` The hitlist position of the entry.
- *name*: `string` The user's name of the position.
- *value*: `string` The user's value to be compared. May return an Error


## Live data via websocket
Address: `ws:BASEURL/live`.

The Websocket times out after 2 Minutes.
```javascript
{
    "date": date,
    "groupConsumption": int,
    "groupProduction": int,
    "selfSufficiency": int,
    "usersConsumption": {"id": string, "consumption": int} [],
    "userDevices": {"icon": string, "name": string, "level": int} [],
    "groupDevices": {"icon": string, "name": string, "level": int} []
}
```
Each object represents one reading of the meters.
- *date*: `date` Time of the reading.
- *groupConsumption*: `int` The overall group consumption in mWh.
- *groupProduction*: `int` The overall group production in mWh.
- *selfSufficiency*: `int` Self sufficiency of the user. (Not yet defined)
- *usersConsumption*: `{"id": string, "consumption": int} []` Consumption of each user in the group.
  - *id*: `string` The user's id.
  - *consumption*: `int` User's overall consumption.
- *userDevices*: `{"icon": string, "name": string, "level": int} []` Detected power consuming devices which belong to the user.
  - *icon*: `string` Device icon.
  - *name*: `string` Device name.
  - *level*: `int` Amount of consumed power. (To be defined).
- *groupDevices*: `{"icon": string, "name": string, "level": int} []` Detected power consuming devices which belong to the group.
  - *icon*: `string` Device icon.
  - *name*: `string` Device name.
  - *level*: `int` Amount of consumed power. (To be defined).
