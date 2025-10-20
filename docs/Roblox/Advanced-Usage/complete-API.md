# Complete API

## Events

Events can be connected to with `:Connect(callback)`, `:Once(callback)`, etc. These methods return connections that you can disconnect with `connection:Disconnect()`.

### board_updated

`board.board_updated`

> Fires when a full update is made to the board. You can use this in conjunction with `board:TimeUntilRefresh()` to make it so that an update timer on clients updates. You may also want to send board information such as minimum value, size, etc to display on the client. The size of the leaderboard (number of entries) especially can be helpful for clients to have. With the size, you can display that users have a rank of greater than the size of the leaderboard when they do not fit in the size cap or do not meet the minimum value to be on the leaderboard, and thus are not on the board.

### top_cache_changed

`board.top_cache_changed`

> Fires when the cache for the top entries changed. When this fires, you can send clients the top few entries of the board from `board:GetTop()` to display. This will not work if you have `config.retrieve_top <= 0`.

### bottom_cache_changed

`board.bottom_cache_changed`

> Similar to `board.top_cache_changed`, but for the bottom entries.

### id_cache_changed

`board.id_cache_changed`

> Fires when the cache for a specific id's information changed. When this fires, you may want to send clients the entries surrounding them on the leaderboard, as well as information like the user's score and rank. The entries around the player will not be available if `config.board_track_before` and `config.board_track_after` are both 0. See `board:GetIdInfo(id)` for information on what to send clients.

## Types

### Entry
```lua
type Entry = {
	key: number,
	points: number,
	timestamp: number
}
```

### BoardEntry
```lua
type BoardEntry = {
	rank: number,
	entry: Entry
}
```

### IdData
```lua
type IdData = {
	rank: number,
	entry: Entry,
	surrounding_entries: {BoardEntry}?
}
```

### FullIdData
```lua
type FullIdData = {
	rank: number,
	entry: Entry,
	surrounding_entries: {BoardEntry}
}
```

### IdInfo
```lua
type IdInfo = {
    rank: number,
    entry: Entry
}
```

### BoardInfo
```lua
type BoardInfo = {
	cap: number?,
	min: number?,
	size: number,
	cutoff: number?
}
```

## Methods

### UpdateBoardAsync

```lua
function board:UpdateBoardAsync(dont_reset_timer: boolean?)
```

> Set `dont_reset_timer` to `true` if you want this to not affect the regular update cycle on the board. If it is `false` or `nil`, then the update timer of the board will reset.
> 
> This will force update the leaderboard, retrieving info on all tracked ids and applying changes from `board:SetId(id)` and `board:RemoveId(id)` calls. This will also make `board:GetTop()`, `board:GetBottom()`, and `board:GetIdInfo(id)` calls return more up-to-date information.
> 
> This method is equivalent to what happens when the update loop as specified by `config.board_update_delay` updates.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### UpdateIdAsync

```lua
function board:UpdateIdAsync(id: number)
```

> Equivalent to `board:UpdateBoardAsync()` except it only updates a single ID. This will immediately write pending changes to this id's entry and retrieve the latest information about the id as defined in the config. This will push changes from `board:SetId(id)` and `board:RemoveId(id)`, and make `board:GetIdInfo(id)` return more up to date information.
>
> Note that if the ID was not tracked when you made this request, then no read operations will be made. Only write operations from `board:SetId(id)` and `board:RemoveId(id)` for this specific id will be pushed.

### TimeUntilRefresh

```lua
function board:TimeUntilRefresh(): number
```

> Returns the number of seconds remaining until the next automatic update as specified by `config.board_update_delay`. Will return `-1` if the board is configured to not automatically update.

### GetIdInfo

```lua
function board:GetIdInfo(id: number): (boolean, IdData?)
```

