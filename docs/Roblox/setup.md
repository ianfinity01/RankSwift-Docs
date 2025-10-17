Don't worry! This will be easy, we made you a ModuleScript that will make it very easy to interface with your Leaderboard Server. You can get it [here](https://create.roblox.com/store/asset/120356645529088){:target="_blank"}.

## Importing the Module

Once you have gotten the module from the Roblox website, open your Roblox Studio on the game you want to connect your leaderboard to. Open the toolbox, go to the Inventory tab (denoted by 4 boxes), and find the Module under "My Models". Once you have imported the Module, it is suggested that you put it in ServerScriptService. This module cannot be used by clients.

## Creating a Board

In a server script or module script, require the module. You now need to create Board objects that will serve as connections to your RankSwift boards. Although you can create multiple Board objects for multiple RankSwift boards, you can only have one Board per RankSwift Board. If you create multiple Boards on the same Roblox Server for the same RankSwift Board, things may not behave as intended. Because of this, if you need to access the Board for a specific RankSwift Board in multiple scripts, it is suggested that you store it in a ModuleScript so that it is only created once.

Boards take the following parameters:

- `server` (string): The IP address of the server where you are hosting RankSwift.
- `port` (number): The port that the server is running on. Make sure the port is correctly forwarded on the server!
- `key` (Secret or string): The API key for the board.
- `config` ([Config](./configuration.md)): The configuration for this Board object.

A script creating a Board object could look like this:
```lua
local ServerScriptService = game:GetService("ServerScriptService")
local Board = require(ServerScriptService.Board)

local config = Board.DefaultConfig()
local board = Board.LoadBoardAsync("123.45.678.910", 3895, "some-api-key-here", config)
```
Note that you create board with `LoadBoardAsync`. Assuming you give it valid inputs, this function will not error. If the board fails to retrieve important information on creation, it will default to acting like it belongs to an empty leaderboard until the server comes back online. It is marked as Async because it makes some HTTP requests and thus yields briefly.

!!! info "Secrets"
    In lieu of a string, your API key can be in the form of a [Secret](https://create.roblox.com/docs/reference/engine/datatypes/Secret){:target="_blank"}. This will keep your API key private from any other developers that have access to your game, as well as keep the key out of your game's code. You can set Secrets in the roblox creator portal, and retrieve them with `HttpService:GetSecret("secret name")`.

To properly use the Board, you'll want to configure it. To learn how to do this, move onto the [Configuration Page](./configuration.md).