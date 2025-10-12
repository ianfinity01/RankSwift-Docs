There's a lot of helpful commands on your server to help you manage your leaderboards. Here's a complete list of all the commands you have access to.
```
Commands:
help:                               This.

board:                              Outputs the current board. Board mut be set first using the board <board_name> command.
board <board_name>:                 Sets the current board.
boards:                             Get a list of all leaderboards.
new_board <name>:                   Create a new board with the given name.
del_board:                          Delete the current board along with all associated information.

keys:                               List all API Keys on the current board.
all_keys:                           List all API Keys on all boards.
new_key <api_key> <write y/n>:      Creates a new API Key on the current board with specified permissions.
del_key <api_key>:                  Removes the specified API Key from the current board.
set_write <api_key> <y/n>:          Sets whether or not a specific API Key on the current board has write permissions.

get <user_id>:                      Gets the number of points the specified user has on the current board.
rank <user_id>:                     Gets the rank of the specified user in the leaderboard.
at_rank <rank>:                     Gets the entry of the leaderboard at the specified rank.
update <user_id> <points>:          Updates the specified user's points on the current board.
remove <user_id>:                   Removes a specific user from the leaderboard.

top <count>:                        Gets the top <count> users in the leaderboard.
bottom <count>:                     Gets the bottom <count> users in the leaderboard.
after <user_id> <count>:            Gets the <count> entries after the given user in the board.
before <user_id> <count>:           Gets the <count> before after the given user in the board.
around <user_id> <before> <after>:  Gets the entries around the given user.
range <start> <end>:                Gets the entries with ranks between <start> and <end>

size:                               Returns the number of entries in the current leaderboard
clear:                              Entirely clears the current leaderboard, erasing all data

populate <count>:                   Fill the current board with <count> dummy entries, good for testing scalability. Overwrites ALL existing data.
stress_test <board_size>:           Outputs the number of operations that can be made in a second on a dummy board with specified size.

cap:                                Get the size cap of the current leaderboard.
cap <size>:                         Set the size cap of the current leaderboard. Set to -1 to remove cap.
cap_trim:                           Trims off elements from the end of the current leaderboard until it's size is under the cap.

cutoff:                             Get the value cutoff of the current leaderboard.
cutoff <size>:                      Set the value cutoff of the current leaderboard.
rem_cutoff:                         Removes the value cutoff of the curent leaderboard.
cutoff_trim:                        Trims off elements from the end of the current leaderboard until it's size is under the cap.

save:                               Saves all boards to file.
Ctrl+C:                             Save all boards, stop the program, and shut down the server.
```

Most of these commands are simple and do exactly what they say, there are just a couple notes to make.

- The commands `cutoff <size>`, `cutoff_trim`, `cap <size>`, `cap_trim`, `stress_test <board_size>`, `populate <count>`, and `clear` are all administrative commands that may take a while to execute on larger boards. During those commands, requests may slow due to the increase CPU load, and the board the command is used on may lock up temporarily while the command executes, causing incoming requests to that board to stall or possibly even fail.
- The commands `stress_test <board_size>` and `populate <count>` are really only for testing purposes and should not be used on an active board. They are helpful if you want to mess around with the board/system and figure out how it works. For more detailed information on the `stress_test <board_size>` command, visit the [stress testing page](./stress-testing.md).
- The `save` command is not something you need to run manually. Boards will automatically save themselves on an interval based on the server [configuration](./configuration.md). Even if you shut down the server with `Ctrl+C`, it should save itself properly in the process. It is only really useful when you want to manually save the boards to export a backup of a board or something similar.

Alright, now let's move onto seeing what kind of load your server can handle with the stress test command. Onto the [stress testing page](./stress-testing.md)!