## Cached Usage
Typically, you should be able to do anything you need to do with the cache methods. Using those may lead to some out of date data, but that should be good enough for almost any situation. This is considered [basic usage](../basic-usage.md). All of the most common information you'll need is available. It works perfectly for simple situations where all you need is a simple visible leaderboard with no other interactions. Depending on your use case however, this may not be enough, or the cache functions won't do what you need them to. For that, it may be better to do the Cacheless Usage.

## Cacheless Usage
This is to treat the roblox module as a wrapper for the server-side APIs. In this case, you'd probably want a config that looks like this:
```lua
local config = Board.DefaultConfig()

-- disable updating
config.board_update_delay = -1

-- make sure nothing is being tracked
config.automatically_track_player_ids = false
```
This way, we won't have any automatic updates and nothing will be tracked -- no cache usage. Then, we can just use our async functions however we want to do what we need. It is suggested that you make use of batch requests where you can. You can also set up your own cache system if you'd like.

## Hybrid Usage

It is important to know how the cachless and cached functions interact with one another. There are many use cases when you'll want a primarily cached board with some non-cached instances. For example, if you had a donation leaderboard, you would probably want donations to appear immediately. You could do this by making a `SetIdAsync` request if you just want the value to change immediately, you can use a `SetId` and an `UpdateIdAsync` to update just that person, or you can even use a `SetId` and an `UpdateBoardAsync` if you also want the top and bottom updating. Generally, in you will be able to do everything you need to with `UpdateIdAsync` and `UpdateBoardAsync` along with your cached functions. If you need to know how other functions interact however, this should be documented in the [Complete API](./complete-API.md).

!!! warning "Reminder: Do Not Use RankSwift for Data Storage"
    Due to how the server handles data, in the event of a crash, data may fall back, leading to some data-loss. Furthermore, because of the size cap and minimum value cutoff features, it is likely that data will be removed from the board. For this reason, RankSwift should not be used for your data storage, only data aggregation. Use Roblox's Datastores for the actual storage of your data, then push the data from those stores onto RankSwift for detailed leaderboard and rank information.

## Non-Player Ids

Say that your leaderboard doesn't store players, but instead stores guilds of players. You would need to assign guilds numerical ids similar to player ids, just make sure they meet the I64 criterion. The leaderboard system will support this, however, you'll need to keep some considerations in mind. The cache system functions with the idea that the same ids aren't being modified by multiple game servers at the same time. If you have a score that may be shared between servers, it is suggested that you use a roblox datastore or memory store to manage the group's score, then push that to the leaderboard with `SetIdAsync` requests. The cache system should still work fine for read-only in this scenario. For this, you would want to disable the `config.automatically_track_player_ids` setting and manually track the guild ids.

## Incredibly High Values

Say that you have a simulator game with exponential score growth, but you still want to use RankSwift for your scores. Since you shouldn't be using RankSwift for your data storage, you can lose some precision of your data to easily account for this situation. If you are doing this, you are likely using a module for storing and managing big integers. In order to get this into a F64 to put on the board, take a logarithm of the value using a built in method from that module. This will keep the values in reasonable areas and keep enough precision that similar scores should still be distinguishable in most cases. When displaying the information, you can then raise your base to the power of the value from RankSwift to get an approximation of the original score.