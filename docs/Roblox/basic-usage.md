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

#### BoardEntry

```lua
type BoardEntry = {
	rank: number,
	entry: Entry
}
```

### Methods

#### UpdateBoardAsync

```lua
function board:UpdateBoardAsync(dont_reset_timer: boolean?)
```

> Set `dont_reset_timer` to `true` if you want this to not affect the regular update cycle on the board. If it is `false` or `nil`, then the update timer of the board will reset.
> 
> This will force update the leaderboard, retrieving info on all players in the game and applying changes from `:SetId()` calls.
> 
> This method is equivalent to what happens when the update loop as specified by `config.board_update_delay` updates.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### TimeUntilRefresh

```lua
function board:TimeUntilRefresh(): number
```

> Returns the number of seconds remaining until the next automatic update as specified by `config.board_update_delay`. Will return `-1` if the board is configured to not automatically update.

#### AfterIdAsync

```lua
function board:AfterIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries after `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the top people on the leaderboard after the top few you have retrieved with `config.retrieve_top`. For that, you would just call this function with `id` set to the last id in the cached top entries.
>
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.

#### BeforeIdAsync

```lua
function board:BeforeIdAsync(id: number, count: number): {BoardEntry}?
```

> Returns the `count` entries before `id` in the board. Useful if you want to make a "view more" system, where, for example, a player could press a button to view more of the bottom people on the leaderboard after the bottom few you have retrieved with `config.retrieve_bottom`. For that, you would just call this function with `id` set to the last id in the cached top entries. Note that values returned by this function have ranks in decreasing order.
> 
> This function will error if the request fails, so make sure to have appropriate error checking. Since it makes a HTTP request, it will also yield.