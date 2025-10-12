Stress testing is an essential part of setting up your server -- it gives you important information and helps you decide how to configure your server.

## Running the Test
You'll want to do this on an empty board that is not in use. You can create a new board with the command `new_board <board_name>`. Then, set the current board to that board with the `board <board_name>` command.

Running the stress test can take some time, and will be CPU and RAM intensive, so be prepared. To get started, just run the command `stress_test <board_size>`. This will run some worst-case tests at maximum utilization on a board with the specified board size. You want to find the largest board_size that will give you the results you want. It is suggested that you start testing with a board size of about 5 or 10 million. Usually you will want to go up from there in size, but not by much. Then, run the command and wait for the results to come in.

## Understanding the Results
You will receive information on how long operations such as update, retrieve around, and rank retrieval will take. The result is in milliseconds, or 1000ths of a second. These operations should be really fast (< 0.01 milliseconds or < 0.00001 seconds) meaning you can do tens of thousands of operations a second. However, be aware, the length returned does NOT account for things like parsing request data. This only measures how long the actual operation will take, nothing about managing the HTTP requests. So, expect operations to take longer than this, but it shouldn't be by too much. Be aware that even the server's process rate often isn't the bottleneck, if your network can't handle receiving 10,000 requests in a second, then neither can the Server.

The time returned on the top and bottom entries should not be treated too seriously, but instead as a basic litmus test. Because of caching, this test could only be performed once instead of thousands of time, so the time returned won't be too accurate. Usually it will take about the same amount of time as the retrieve around requests.

Note that things will slow temporarily before writing to file. The time provided is a maximum, it should not take as long as it says, but be aware that it is possible that it will take that long. Requests should not slow down by much unless you are really maxing out your CPU. This is really the only part of the saving process that you need to worry about, because everything else will happen in tandem and have no real effects if its longer or shorter.

Reading from file will always be the part that takes the longest, but that's ok! Your server will only read from file whenever your server starts up, and usually, your server won't be starting up much because it won't be crashing much. Plus, when it crashes, people will already be expecting your server to be down for a bit, so a little longer shouldn't be a big deal.

If you want to know how these lengths will scale with board size, if you double the number of entries, the read and write times will also double, but the update, rank retrieval, retrieve around, and other similar operations will only lengthen slightly. They may lengthen more depending on your RAM Cache usage, something you don't really need to understand unless you're a programmer. Usually, you don't need to be worried about your update and retrieval operations getting longer when increasing your board size, you only need to be worried about RAM usage (as described below) and read/write times. It's up to YOU to decide what you're ok with.

??? info "Information for Programmers"
    For those of you who know a bit more programming, the update and retrieval operations are `O(log(n))` while read and write operations are `O(n)`. This is because the board is internally represented by a binary tree, giving the `log(n)` complexity, but of course, read and write can never be less than `O(n)`.

!!! warning "Just Because You Can Doesn't Mean You Should"
    Almost every limit imposed by the Server on things like number of requests, size of requests, etc, must be done by YOU. The server does not prevent you from, for example, retrieving the top 100000 entries of a leaderboard with an API request. Doing this could temporarily lock up your leaderboard and slow your server. Make sure to impose rate limits and parameter limits yourself so that you don't over-exert your server. YOU are responsible for what happens to your server.

## RAM Considerations
Luckily, one thing that **doesn't** vary between devices is the RAM usage of a board. A board will take up 0.13 Gigabytes of RAM per million entries, meaning a board with 100 million entries would take up 13 gigabytes of ram, and a board with 10 million entries would take 1.3 gigabytes of ram. While saving your boards to file, unless you have the `save_lock` configuration set to `true`, then the server will make a temporary copy of a part of the data, which will momentarily consume extra ram. This should never consume more than an additional 0.06 Gigabytes of RAM per million entries, and it will only happen with one board at a time. Let's look at an example. Say that you had 3 boards, with 10 million entries, 5 million entries, and 30 million entries. The regular RAM consumption would be `1.3 + 0.65 + 3.9 = 5.85` gigabytes of RAM, but during a save, the RAM consumption could go up to `5.85 + 30*0.06 = 7.65.` It shouldn't go this high, but be prepared for the possibility. 

!!! warning "Using Too Much RAM"
    Using too much RAM can be dangerous, it can cause things to slow down, or even completely crash without saving. Always make sure that you have a bit of a cushion between the amount of RAM you think you may use in the worst case and the amount of RAM you have available in total.
!!! failure "Never Use Too Much RAM"
    Never set your board up so that it could take more RAM than you have available. Utilize the size cap feature to minimize RAM usage as described in [the setup page](./setup.md).

Now that you know what your server can handle, let's see how to interact with your server using it's [API](./API.md).