---
layout: post
title: How to make sure an app is always running in macOS.
description: How to make sure an app is always running in macOS.
summary: How to make sure an app is always running in macOS.
comments: false
tags: [macos, scripts, onedrive, launchd]
---
I have Microsoft OneDrive installed on my MacBook Pro, and I need it to always be running for two reasons.
Firstly, my home and work computers must always be synced to ensure that I have access to the files I'm working on.
Secondly, I use OneDrive as my backup program, so if I accidentally delete a file, I can recover it from OneDrive's recycle bin.

Even if there is an option to start OneDrive on every boot in OneDrive's settings, I have noticed that OneDrive is not running. There are many reasons why OneDrive may have stopped (a system update overwriting the setting or a program crash). To be sure that everything is synced I will show a way to ensure that OneDrive is always running, and if OneDrive crashes it will instantly start up again.   

Now, this solution can be used for any application you need always running, not just OneDrive. To our help we are going to use a built-in program called launchd.

### The Daemon

Launchd is a daemon that manages various types of background jobs on macOS. It is the first program that starts when the computer boots up, and the last that stops when the computer shuts down. Its primary function is to launch processes or jobs, either on-demand or on a scheduled basis. There are two types of services that handle these processes and jobs: daemons and agents. A daemon is a system-wide service for jobs and processes that are related to the operating system itself, and cannot interact with a specific user or a user's home directory. In contrast, an agent is a background process that runs on behalf of a specific user.
Since we need to monitor and restart OneDrive, we will use LaunchAgent, which will monitor and restart the application if it stops running.

To avoid any conflicts between the original service (installed by Microsoft in `~/Library/Preferences/com.microsoft.OneDrive.plist`) and our new service, we should first uncheck the setting "Open at login" in OneDrives settings.

We can also check the macOS settings for Login Items, so OneDrive is not in the list of apps that starts when we login.

### The Information Property List
So, now we are ready to initiate a new job for the LaunchAgent. The first step is to create a new file called a plist (Information Property List) and name it `com.<NAME>.keep-onedrive-alive.plist`.

You can swap out `<NAME>` with any string. Apple's plist files are named `com.apple.servce-name.plist`. Microsofts plist files are called `com.microsoft.service-name.plist`. This is called a reverse domain naming system. Since I am not a company, I will just put my name there instead. You can put your name or your company name or just any word that makes you remember what this service does. I will name my file `com.jorgen.keep-onedrive-alive.plist`.

Plists are the configuration files that specify what kind of service the LaunchAgent should perform. In our case we want the LaunchAgent to always monitor OneDrive and restart the program if it stops. Copy and paste this XML code into your file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>KeepAlive</key>
    <true/>
    <key>Label</key>
    <string>com.jorgen.keep-onedrive-alive</string>
    <key>Program</key>
    <string>/Applications/OneDrive.app/Contents/MacOS/OneDrive</string>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

If we look at the key-value pairs in this file, we can see that `KeepAlive` is set to `true`. We can see the path to the binaries that should be started or restarted by the agent. If you want to start another program than OneDrive, just replace the path to the program. You may have to right-click on a program in the Apps folder and choose "Show Package Contents" to find the binary. In this case, the binary is inside the folder `/Contents/MacOS/`.

Finally we can see that the value of `RunAtLoad` is set to `true`. This means that the program will always be started when the computer starts or restarts.

Then save the file in your user library. This is the path to where my file is located:

```shell
 ~/Library/LaunchAgents/com.jorgen.keep-onedrive-alive.plist
```
The ~ sign represents my home folder. If you like to drag-and-drop the file in Finder instead, click the Go menu and hold down the alt key to see the Library option in the list of locations. Then drag the file inside the folder LaunchAgents.

Launchctl is a small application that interfaces with launchd to manage and inspect daemons and agents. To start the service in the terminal we will use launchctl to run

```shell
launchctl load ~/Library/LaunchAgents/com.jorgen.keep-onedrive-alive.plist
```
This command will then

1. Launch OneDrive

2. Re-Launch OneDrive if it is quit or crashes. Basically anytime it stops running, it will automatically start again.

To test that the service is working, just quit OneDrive and watch how the it instantly restarts.
This also means that you can no longer quit OneDrive. If you need to temporarily stop this service, you can run

```shell
launchctl unload ~/Library/LaunchAgents/com.jorgen.keep-onedrive-alive.plist
```

### Resources
If you want to know more about launchd, daemons or launchctl, you can use the man command in the terminal. For instance

```Shell
man launchd
```
Apple also has an [overview of launchd on its help pages](https://support.apple.com/guide/terminal/apdc6c1077b-5d5d-4d35-9c19-60f2397b2369/mac).

This guide is based on this [technical solution posted at stackexchange.com](https://apple.stackexchange.com/a/333796).
