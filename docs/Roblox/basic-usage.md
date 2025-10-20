Here are all the functions and configurations most people will need when using RankSwift.

## Configuration

The only configuration settings most people will need to mess with are the following:

```lua
local config = Board.DefaultConfig()

-- you might want to decrease the board update delay if your server isn't experiencing much strain (which it shouldn't be if you aren't a top game).
config.board_update_delay = 240 -- some interval (in seconds)

-- you will likely want players to see the people around them in the leaderboard so they know what their immediate competition looks like.
config.board_track_before = 15 -- set these to zero if you don't need to retrieve these players
config.board_track_after =  15 -- track the 15 entries before and after each user in your game, for a total of 31 entries (including the user itself).

-- retrieve the top 25 people in the leaderboard
config.retrieve_top = 25

-- config.retrieve_bottom also exists
```

## Board Usage

Here are some parts of the Board object that you may want to use:

### Events

Events can be connected to with `:Connect(callback)`, `:Once(callback)`, etc.

#### board_updated

`board.board_updated`

> Fires when a full update is made to the board. You can use this in conjunction with `board:TimeUntilRefresh()` to make it so that an update timer on clients updates. You may also want to send board information such as minimum value, size, etc to display on the client. The size of the leaderboard (number of entries) especially can be helpful for clients to have. With the size, you can display that users have a rank of greater than the size of the leaderboard when they do not fit in the size cap or do not meet the minimum value to be on the leaderboard, and thus are not on the board.

#### top_cache_changed

`board.top_cache_changed`

> Fires when the cache for the top entries changed. When this fires, you can send clients the top few entries of the board from `board:GetTop()` to display. This will not work if you have `config.retrieve_top <= 0`.

#### bottom_cache_changed

`board.bottom_cache_changed`

> Similar to `board.top_cache_changed`, but for the bottom entries.

#### id_cache_changed

`board.id_cache_changed`

> Fires when the cache for a specific user's information changed. When this fires, you may want to send clients the entries surrounding them on the leaderboard, as well as information like the user's score and rank. The entries around the player will not be available if `config.board_track_before` and `config.board_track_after` are both 0. See `board:GetIdInfo(id)` for information on what to send clients.

### Types

#### Entry
```lua
type Entry = {
	key: number,
	points: number,
	timestamp: number
}
```

#### BoardEntry
```lua
type BoardEntry = {
	rank: number,
	entry: Entry
}
```

#### IdData
```lua
type IdData = {
	rank: number,
	entry: Entry,
	surrounding_entries: {BoardEntry}?
}
```

#### BoardInfo
```lua
type BoardInfo = {
	cap: number?,
	min: number?,
	size: number,
	cutoff: number?
}
```

### Methods

#### UpdateBoardAsync

```lua
function board:UpdateBoardAsync(dont_reset_timer: boolean?)
```

> Set `dont_reset_timer` to `true` if you want this to not affect the regular update cycle on the board. If it is `false` or `nil`, then the update timer of the board will reset.
> 
> This will force update the leaderboard, retrieving info on all players in the game and applying changes from `board:SetId(id)` calls.
> 
> This method is equivalent to what happens when the update loop as specified by `config.board_update_delay` updates.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### UpdateIdAsync

```lua
function board:UpdateIdAsync(id: number)
```

> Equivalent to `board:UpdateBoardAsync()` except it only updates a single ID. This will immediately write pending changes to this id's entry and retrieve the latest information about the user as defined in the config. This will push changes from `board:SetId(id)` and `board:RemoveId(id)`.
>
> Note that if the ID was not tracked when you made this request, then no read operations will be made. Only write operations from `board:SetId(id)` and `board:RemoveId(id)` for this specific id will be pushed.

#### TimeUntilRefresh

```lua
function board:TimeUntilRefresh(): number
```

> Returns the number of seconds remaining until the next automatic update as specified by `config.board_update_delay`. Will return `-1` if the board is configured to not automatically update.

#### GetIdInfo

```lua
function board:GetIdInfo(id: number): (boolean, IdData?)
```

> Gets cached information about the specified id. This is how you get the rank, value, and surrounding entries of the players in your game. Will return `false, nil` if the specified `id` has no cached values. This will happen if `id` is not tracked, if there has not been a board update since it started being tracked, or if all requests made to retrieve the info of this id failed. Assuming `id` is the user id for a player you are tracking (a player in your game) and that you have `config.immediately_retrieve_newly_tracked_ids` set to `true`, then if this returns `false, nil`, assume connection to your server failed, or the player just joined the game and this data was not retrieved yet. It will return `true, nil` if the id was retrieved but was not on the board. If this happens, assume no set request has gone through for this id yet, or that they did not make it onto the leaderboard due to the size cap or the minimum value cutoff. Otherwise, it will return `true, IdData`. The `surrounding_entries` field of the returned `IdData` will only be nil if `config.board_track_before <= 0` and `config.board_track_after <= 0`. It is possible for `surrounding_entries` to not be nil even if `config.board_track_before <= 0` and `config.board_track_after <= 0`, however.
>
> Note that what this returns will not reflect any scheduled changes from `board:SetId(id)` or `board:RemoveId(id)`. It will only reflect the cached information from the last request made to the server on an update. If you want to get scheduled changes, use the method `board:GetSetpoint(id)`.

