---
title: "macOS Time Machine Backups and SMB"
date:
  created: 2017-09-29

linktitle: "macOS Time Machine and SMB"
slug: "macos-time-machine-backups-and-smb"

description: "MacOS Time Machine and SMB - How to fix OSStatus Error 17 and how to authenticate."

tags:
- macOS
- Apple
- Server

authors:
- harry
---
## Time Machine doesn't work on Network Shares after the upgrade

After I've updated my Mac Pro to macOS High Sierra, Time Machine Backups stopped working.

## Shared with SMB?

The backups are on a Mac Mini and I've found out that it is necessary to share the folder via SMB.

But that checkmark has already been set. Therefore I've tried to reconnect the Mac Pro to the Backup destination.

<!-- more -->

## OSStatus Error 17

And I've received Error 'OSStatus Error 17'. I wasn't able to find out a description what this means, but looks like that there is an authentication problem.

![Image Description](../images/20170929-TimeMachine_SMB_Error.png)

I used `Username` and `Password`. **That doesn't work here with SMB!**

## Fixed!

The solution is to use `Domain\Username` and `Password`. The Domain is simply the Servername in this case.

![Image Description](../images/20170929-TimeMachine_DomainUsername.png)

Time Machine is now working just like before...

![Image Description](../images/20170929-TimeMachine_Running.png)