> Gets cached information about the specified id. This is how you get the rank, value, and surrounding entries of traccked ids. Will return `false, nil` if the specified `id` has no cached values. This will happen if `id` is not tracked, if there has not been a board update since it started being tracked, or if all requests made to retrieve the info of this id failed. Assuming `id` is tracked and that you have `config.immediately_retrieve_newly_tracked_ids` set to `true`, then if this returns `false, nil`, assume connection to your server failed, or the information is still being retrieved if the id just began getting tracked. It will return `true, nil` if the id was retrieved but was not on the board. If this happens, assume no set request has gone through for this id yet, or that they did not make it onto the leaderboard due to the size cap or the minimum value cutoff. Otherwise, it will return `true, IdData`. The `surrounding_entries` field of the returned `IdData` will only be nil if `config.board_track_before <= 0` and `config.board_track_after <= 0`. It is possible for `surrounding_entries` to not be nil even if `config.board_track_before <= 0` and `config.board_track_after <= 0`, however.
>
> Note that what this returns will not reflect any scheduled changes from `board:SetId(id)` or `board:RemoveId(id)`. It will only reflect the cached information from the last request made to the server on an update. If you want to get scheduled changes, use the method `board:GetSetpoint(id)`.

### GetIdInfoAsync

```lua
function board:GetIdInfoAsync(id: number): IdInfo?
```

> Gets the rank and Entry of the entry with id `id`. Returns `nil` if no entry has the specified id.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### SetId

```lua
function board:SetId(id: number, value: number)
```

> Next time the id is updated (either from `board:UpdateIdAsync(id)`, `board:UpdateBoardAsync()`, or from the automatic updates from `config.board_update_delay`), the entry with id `id` will have its value/points set to `value`. If such an entry does not exist, it will be added, assuming that the entry does not get truncated either due to the board size cap or the minimum value cutoff. If you call this function multiple times between updates, only the last set will be kept.
> 
> This function does not directly make a request, but instead schedules a request on the next update. You can call this function as many times as you want with no drawbacks. This function will not yield.

### SetIdAsync

```lua
function board:SetIdAsync(id: number, value: number): boolean
```

> Updates `id` to have `value` on the leaderboard. Returns whether or not the value ended up on the leaderboard (`true` if it was added or was already on the board, `false` if it was truncated due to the size cap or the minimum value cutoff). This will also clear any pending set requests from `SetId` or `RemoveId` to make sure your set doesn't get overridden.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### RemoveId

```lua
function board:RemoveId(id: number)
```

> Functions the same as `board:SetId(id)`, except instead of scheduling a change to the entry, it schedules the removal of the entry.

### RemoveIdAsync

```lua
function board:RemoveIdAsync(id: number): boolean
```

> Removes `id` from the leaderboard. Returns whether or not the value was on the leaderboard before being removed. This will also clear any pending set requests from `SetId` or `RemoveId` to make sure your removal doesn't get overridden.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### GetSetpoint

```lua
function board:GetSetpoint(id: number): (boolean, number?)
```

> This is related to `board:SetId(id)` and `board:RemoveId(id)`. This will return `false, nil` if the entry with the specified `id` is not currently scheduled to be updated on the next board update. If it is scheduled to have its value updated on the next update, then it will return `true, value` if it is scheduled to be set, or `true, nil` if the id is scheduled to be removed from the board.

### GetTop

```lua
function board:GetTop(): {BoardEntry}?
```

> Will return the top `config.retrieve_top` entries. Will error if `config.retrieve_top <= 0`. This will return nil if the top values are not cached. This should only happen if a request to retrieve the top entries fails. If it returns nil, assume that connecting to the server failed.
>
> This request uses cached data, so things may be slightly out of date based on your `config.board_update_delay` configuration. That said, calling this function makes no request, so call it as much as you'd like. This function will not yield.

### GetTopAsync

```lua
function board:GetTopAsync(count: number, dont_use_cache: boolean?): {BoardEntry}
```

> Gets the top `count` entries of the leaderboard. If `dont_use_cache` is `false` or `nil`, then it will use a cache on the server side to reduce server strain. Check out your server-side configuration for more information on this. If `dont_use_cache` is `true`, the cache will be bypassed and the newest data will always be returned.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### GetBottom

```lua
function board:GetBottom(): {BoardEntry}?
```

> Will return the bottom `config.retrieve_bottom` entries. Will error if `config.retrieve_bottom <= 0`. This will return nil if the top values are not cached. This should only happen if a request to retrieve the bottom entries fails. If it returns nil, assume that connecting to the server failed.
>
> This request uses cached data, so things may be slightly out of date based on your `config.board_update_delay` configuration. That said, calling this function makes no request, so call it as much as you'd like. This function will not yield.

### GetBottomAsync

