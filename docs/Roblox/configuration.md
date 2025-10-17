When creating a Board object, you specify a config. For the prior example, we created a default config, but let's see what you can do with this config.

## Format
The format of the config is as follows:
```lua
type Config = {
	board_update_delay: number,
	immediately_retrieve_newly_tracked_ids: boolean,
	automatically_track_player_ids: boolean,
	clear_cache_on_tracking_stop: boolean,
	board_track_before: number,
	board_track_after: number,
	retrieve_top: number,
	retrieve_bottom: number,
	use_top_and_bottom_cache: boolean,
	update_caches_on_async_request: boolean
}
```
And the default config has these values:
```lua
local DEFAULT_CONFIG = {
    board_update_delay = 10*60,
    immediately_retrieve_newly_tracked_ids = true,
    automatically_track_player_ids = true,
    clear_cache_on_tracking_stop = true,
    board_track_before = 0,
    board_track_after = 0,
    retrieve_top = 0,
    retrieve_bottom = 0,
    use_top_and_bottom_cache = true,
    update_caches_on_async_request = true
}
```
You can make your config manually as a table, but it is suggested that you create a default config and edit its values like this:
```lua
local config = Board.DefaultConfig()

config.board_update_delay = 1000 -- change any configuration option
```

## Configuration Options

Below is a list of the functions of each configuration option:

- `board_update_delay`: The delay in seconds before the board will attempt to retrieve new data about the board from the server. Longer values will reduce the strain the server but make it take longer for changes to appear in your game. The default is 10 minutes. You can manually force the board to update with `board:UpdateBoardAsync().` This command may error, and will reset the update timer (unless you pass true as a parameter). Set this to 0 or less to not automatically update the board.

- `immediately_retrieve_newly_tracked_ids`: When a new userId starts being tracked on the leaderboard, this defines whether or not the server should immediately attempt to retrieve information about the user from the server. If this is true, every time a new userId is tracked, the board will make a request to the Server to retrieve their information. If this is false, the user's data on the leaderboard may not be available until the next leaderboard update. In many cases you will want to leave this as `true`, but you can change it to `false` to reduce server strain.

- `automatically_track_player_ids`: This defines if the board should automatically start tracking the user ids of players who join the game/players who are already in the game. This will also make it so that when a player leaves the game, their userId will stop being tracked. Generally, you will want to leave this as `true` unless some players won't need to see their information on the leaderboard immediately, or if you are using something other than player userIds as the keys/ids on the leaderboard. You can manually start and stop tracking ids with `board:TrackId(userId)` and `board:StopTrackingId(userId)`. If you want to be careful about server strain, it is suggested to disable this and only start tracking a userId when a player attempts to view the leaderboard, and keep `immediately_retrieve_newly_tracked_ids` as `true`.

!!! warning "Stopping Tracking IDs"
    You need to make sure you stop tracking IDs when you no longer need to retrieve information about them (ex: when a player leaves the game). If you do not, information will continue to be retrieved about them until the game server closes. If you manually start tracking a user id, you will also have to manually stop tracking it.

!!! info "Tracked IDs"
    Tracked IDs are IDs that will have information about them updated and cached whenever the Leaderboard updates.

- `clear_cache_on_tracking_stop`: This defines if the board should clear all cached information about a userId when it stops being tracked (within a `task.defer` so that you can use the cache for a moment more). This is helpful to prevent memory leaks. You will usually want to leave this as `true`. The only time you will want this to be `false` is if you will want to use cached information about a userId for some time after it stops being tracked. If this is set to `false`, make sure to prevent memory leaks by running `board:ClearIdCache(id)` when you no longer need the cached information on an id.

- `board_track_before`: The number of users that should be retrieved before every tracked id. This is helpful when you want to display all users in a range around players in your game. Larger values will retrieve more of the players before the specified ID, but will put more strain on the server. There is NO SAFEGUARD on how large this value can be, just use common sense when setting it. Set this to 0 or a negative value to not retrieve entries before tracked ids.

- `board_track_after`: Similar to `board_track_before`, except the number of people after the user instead of before.

- `retrieve_top`: The number of the top users on the leaderboard to retrieve on leaderboard update. There is NO SAFEGUARD on how large this value can be, just use common sense when setting it. Useful for displaying the top people on the leaderboard.

- `retrieve_bottom`: Similar to `retrieve_top` except the bottom users on the leaderboard.

- `use_top_and_bottom_cache`: Whether or not the values automatically retrieved on automatic board update by `retrieve_top` and `retrieve_bottom` can use the cache on the server. It is suggested that you leave this as `true` since the cache on the server does not last very long and can save a lot of strain on the server. This should not make values too outdated on retrieval, but check your server config to make sure.

- `update_caches_on_async_request`: This defines whether or not caches should be automatically updated when you make manual async requests such as `GetIdInfoAsync`, `GetTopAsync`, `RemoveIdAsync`, `GetBoardInfoAsync`, etc. This DOES NOT affect `UpdateBoardAsync` or `UpdateIdAsync`, as the sole purpose of these is to update caches. This just keeps data more up to date, but some people may want caches to only update with board updates.

Now that it's configured, head onto the [Basic Usage Page] if you want to use this board in a usual way, the [Tutorial Page] if you want to see an example, or the [Advanced Usage Page] if you want to do more with this leaderboard in more niche use cases.