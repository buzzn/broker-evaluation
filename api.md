# Broker Api
Each call returns the HTTP error code `200` or a HTTP error code with a optional error object: `{"errorName": string, "errorDescription": string}`.
Attention: Each call requires a logged-in user! The session token is sent via the according Authorization header (Exception: `/set-password`).
If the session has expired, each request will result in a `401` and an according error object.
We use POSIX time for all date/time values.

# User login
## Login
`POST /login`
Parameters:
- `user` string The account's username.
- `password` string The account's password.
Returns:
`200` `{"sessionToken": string}` If username exists and password is correct.
    -  *sessionToken* The session token.
`401` `{errorObject}` If username does not exist or password is incorrect.

## Logout
`Get /logout`
Parameters: none
Invalidates the user's session.
Returns: `200` `{}` If the user was logged in and the session was invalidated.

## Hints
### Get all hints
`GET /hints`
Parameters: none
Gets all the hints how to save power available for the user.
Returns: `200` {"icon": string, "headline": string, "description": string}[]`

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

## Start a challenge
`GET /challenges/start/$id`
Parameters: `id` int The id of the challenge to start
Starts the specified challenge.
Returns:`200` `{}`
    or `404` `{ErrorObject}` If the `id` is not valid.

## Get status of running challenge(s)
`GET /challenges/status`
Parameters: none
Shows the current running challenge.
Returns:`200` `{"id": int, "progress": float, "timeleft": int}` | `204` `{}`
- *id*: The id of the running challenge.
- *progress*: A value indicating the distance to the target, usually in kWh
- *timeleft*: Seconds left until the challenge ends.
- Or `204` `{}` if no challenge is running.
Example one challenge running:`200` `{"id": 1, "progress": 0.8, "timeleft": 72032}`


## Consumption usage history
### User
`GET /individual-consumption-history/begin/$begin/end/$end/tics/$tics`
Parameters:
- *begin*: `string` Start time of consumption. Default is today at 0:00.
- *end*: `string` End time of consumption. Default is $now.
- *tics*: `int` Interval time in seconds. Default is 60.
Shows the history of consumption of the given time interval.
Returns: `200` [float] An array of values where each one stands for the total power consumed at the time.
         or `204` `{}` if there is no history.
Example: `200` `[2903.0, 2903.1, ..., 1523.0] `

### Group
`GET /group-consumption-history/begin/$begin/end/$end/tics/$tics`
Parameters:
- *begin*:`string` Start time of consumption. Default is today at 0:00.
- *end*:`string` End time of consumption. Default is $now.
- *tics*:`int` Interval time in seconds. Default is 60.
Shows the history of consumption of the given time interval.
Returns:
- `200` `{"consumed": float[], "produced": float[]}` or `204` `{}` if there is no history.
- *consumed*: An array of values where each one stands for the total power consumed at the time.
- *produced*: An array of values where each one stands for the total power produced at the time.
Example: {"consumed": [2903.0, 2903.1, ..., 1523.0], "produced": [1234.5, 1234.0, ..., 1523.9]}

## Profile
### List profile
`GET /profile`
Parameters: none
Gets the user's profile.
Returns: `200` `{"name": string, "firstname": string, "nick": string, "flatSize": int, "flatPopulation": int, "groupAddress": string, "mail": string, "avatar": string }`
- *name*: The user's name.
- *firstname*: The user's name.
- *nick*: The user's nick name.
- *groupaddress*: The user's address.
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
Returns:`200` `{'position': int, 'name': string, 'value': string}[]`
- *position*: `int` The hitlist position of the entry.
- *name*: `string` The user's name of the position.
- *value*: `string` The user's value to be compared.may return an Error


## Live data via websocket
Address: `ws:BASEURL/live`.
The Websocket times out after 2 Minutes.
```javascript
{
    "date": date,
    "userConsumption": int,
    "groupConsumption": int,
    "groupProduction": int,
    "selfSufficiency": int,
    "consumersUser": {"icon": string, "name": string, "level": int}
    "consumersGroup": {"icon": string, "name": string, "level": int}
}
```
