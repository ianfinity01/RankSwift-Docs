If you're using the Roblox module, you shouldn't have to use this. Just head on down to the [Roblox Setup](../Roblox/setup.md) page. For those of you who want to be a bit more hands on with this leaderboard server and make your own requests to it, then this is for you.

## AUTHENTICATION
In order for requests to know what board to route to, and to make sure you have access, you must include your API key in your requests. Just add it as the `x-api-key` field in your headers. You can learn how to get an API Key from the [setup page](./setup.md). 

## TYPES
Below is a list of types that you will encounter in the Endpoints section, for your reference.

- `i64`: Signed 64 bit integer. Any integer from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.
- `f64`: 64 bit floating point number. Realistically any number can work here. Make sure your value isn't NaN when sending the request, or you may encounter issues.
- `usize`: 64 bit unsigned integer. Any integer from 0 to 18,446,744,073,709,551,615. If you are hosting the server on a 32 bit operating system, however, this will be a 32 bit unsigned integer. Assume you are not hosting on a 32 bit operating system.
- `String`: Plain text, wrapped in quotation marks in the JSON.
- `bool`: Either `true` or `false`. 
- `Entry`: A JSON fragment as described below. The key is the id (a user ID), and the points is the value (score). The timestamp is the number of seconds as a float since the UNIX epoch (00:00:00 UTC on January 1st, 1970).
```json
{
    "key": i64,
    "points": f64,
    "timestamp": f64
}
```

If a type is followed by a `?`, then it is optional, and may not be there. If you specify the wrong type, fail to specify a non-optional type, specify extra values, or provide a number value that is out of range of what is allowed for that type, you will get a `400 BadRequest` error.

!!! info "Ranks"
    Ranks start at 1, not 0. This means that if you specify a rank of 0 in a request, it will fail with a `400 BadRequest` error. The entry in the leaderboard with the highest value will have rank 1, then the next highest value will have rank 2, etc.

## ENDPOINTS

!!! info "POST Requests"
    For simplicity's sake, every API Request uses the method `POST`. This was designed to be easy to integrate into roblox, and this shouldn't cause issues for you.

### /update
Updates an entry on the board, adding it if it is not present.

Request:
```json
{
    "id": i64,
    "value": f64
}
```
Response:
```json
{
    "code": i64,
    "message": String
}
```
Requires write permissions on the API key in use.

- code `0`: The entry was already in the board and was successfully updated.
- code `1`: The entry was not already in the board, but was successfully added.
- code `-1`: The entry was not added to the board, either due to it being beyond the entry cap or being below the value cutoff.

### /remove
Removes an entry from the leaderboard entirely.

Request:
```json
{
    "id": i64
}
```
Response:
```json
{
    "code": i64,
    "message": String
}
```
Requires write permissions on the API key in use.

- code `0`: The entry was in the board but was successfully removed.
- code `1`: The entry was already not in the board so nothing happened.

### /get
Gets the entry (timestamp and value) of an entry with a specified id (if that id is present on the board).

Request:
```json
{
    "id": i64
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entry": Entry?
}
```

- code `0`: The entry was found in the board and was successfully retrieved. The entry WILL be present in the response json.
- code `-1`: The entry was not found in the board. The entry will NOT be present in the response json.

!!! info "/get or /info"
    This request is slightly faster than `/info`, but does not return rank. Generally speaking, you will just want to use `/info`, but if you are sure you won't need the rank, use this.

### /info
Gets the entry (timestamp and value) and rank of an entry with a specified id (if that id is present on the board).

Request:
```json
{
    "id": i64
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entry": Entry?,
    "rank": usize?
}
```

- code `0`: The entry was found in the board and was successfully retrieved. The entry and rank WILL be present in the response json.
- code `-1`: The entry was not found in the board. The entry and rank will NOT be present in the response json.

### /board
Gets some information about the board that can be used to perform optimizations or display general information.

Request:
```json
{}
```
Response:
```json
{
    "cap": usize?,
    "cutoff": f64?,
    "min": f64?,
    "size": usize,
    "version": String
}
```

`cap` will be present if the leaderboard has an entry cap.
`cutoff` will be present if the leaderboard has a value cutoff.
`min` will be present if the leaderboard has a size of at least 1. It will only be missing if the board is empty.
`size` will always be present. It is the number of entries in the leaderboard.
`version` is a string that can be used to make sure your code matches up with the server version. The current version is `"1.0.0"`.

!!! info "Making Fewer Requests"
    Using the information from the `cap`, `size`, and `min` parameters can help you reduce the number of unnecessary requests you make to your server. If you are about to make an update request, but the size of the board is at or above the cap, and the value you are going to update is below the minimum, then you can assume that the value will most likely be truncated and not included in the board, so you can avoid sending the request. Furthermore, if the value you are updating is below the `cutoff`, then you can assume it won't be added either. On the Roblox end, the provided module performs these checks for you.

### /atrank
Gets the entry (id, value, and timestamp) at a specified rank if there is an entry on the board with that rank. Will error if a rank of 0 is specified.

Request:
```json
{
    "rank": usize
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entry": Entry?
}
```
If you try to make this request with `"rank": 0`, then you will get a `400 BadRequest` error. Ranks start at 1, with the entry with the highest value having rank 1.

- code `0`: There was an entry with the specified rank, and the entry will be present in the response json.
- code `-1`: There was NO entry with the specified rank, and there will be no entry present in the response json.

### /top
Gets the top (highest value) few entries on the leaderboard.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size number of entries returned, and large `count` values may temporarily stall your server and make requests hang. You are responsible to make sure your `count` is not too large when making the request.

