Roblox allows you to make 500 HTTP requests a minute, or about 8 requests a second. Since this relies on an external leaderboard server, this module will make many HTTP requests. In case your game is making the most of this limit, this provides a reference for how many requests this system will take.

#### UpdateBoardAsync

When your board automatically updates on a timer from `config.board_update_delay`, or when you call `board:UpdateBoardAsync()`, **2** HTTP requests will be made, one to retrieve new information about the board, then a batch request.

#### UpdateIdAsync

This makes use of a batch request, and only makes **1** HTTP request.

#### ExecuteBatchAsync

Although this batch request is comprised of many smaller requests, this is only **1** HTTP request.

#### LoadBoardAsync

When creating a leaderboard, it will always make a request to retrieve board information. As long as you dont have `config.board_update_delay <= 0` and either `immediately_retrieve_newly_tracked_ids == false` or `automatically_track_player_ids == false`, then it will also make a second request to retrieve information about the players already in your game. Thus, this will take **1-2** HTTP requests.

#### TrackId

If you have `config.immediately_retrieve_newly_tracked_ids`, then a request will be made whenever an id begins being tracked. This includes when ids are tracked automatically from player joins with `config.automatically_track_player_ids`. Thus, when a player joins or a id is tracked, **1** HTTP request will be made.

#### Everything Else

Anything else that has Async in the name will make **1** HTTP request. If a function interacts with the cache, it will not make an HTTP request.