```lua
function board:GetBottomAsync(count: number, dont_use_cache: boolean?): {BoardEntry}
```

> Gets the bottom `count` entries of the leaderboard. If `dont_use_cache` is `false` or `nil`, then it will use a cache on the server side to reduce server strain. Check out your server-side configuration for more information on this. If `dont_use_cache` is `true`, the cache will be bypassed and the newest data will always be returned.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### AroundIdAsync

```lua
function board:AroundIdAsync(id: number, before: number, after: number): FullIdData?
```

> Gets the `before` entries before `id` and the `after` entries after `id` in the leaderboard, as well as the entry with `id` itself for a total of `before + after + 1` entries. Will return `nil` only if `id` is not on the board. The returned `surrounding_entries` field may not contain all `before + after + 1` entries if there are not enough entries around `id` in the board. If `before >= config.board_track_before`, `after >= config.board_track_after`, and `id` is being tracked, then the cache for `id` will be updated based on the results of this call. To disable this, set `config.update_caches_on_async_request` to `false`.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### AfterIdAsync

```lua
function board:AfterIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries after `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the top people on the leaderboard after the top few you have retrieved with `config.retrieve_top`. For that, you would just call this function with `id` set to the last id in the cached top entries from `board:GetTop()`.
>
> Take care to make sure `count` is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### BeforeIdAsync

```lua
function board:BeforeIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries before `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the bottom people on the leaderboard after the bottom few you have retrieved with `config.retrieve_bottom`. For that, you would just call this function with `id` set to the last id in the cached top entries from `board:GetBottom()`. Note that values returned by this function have ranks in decreasing order.
>
> Take care to make sure `count` is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### GetRangeAsync

```lua
function board:GetRangeAsync(start_rank: number, end_rank: number): {BoardEntry}
```

> Returns all entries between the ranks `start_rank` and `end_rank` inclusive. Will error if `start_rank > end_rank` or if `start_rank <= 0` or `end_rank <= 0`. Ranks start at 1 and cannot be 0. The returned list may have less than `end_rank - start_rank + 1` entries if `end_rank` is greater than the size of the board. The returned list can be empty.
>
> Take care to make sure the range is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### GetAtRankAsync

```lua
function board:GetAtRankAsync(rank: number): Entry?
```

> Get the entry at the specified rank. Will return nil if no entry has that rank. Will error if `rank <= 0`, since ranks start at 1.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### PredictInBounds

```lua
function board:PredictInBounds(value: number): boolean
```

> Based on the most recently retrieved information about the board (size cap, minimum value, minimum value cutoff), this function will attempt to predict whether or not an entry with value `value` could be on the board. Returns `true` if it determines it could be on the board, `false` if it determines it would most likely be truncated and not added. This is used internally on updates to reduce the number of set requests that are made.

### GetBoardInfo

```lua
function board:GetBoardInfo(): BoardInfo
```

> Returns the last retrieved information about the board. If retrieving info about the board failed on the last update, this will default to returning `{cap = nil, min = nil, size = 0, cutoff = nil}`. This uses cached information from the last board update. `cap` will be nil if no size cap is set on the board, otherwise it will be the size cap. `min` will be nil only if the board is empty. Otherwise, it will return the smallest value/points on an entry in the leaderboard. `size` will always be the number of entries in the leaderboard. `cutoff` will be nil if no minimum value cutoff is set on the board, otherwise it will be the minimum value allowed on the board.

### GetBoardInfoAsync

```lua
function board:GetBoardInfoAsync(): BoardInfo
```

> Returns some information about the board. Consider using `board:GetBoardInfo()` instead if you do not absolutely need the most up-to-date information. `cap` will be nil if no size cap is set on the board, otherwise it will be the size cap. `min` will be nil only if the board is empty. Otherwise, it will return the smallest value/points on an entry in the leaderboard. `size` will always be the number of entries in the leaderboard. `cutoff` will be nil if no minimum value cutoff is set on the board, otherwise it will be the minimum value allowed on the board.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### BatchExecuteAsync

```lua
function board:BatchExecuteAsync(batch: Batch): {unknown}
```

> Executes the specified batch request and returns a list of the responses. Note that some of the items in the return list may be nil, as certain batch requests can return nil. This means that you cannot use `#result` to get the length of the result. Instead, use `batch:GetSize()`. The returned list will have entries be in order of the requests in the batch request. Thus, if a set request was 3rd in the batch request, then the boolean result of the set request will be at index 3 of the returned table.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