#### SetId

```lua
function board:SetId(id: number, value: number)
```

> Next time the id is updated (either from `board:UpdateIdAsync(id)`, `board:UpdateBoardAsync()`, or from the automatic updates from `config.board_update_delay`), the entry with id `id` will have its value/points set to `value`. If such an entry does not exist, it will be added, assuming that the entry does not get truncated either due to the board size cap or the minimum value cutoff. If you call this function multiple times between updates, only the last set will be kept.
> 
> This function does not directly make a request, but instead schedules a request on the next update. You can call this function as many times as you want with no drawbacks. This function will not yield.

#### RemoveId

```lua
function board:RemoveId(id: number)
```

> Functions the same as `board:SetId(id)` except instead of scheduling a change to the entry, it schedules the removal of the entry.

#### GetSetpoint

```lua
function board:GetSetpoint(id: number): (boolean, number?)
```

> This is related to `board:SetId(id)` and `board:RemoveId(id)`. This will return `false, nil` if the entry with the specified `id` is not currently scheduled to be updated on the next board update. If it is scheduled to have its value updated on the next update, then it will return `true, value` if it is scheduled to be set, or `true, nil` if the id is scheduled to be removed from the board. Generally, you won't need to use this.

#### GetTop

```lua
function board:GetTop(): {BoardEntry}?
```

> Will return the top `config.retrieve_top` entries. Will error if `config.retrieve_top <= 0`. This will return nil if the top values are not cached. This should only happen if a request to retrieve the top entries fails. If it returns nil, assume that connecting to the server failed.
>
> This request uses cached data, so things may be slightly out of date based on your `config.board_update_delay` configuration. That said, calling this function makes no request, so call it as much as you'd like. This function will not yield.

#### GetBottom

```lua
function board:GetBottom(): {BoardEntry}?
```

> Will return the bottom `config.retrieve_bottom` entries. Will error if `config.retrieve_bottom <= 0`. This will return nil if the top values are not cached. This should only happen if a request to retrieve the bottom entries fails. If it returns nil, assume that connecting to the server failed.
>
> This request uses cached data, so things may be slightly out of date based on your `config.board_update_delay` configuration. That said, calling this function makes no request, so call it as much as you'd like. This function will not yield.

#### AfterIdAsync

```lua
function board:AfterIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries after `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the top people on the leaderboard after the top few you have retrieved with `config.retrieve_top`. For that, you would just call this function with `id` set to the last id in the cached top entries from `board:GetTop()`.
>
> Take care to make sure `count` is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### BeforeIdAsync

```lua
function board:BeforeIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries before `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the bottom people on the leaderboard after the bottom few you have retrieved with `config.retrieve_bottom`. For that, you would just call this function with `id` set to the last id in the cached bottom entries from `board:GetBottom()`. Note that values returned by this function have ranks in decreasing order.
>
> Take care to make sure `count` is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### GetRangeAsync

```lua
function board:GetRangeAsync(start_rank: number, end_rank: number): {BoardEntry}
```

> Returns all entries between the ranks `start_rank` and `end_rank` inclusive. Also helpful for making a "view more" system, as described above. Will error if `start_rank > end_rank` or if `start_rank <= 0` or `end_rank <= 0`. Ranks start at 1 and cannot be 0. The returned list may have less than `end_rank - start_rank + 1` entries if `end_rank` is greater than the size of the board. The returned list can be empty.
>
> Take care to make sure the range is not too large, as it may cause the server to stall and return a massive amount of entries. There is no safeguard in place, you need to filter requests yourself.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### GetAtRankAsync

```lua
function board:GetAtRankAsync(rank: number): Entry?
```

> Get the entry at the specified rank. Will return nil if no entry has that rank. Will error if `rank <= 0`, since ranks start at 1.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### GetBoardInfo

```lua
function board:GetBoardInfo(): BoardInfo
```

> Returns the last retrieved information about the board. If retrieving info about the board failed on the last update, this will default to returning `{cap = nil, min = nil, size = 0, cutoff = nil}`. This uses cached information from the last board update. `cap` will be nil if no size cap is set on the board, otherwise it will be the size cap. `min` will be nil only if the board is empty. Otherwise, it will return the smallest value/points on an entry in the leaderboard. `size` will always be the number of entries in the leaderboard. `cutoff` will be nil if no minimum value cutoff is set on the board, otherwise it will be the minimum value allowed on the board.

#### GetConfig

```lua
function board:GetConfig(): Config
```

> Returns a read-only (frozen) version of the config of the board.

# Missing Something?

You can find a complete list of every method and function on the [API Page](./Advanced-Usage/complete-API.md)