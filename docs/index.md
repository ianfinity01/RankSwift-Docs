# INTRODUCTION

## What is RankSwift?
RankSwift is a leaderboard server designed to handle massive amounts of users and requests. This was designed with Roblox in mind, but can be used for anything. Unlike most databases, it stores everything in memory and backs things up to file to make sure it can handle as many requests per second as possible.

## Why Rankswift?
Rankswift can easily store up to 25-100 million entries with tens of thousands of requests per second, making it work for almost any use case.

## Why Not Use Ordered Datastores (Roblox)?
Ordered datastores *can* work for leaderboards in roblox, but they are functionally limited. Because of the limited exposed methods we are given for Roblox Ordered Datastores, it is not possible to retrieve the rank of any specific user without iterating over EVERY user on the board and making hundreds or thousands of API calls. Futhermore, your control over them is limited. You cannot easily save backups, clear out the board, or anything similar. All you can do with them is store key value pairs and retrieve the top few users.

It is a common experience for users to be discouraged by these leaderboards, because all they can see are the best of the best, the top players in the world. RankSwift aims to fix this by allowing players to track their global rank for tangible proof of progress, and even see the players around them so they know what they're up against, giving EVERY player something to work toward.

Roblox Ordered Datastores are also restricted to one game instance -- you cannot easily access another game's Ordered Datastores. These boards can be shared between games all you'd like with no issues. If you so please, you can also make use of the server's APIs to set up your own website displaying leaderboard information completely outside of the game.

!!! warning "Data Storage"
    Do not use RankSwift leaderboard as the primary storage method of crucial player information. Keep your data storage in Roblox Datastores or a MySQL database. This system is just for *displaying* and *aggregating* this information, not storing it. Because of the way it saves things, in the event of a crash, data may roll back slightly. Furthermore, because of how the leaderboard is organized with size caps and minimum values, important player information may be lost if you rely on a RankSwift leaderboard for data storage. Just send current values from your actual data storage medium to this server so that it can be aggregated and displayed.

## How Can I Self-Host the Server?
Once you have purchased access to the leaderboard, you will get all the code for it via a github repository. If you will be hosting it on windows, the executable will be enough. Otherwise, you will need to build the code on your operating system using Rust's Cargo. Don't worry, this process is pretty straight forward and simple. It is strongly suggested that you use a hosting service such as Oracle Cloud or Amazon AWS to run this, since it will need to be running constantly and will use up a gigabyte or two of RAM. For detailed instructions, go to the [Installation](./Server/installation.md) page.

**IF YOU DON'T WANT TO HOST THE SERVER YOURSELF,** you can contact me directly and I can host it for you for a monthly rate. You can reach me at [ianfinity1@gmail.com](mailto:ianfinity1@gmail.com) or discord (inconsistent). My tag is ianfinity.

## How Can I Connect My Roblox Game to a Board?
There is a roblox module available for **free** that will handle all the API calls for you and give you a nice, easy-to-use interface with your board. All you need to do is give it the server's ip, port, and the access key you generate on the board, all of which there are simple steps to get. This module will also cache a lot of values and try to make everything as simple to use as possible while minimizing strain on your leaderboard server. For detailed roblox setup steps, go to the [Roblox Setup](./Roblox/setup.md) page. Get the Module Script [here](https://create.roblox.com/store/asset/120356645529088){:target="_blank"}.

## Can I See RankSwift in Action?
Of course! We have a Roblox demo available [here](./Roblox/demo.md). The demo should help you see what's possible even if you don't plan to use this for Roblox.

## Price
I wanted to ensure that nobody avoids using RankSwift because of its price point, that's why the code is only available for $25 despite the server being multiple thousands of lines of code. If you're a larger development group or build a successful game using my server, it would be greately appreciated if you [donated](https://ko-fi.com/ianfinity#){:target="_blank"}.