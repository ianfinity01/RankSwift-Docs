There are some important global settings you will want to know about. Here's where to find them and how to change them.

After your first time running the server, there should be a configuration file in it. It should be located in `/target/release/config.json`. Note that when you run your server with `cargo run --release`, the built executable and all of its surrounding files will be in `/target/release`. The `config.json` file will have the following format:

```json
{
    "port": 3895,
    "save_interval": 600,
    "lock_save": false,
    "cache_len": 5
}
```

!!! warning "Changing the Config"
    When changing the configurations, you'll need to shut down your server, make the changes, then start it back up. The server only reads changes to the config file on startup. Thus, it is best to set up your config when first getting started, but usually you won't have to change anything.

- `port` is the port that the server will run on. Usually you won't have to change this.

- `save_interval` is the delay (in seconds) that the server waits before attempting to back up the leaderboard to file. Since your leaderboards are all saved to file, in the event of your server crashing (due to, for example, the power going out), the server needs to periodically back up the boards to file so that when your server starts back up, it can pick up from where it left off. The larger this number, the less strain it will put on your server, but the more data you risk losing in the event of an unexpected crash. 

- `lock_save` is a more technical setting. Generally, you will want to leave this as false. When everything saves to file, it undergoes a complex process to keep a static version of the leaderboard without freezing it, so that it can write that version of the board to file. This process can lead to some temporary slow-downs when saving ends. These slowdowns should be short and inconsequential, unless your server is regularly using up every last bit of its CPU's resources while handling requests. If your server is incredibly high traffic, you may benefit from just letting the leaderboard lock up and carry out the save quickly, instead of having a prolongued slowdown negatively impacting your leaderboard's accessability. Be aware that when the leaderboard is locked up while saving, all requests to that board will be stalled or will possibly fail, so this is only beneficial in the extreme edge case of absolute maximum CPU usage. The benefit here is that this lock up will be considerably shorter than the slowdown that would ensue. 99 times out of 100, you will want to leave this as false, because the slow down should be short and should have no noticable impact on requests to your server.

- `cache_len` refers to the caching of the top and bottom entries of your leaderboards. One of the most common requests to leaderboards is retrieving the top or bottom few entries on the board. To speed up this process, the server will save the top and bottom few entries temporarily. That way, it can just provide that temporary save instead of manually re-calculating the top or bottom few entries. The only downside to this is that when cached data is returned, it may be slightly out of date. Increasing `cache_len` will mean your server has to re-calculate the top or bottom entries less often, but the data can be more out of date. Decreasing `cache_len` will mean the server will have to re-calculate this data more often, but the data will be less out of date. Retrieving the top or bottom few entries is still a fast operation, however, so you do not need to be too concerned with this setting, its effects should be marginal.

!!! info "Reverting to Default Configuration"
    If you need to revert to default configurations, stop the server, delete the json file, and re-start the server. Alternatively, you can just paste in the default provided above.