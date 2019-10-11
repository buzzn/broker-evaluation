# Broker Api
Each call returns the HTTP error code `200` or an error HTTP with an error object: `{"errorName": string, "errorDescription": string}`.  
Attention: Each call requires a logged-in user! (Exception: `/set-password`.) We use the JSON/JavaScript datetime representation as specified by the W3C. Example: '2019-10-08T10:44:57+01:00'

## Hints
`GET /hints`  
Parameters: none  
Gets all the hints how to save power available for the user.  
Returns:`{"icon": string, "headline": string, "description": string}[]`

## Challenges
### List available challenges
`GET /challenges`  
Parameters: none  
Gets all the challenges available for the user.  
Returns:`{"id": int, "name": string, "description": string, "total": int, "succeeded": int}[]`
- *id*: Challenge's id
- *name*: Challenge's name
- *description*: Challenge's description
- *total*: Counts how often this challenge has been run by the user.
- *succeeded*: Counts how often this challenge has been succeeded by the user. Always < total.

## Start a challenge
`GET /challenges/start/id/$id`  
Parameters: `id` int The id of the challenge to start  
Starts the specified challenge.  
Returns:`{}`

## Get status of running challenge(s)
`GET /challenges/status`  
Parameters: none  
Shows the current running challenge.  
Returns:`{"id": int, "progress": float, "timeleft": int}` | `[]`
- *id*: The id of the running challenge.
- *progress*: A value indicating the distance to the target, usually in kWh
- *timeleft*: Seconds left until the challenge ends.
- Or `[]` if no challenge is running.
Example one challenge running:`{"id": 1, "progress": 0.8, "timeleft": 72032}`

## Consumption usage history
### User
`GET /individual-consumption-history/begin/$begin/end/$end/tics/$tics`  
Parameters:
- *begin*: `string` Start time of consumption. Default is $now-24h.
- *end*: `string` End time of consumption. Default is $now. 
- *tics*: `int` Interval time in seconds. Default is 60.
Shows the history of consumption of the given time interval.  
Returns: [float] An array of values where each one stands for the total power consumed at the time.  
Example: [2903.0, 2903.1, ..., 1523.0] 

### Group
`GET /group-consumption-history/begin/$begin/end/$end/tics/$tics`  
Parameters:
- *begin*:`string` Start time of consumption. Default is $now-24h.
- *end*:`string` End time of consumption. Default is $now.
- *tics*:`int` Interval time in seconds. Default is 60.
Shows the history of consumption of the given time interval.  
Returns:
- `{"consumed": float[], "produced": float[]}`
- *consumed*: An array of values where each one stands for the total power consumed at the time.
- *produced*: An array of values where each one stands for the total power produced at the time.
Example: {"consumed": [2903.0, 2903.1, ..., 1523.0], "produced": [1234.5, 1234.0, ..., 1523.9]} 

## Profile
### List profile
`GET /profile`  
Parameters: none  
Gets the user's profile.  
Returns: `{"name": string, "flatSize": int, "flatPopulation": int}`
- *name*: The user's name.
- *flatSize*: The user's flat size in m^2 
- *flatPopulation*: The number of people living in the flat.

### Update  profile
`POST /profile`  
Parameters:
- *name*: The user's name.
- *flatSize*: The user's flat size in m^2
- *flatPopulation*: The number of people living in the flat.
Sets the user's profile stats.  

### Set password
`POST /set-password`  
Parameters:
- *password*: `string` Password to set
- *token*: `string` User token to identify the correct one.
Sets an initial password. Does not require the user to be logged in.

### Update password
`POST /update-password`  
Parameter:
- *password*: `string` Password to set
Sets a new password.

## Community chart
`GET /hitlist`  
Parameters: none  
Shows the group's hitlist.  
Returns:`{'position': int, 'name': string, 'value': string}[]`
- *position*: `int` The hitlist position of the entry.
- *name*: `string` The user's name of the position.
- *value*: `string` The user's value to be compared.may return an Error


## Live data via websocket
```javascript
{
    'date': date,
    'userConsumption': int,
    'groupConsumption': int,
    'groupProduction': int,
    'selfSufficiency': int,
    'consumersUser': {'icon': string, 'name': string, 'level': int}[]
    'consumersGroup': {'icon': string, 'name': string, 'level': int}[]
}
```