### GetConfig

```lua
function board:GetConfig(): Config
```

> Returns a read-only (frozen) version of the config of the board.

### ClearIdCache

```lua
function board:ClearIdCache(id: number)
```

> Clears all cached data for an id.

### TrackId

```lua
function board:TrackId(id: number)
```

> Begins tracking `id`. If `config.immediately_retrieve_newly_tracked_ids` is `true`, then it will also retrieve information about the id to cache. This information will be retrieved on a different thread and thus will not be immediately available, but it will begin being retrieved immediately, and when it is retrieved, the `id_cache_changed` event will fire. Will do nothing if id is already tracked.

### StopTrackingId

```lua
function board:StopTrackingId(id: number)
```

> Stops tracking `id`. If `config.clear_cache_on_tracking_stop` is `true`, then it will also call `board:ClearIdCache(id)` after a `task.defer` (at the end of the current resumption cycle). This means cached information will still be available for a moment after the id stops being tracked, but only a moment. Will not do anything if id is already not tracked.

### IsTracked

```lua
function board:IsTracked(id: number): boolean
```

> Checks if an id is being tracked on the board.

### Destroy

```lua
function board:Destroy()
```

> Clears all event connections, destroys all events, and makes the board no longer retrievable by `Board.GetExistingBoardByKey(key)`. This will allow you to create a new board object with the same key, and will allow the board object to get garbage collected.

## Static Functions

### DefaultConfig

```lua
function Board.DefaultConfig(): Config
```

> Creates a default editable configuration to use in the creation of a board.

### ValidateConfig

```lua
function Board.ValidateConfig(config: Config)
```

> Throws an error if the passed value is not a valid config. Used internally to check parameters to `LoadBoardAsync`.

### IsI64Safe

```lua
function Board.IsI64Safe(id: number): boolean
```

> Returns true if the passed value can be safely converted into a I64 value (signed 64 bit integer). This can be used to validate ids for entries.

### AssertI64Safe

```lua
function Board.AssertI64Safe(id: number)
```

> Throws an error if the passed value cannot be safely converted into a I64 value (signed 64 bit integer). This can be used to validate ids for entries.

### IsGoodF64

```lua
function Board.IsGoodF64(value: number): boolean
```

> Returns true if the passed value is a "good F64" (meaning it is not infinity and is not NaN). This can be used to validate values for entries.

### AssertGoodF64

```lua
function Board.AssertGoodF64(value: number)
```

> Throws an error if the passed value is not a "good F64" (meaning it is not a number, is infinity or is NaN). This can be used to validate values for entries.

### IsUSizeSafe

```lua
function Board.IsUSizeSafe(rank: number): boolean
```

> Returns true if the passed value can be safely converted into a USize (unsigned 32 bit integer). This can be used to validate ranks.

### AssertUSizeSafe

```lua
function Board.AssertUSizeSafe(rank: number)
```

> Throws an error if the passed value cannot be safely converted into a USize (unsigned 32 bit integer). This can be used to validate ranks.

### LoadBoardAsync

```lua
function Board.LoadBoardAsync(server: string, port: number, key: Secret|string, config: Config): Board
```

> Creates a new Board object connected to the server at `http://<server>:<port>/`. Uses the specified API Key. This will perform multiple HTTP requests on creation, but if those HTTP requests fail, a board will still return. This board will act as if it is empty in this case, and later requests may still work. Since this involves HTTP requests, this call will yield. While a board is being created, another board cannot be created with the same key, but if you attempt to get the board with `GetExistingBoardByKey`, it will return nil.
> 
> Because boards can be retrieved with `GetExistingBoardByKey`, boards will not be garbage collected. If you wish to delete a board object, use `board:Destroy()`.
> 
> Will error if invalid parameters are passed. Will also error if a board has already been created with the specified key.

### GetExistingBoardByKey

```lua
function Board.GetExistingBoardByKey(key: Secret|string): Board?
```

> Returns an existing board with the specified key. Will return nil if there is no such board or the board is still being created (see `LoadBoardAsync`).

### GetWorkingVersion

```lua
function Board.GetWorkingVersion(): string
```