Request:
```json
{
    "count": usize,
    "no_cache": bool?
}
```
Response:
```json
[
    [
        usize,
        Entry
    ],
    ...
]
```
If you do not specify `no_cache`, it will default to `false`. When `no_cache` is `true`, it will always return the most up to date data by avoiding using the cache. To learn more about this cache, go to the [Server Configuration Page](./configuration.md).

The result is just a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `count` entries in the board. The ranks will be in increasing order, meaning the entries' values are decreasing.

### /bottom
Gets the bottom (lowest value) few entries on the leaderboard.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size number of entries returned, and large `count` values may temporarily stall your server and make requests hang. You are responsible to make sure your `count` is not too large when making the request.

Request:
```json
{
    "count": usize,
    "no_cache": bool?
}
```
Response:
```json
[
    [
        usize,
        Entry
    ],
    ...
]
```
If you do not specify `no_cache`, it will default to `false`. When `no_cache` is `true`, it will always return the most up to date data by avoiding using the cache. To learn more about this cache, go to the [Server Configuration Page](./configuration.md).

The result is just a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `count` entries in the board. The ranks will be in decreasing order, meaning the entries' values are increasing.

### /after
Gets entries after (lower value, higher rank) a specified entry on the leaderboard.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size number of entries returned, and large `count` values may temporarily stall your server and make requests hang. You are responsible to make sure your `count` is not too large when making the request.

Request:
```json
{
    "id": i64,
    "count": usize
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entries": [
        [
            usize,
            Entry
        ],
        ...
    ]?
}
```

- code `0`: The entry with the specified id was found, and entries are present.
- code `1`: The entry with the specified id was not on the board, and the entries field in the response is not present.

The entries field, if present, is a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `count` entries after the specified entry in the board. The ranks will be in increasing order meaning the entries' values are decreasing.

### /before
Gets entries before (higher value, lower rank) a specified entry on the leaderboard.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size number of entries returned, and large `count` values may temporarily stall your server and make requests hang. You are responsible to make sure your `count` is not too large when making the request.

Request:
```json
{
    "id": i64,
    "count": usize
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entries": [
        [
            usize,
            Entry
        ],
        ...
    ]?
}
```

- code `0`: The entry with the specified id was found, and entries are present.
- code `1`: The entry with the specified id was not on the board, and the entries field in the response is not present.

The entries field, if present, is a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `count` before the specified entry in the board. The ranks will be in decreasing order meaning the entries' values are increasing.

### /around
Gets entries around (before and after) a specified entry on the leaderboard.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size of this range, and large ranges may temporarily stall your server and make requests hang. You are responsible to make sure your range is not too large when making the request.

Request:
```json
{
    "id": i64,
    "before": usize,
    "after": usize
}
```
Response:
```json
{
    "code": i64,
    "message": String,
    "entries": [
        [
            usize,
            Entry
        ],
        ...
    ]?
}
```

- code `0`: The entry with the specified id was found, and entries are present.
- code `1`: The entry with the specified id was not on the board, and the entries field in the response is not present.

The entries field, if present, is a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `before` entries before the specified entry, or not `after` entries after the specified entry. It will return as many entries as it can. The ranks will be in increasing order meaning the entries' values are decreasing.

### /range
Returns all entries in a range of ranks.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the size of this range, and large ranges may temporarily stall your server and make requests hang. You are responsible to make sure your range is not too large when making the request.

Request:
```json
{
    "start": usize,
    "end": usize
}
```
Response:
```json
[
    [
        usize,
        Entry
    ],
    ...
]
```
The result is just a list of rank, entry pairs. That is to say, the `usize` in the response is the rank. Note that the length of the returned list may be less than your specified count if there are not `count` entries in the range. The ranks will be in increasing order, meaning the entries' values are decreasing.

If `start` or `end` are 0, you will get a `400 BadRequest` error. The `start` and `end` are both inclusive. At most, this will return `end - start + 1` entries. If `start > end` then the returned list will be empty. The list will also be empty if the starting rank is greater than the number of entries in the board.

### /batch
Useful for reducing the number of individual HTTP requests you make to the server. May also marginally reduce network strain by only sending one request header for all the requests in the batch instead of there being a header for each individual request.

!!! warning "No Safeguard"
    There is NO SAFEGUARD on the number of requests in your batch, and large ranges may temporarily stall your server and make requests hang. You are responsible to make sure your batch is not too large when making the request.

Request:
```json
[
    {
        "req_type": String,
        "payload": String
    },
    ...
]
```
Response:
```json
[
    String,
    ...
]
```

`req_type` must be one of the following:

- `"Update"`
- `"Remove"`
- `"Get"`
- `"Info"`
- `"Board"`
- `"AtRank"`
- `"Top"`
- `"Bottom"`
- `"After"`
- `"Before"`
- `"Around"`
- `"Range"`

Any other value will result in a `400 BadRequest` error. Note that you cannot have batch requests inside batch requests.

`payload` should be the stringified JSON that you would have included in the corresponding request. All responses will be the JSON stringified responses of corresponding requests. The first entry in the batch will correspond to the first string in the response, and so on.

!!! warning "Requests Execute in Order"
    All requests in a batch will execute in order. That means if you put an `Info` request before a `Update` request, the information in the `Info` request's corresponding response will be from BEFORE the update requests was carried out. Similarly, if you put the `Update` request before the `Info` request, the `Info` request will give back information from AFTER the update carries out. There is NO GUARANTEE that no requests from other sources may happen in between two consecutive requests in a batch.

!!! failure "Bad Requests"
    If any of the payloads are invalid and return a `400 BadRequest` error, the entire batch request will return a `400 BadRequest` error, and you will not get any info back. It is important to note that any requests before the maliformed request WILL STILL EXECUTE, but not any of the ones after it.