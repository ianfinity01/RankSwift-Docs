## Getting the Server Files
First and foremost, you need to *have* the server to set it up. You can buy the code for a low one-time price here. 

!!! tip "Hosting"
    It is strongly encouraged that you host this on some form of hosting service such as Oracle Cloud or Amazon AWS. More likely than not, you will want your server running 24/7, and this is the best way to make sure it keeps running and is always accessable. Furthermore, it keeps you from exposing your IP, keeps you from eating up your network bandwidth, and keeps the Server from eating up your RAM or processing power.

    You can completely bypass setting up hosting by paying us to host it for you. This will be a small monthly rate. If you want to do this, contact me at [ianfinity1@gmail.com](mailto:ianfinity1@gmail.com).

## Installation
The leaderboard may recieve optional updates as time goes on adding new features, optimizations, or bug fixes. It is for that reason that we will be cloning the github repository instead of just pasting in and running the code, that way whenever there's an update, you can just run the command `git pull` to get the latest version.

!!! warning "GitHub Access"
    In order to have permissions to access the repository and download the code, you not only need to use the account you provided when buying access, but you must also accept the invitation. When you bought access, you should have recieved an email on the email attached to your github account. This email will say something like `ianfinity01 invited you to Development-Infinity-RBLX/RankSwift-Sever`. You need to go there, press view invitation, and press accept in order to have access.

=== "Using Github Desktop"
    First, you'll need to install github desktop. You can find it [here](https://desktop.github.com/download/). It may require you to also install git, just follow its instructions. Once you have Github Desktop downloaded and installed, log into it with the same Github account you provided when you bought the server. Now click on `File` in the top left, and press `Clone respository...`. Then select the `URL` tab and paste in the following URL: `https://github.com/Development-Infinity-RBLX/RankSwift-Server`. Then, choose the path you want it to clone to under `Local path` and press the blue `Clone` button. If you used the right github account and everything was done properly, all the code should be downloaded into the directory you chose. If there is ever an update for the server code, you should get an option to `Pull origin` in a blue button. Just press that and it should make sure you have the most up to date version of the code.

=== "Using Command Line"
    First, you'll need to install git. If you're on linux, you'll want to use some form of package manager like `apt`, but you should be able to find simple installation instructions for your operating system on [their website](https://git-scm.com/). To check if you have git installed, run the command `git --version`. If you have it installed, it should say something like `git version ...`. Once you have git installed, go to github.

    Now, in order to clone the repository (download the code) you need to provide git with the credentials it needs to verify that you have access to the repository. To do this, we will be generating a **Personal Access Token**. On any device, make sure you're logged into the GitHub account that you provided when you bought the server. On GitHub, click on your profile in the top right, and click on `Settings`. Scroll to the bottom, and on the left, there should be a tab called `Developer settings`. In developer settings, click on `Personal access tokens` then `Tokens (classic)`. In the top right, press `Generate new token`. It is important that you generate a classic token (`Generate new token (classic)`) because the regular Fine-grained tokens don't like to recognize repositories you don't have write access for.
    
    You should be at a page like this:
    ![](../assets/PAT-Page.png)

    Write whatever note you'd like, then select `No expiration`. GitHub will warn you about creating a token with no expiration, however, if you set an expiration, then you will stop being able to recieve updates for your code once the token expires, and you will have to make another token. As long as you keep the token you generate private, this should not cause any issues. Then, check the "repo" box. This will give this token access to view (and clone) the RankSwift repository you got access to. The page should look like this:

    ![](../assets/PAT-Page-Complete.png)

    Scroll to the bottom and press the green `Generate token` button. You will then be given the token and an option to copy it.
    
    ![](../assets/Generated-Token.png)
    
    !!! warning "Token Safety"
        Keep this token private, it gives access to repositories your account has access to.

    Once you have the token, we can return to the command line. The following command will now clone the github repository and all the server code: `git clone https://<your-username>:<your-token>@github.com/Development-Infinity-RBLX/RankSwift-Server.git`

    In my example case, the command I would run would be `git clone https://ianfinity01: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@github.com/Development-Infinity-RBLX/RankSwift-Server.git`.

    The repository should then clone into a folder called `RankSwift-Server` in your current directory. To get updates, go into that folder (typically with the command `cd RankSwift-Server`) and run the command `git pull`.

## Building the Code
Great! Now that you have the code on your computer, it's time for the easy part. All you have to do is install the programming language rust! Luckily, rust makes this very easy for you with their "rustup" commandlet.

=== "Linux or MacOS"
    All you have to do is run `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` in a terminal. You will then be prompted with installation options. Just proceed with the default installation.

=== "Windows"
    Go to the [rust installation page](https://rust-lang.org/tools/install/) and press `DOWNLOAD RUSTUP-INIT.EXE (X64)`. Only choose the 32-BIT version if you know you're on 32-BIT windows, chances are, you aren't. Once the executable has downloaded, run it. A terminal will open up and ask you about installation options. You just need the default installation, so just press enter.

!!! info "Restart your terminal"
    When rust installs, whatever terminals you may have open will not recognize that you just installed it and that there are new commands. You will need a new terminal window open for the next step.

## Starting the Server
This is the easiest part. Now that you have rust and the code, just open up a terminal in the directory of the repository you cloned. Unless you renamed it, the directory will be named `RankSwift-Server`. Then, to start your server, run the command `cargo run --release`. The first time you do this, it will take a while because the code has to install all its dependencies. It will also take some time to start if there was an update to the code (likely because you ran `git pull`).  You'll know that your server is running if you see this:
```
🚀 Rocket has launched from http://0.0.0.0:3895
CLI launched, type 'help' for a list of commands.
>
```

Note that if you close your terminal, it will stop the server and your leaderboards will not be accessable until you start your terminal again. If you need to keep your terminal running in the background on a hosting service, I suggest installing a package like `tmux` to create a "detatched session" for your server. You also may want to set up something to make your server start up when your hosting service starts up incase it crashes at some point. You will have to figure out how to do this for your operating system, and there are usually multiple ways to do it.

!!! warning "Remember how I said this was the easiest part?"
    I lied. Kind of. Your server is now running, but nothing outside your current network can access it. In order to have your server accessable to the outside world (Roblox servers, your websites, etc) you need to do a process called "Port Forwarding". Port forwarding is basically telling your router that when it recieves a request for your server, it should let it in and send it to your server. **You will need to be able to log into your router to do this**. If you are hosting this on a hosting service, they usually have their own tools for port forwarding, which you will need to look up or find. If you're self-hosting this (which you shouldn't be), [here is a good tutorial on port forwarding](https://www.noip.com/support/knowledgebase/general-port-forwarding-guide).

    Unless you changed it in the config, RankSwift will use the port **3895**.

!!! failure "Port forwarding"
    Seriously, port forwarding is a necessary step, make sure to do what is outlined above.

Now that your server is running, it's time to set up your leaderboards! Let's head onto [setup](./setup.md).