> Returns a string representing the server version that the module supports. If the working version does not match the server version, requests will fail. Currently `1.0.0`.

## Batch Requests

Batch requests execute multiple requests at once, reducing the number of HTTP requests made and marginally reducing network strain on the server for multiple requests. Also allows for faster responses. Requests added to a batch request will execute in the order they were added in. If any request in the batch errors, all requests error and no result is returned, even though some requests may have executed. The methods should safeguard this, however, ensuring that parameters are valid so that you do not recieve a 400 error. Since requests execute in order, if you have a read request before a write request to the same entry, the read request will return information from before the write request was performed. 

### CreateBatchRequest

!!! info "update_caches_on_async_request"
    Batch requests are also affected by the configuration option `config.update_caches_on_async_request`.

```lua
function Board.CreateBatchRequest(): Batch
```

> Creates an empty batch request object.

### BatchAroundId

```lua
function Board.BatchAroundId(batch: Batch, id: number, before: number, after: number)
```

> Adds an "Around" request to the batch. Functions similarly to `board:AroundIdAsync(id, before, after)`. Adds a `FullIdData?` to the result.

### BatchAfterId

```lua
function Board.BatchAfterId(batch: Batch, id: number, count: number)
```

> Adds an "After" request to the batch. Functions similarly to `board:AfterIdAsync(id, count)`. Adds a `{BoardEntry}?` to the result.

### BatchBeforeId

```lua
function Board.BatchBeforeId(batch: Batch, id: number, count: number)
```

> Adds a "Before" request to the batch. Functions similarly to `board:BeforeIdAsync(id, count)`. Adds a `{BoardEntry}?` to the result.

### BatchGetRange

```lua
function Board.BatchGetRange(batch: Batch, start_rank: number, end_rank: number)
```

> Adds a "Range" request to the batch. Functions similarly to `board:GetRangeAsync(start_rank, end_rank)`. Adds a `{BoardEntry}` to the result.

### BatchAtRank

```lua
function Board.GetAtRankAsync(batch: Batch, rank: number)
```

> Adds an "AtRank" request to the batch. Functions similarly to `board:GetAtRankAsync(rank)`. Adds an `Entry?` to the result.

### BatchGetEntryWithId

```lua
function Board.BatchGetEntryWithId(batch: Batch, id: number)
```

> Adds a "Get" request to the batch. Functions similarly to `board:GetEntryWithIdAsync(id)`. Adds an `Entry?` to the result.

### BatchGetBoardInfo

```lua
function Board.BatchGetBoardInfo(batch: Batch)
```

> Adds a "Board" request to the batch. Functions similarly to `board:GetBoardInfoAsync()`. Adds a `BoardInfo` to the result.

### BatchGetTop

```lua
function Board.BatchGetTop(batch: Batch, count: number, dont_use_cache: boolean?)
```

> Adds a "Top" request to the batch. Functions similarly to `board:GetTopAsync(count, dont_use_cache)`. Adds a `{BoardEntry}` to the result.

### BatchGetBottom

```lua
function Board.BatchGetBottom(batch: Batch, count: number, dont_use_cache: boolean?)
```

> Adds a "Bottom" request to the batch. Functions similarly to `board:GetBottomAsync(count, dont_use_cache)`. Adds a `{BoardEntry}` to the result.

### BatchSetId

```lua
function Board.BatchSetId(batch: Batch, id: number, value: number)
```

> Adds an "Update" request to the batch. Functions similarly to `board:SetIdAsync(id, value)`. Adds a `boolean` to the result.

### BatchRemoveId

```lua
function Board.BatchRemoveId(batch: Batch, id: number)
```

> Adds a "Remove" request to the batch. Functions similarly to `board:RemoveIdAsync(id)`. Adds a `boolean` to the result.

### BatchGetIdInfo

```lua
function Board.BatchGetIdInfo(batch: Batch, id: number)
```

> Adds an "Info" request to the batch. Functions similarly to `board:GetIdInfoAsync(id)`. Adds an `IdInfo?` to the result.

## Batch Methods

### GetSize

```lua
function batch:GetSize(): number
```

> Returns the number of requests in the batch request.

### TypeAtIndex

```lua
function Batch:TypeAtIndex(index: number): string?
```

> Returns the type of request at the specified index, as a string. Returns `nil` if there is no request at that index. 