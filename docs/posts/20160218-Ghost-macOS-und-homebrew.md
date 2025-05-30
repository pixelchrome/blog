---
title: "Ghost, Mac OS X und homebrew"
date:
  created: 2016-02-18

linktitle: "Ghost, Mac OS X und homebrew"
slug: "ghost-mac-os-x-und-homebrew"

tags:
- Ghost
- macOS
- Blog

authors:
- harry

description: "Wie installiere ich ghost unter Mac OS X mit homebrew?"

image: attachments/images/logos/nodejs-new-pantone-black.png
---

## Wie installiere ich Ghost unter Mac OS X?

## Ganz einfach! Oder?

> Mit brew nodejs installieren und dann mit `npm install` & `npm start` starten.

Im Prinzip ja, aber... wenn man nodejs neu installieren muss, dann wird ein `brew install nodejs` die neueste Version installieren. Die ist aber leider nicht supportet von [Ghost](http://support.ghost.org/supported-node-versions/).

<!-- more -->

![Image Description](../images/20160218-homebrew_nodejs.png)

Also dann eben die LTS Version von nodejs installieren? `brew install homebrew/versions/node4-lts`

![Image Description](../images/20160218-nodejs-lts.png)

Die Version ist leider auch zu neu :( . Zumindest für Ghost 0.7.6.

![Image Description](../images/20160218-homebrew_node4-lts.png)

## Wie funktioniert es dann?

Doch wieder auf die bewährte 0.12.x Version zurückgreifen.
```sh
$ brew install homebrew/versions/node012
```

Ghost entpacken und mit `npm start` starten.

![Image Description](../images/20160218-running_ghost.png)
