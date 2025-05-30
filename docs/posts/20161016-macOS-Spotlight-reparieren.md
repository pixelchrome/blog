---
title: "macOS Spotlight kaputt?"
date:
  created: 2016-10-26

linktitle: "macOS Spotlight reparieren"
slug: "macos-spotlight-kaputt"

description: "Mac OS X Spotlight zeigt keine lokalen Suchergebnisse? mdutil gibt nur Fehlermeldungen aus? Hier die Lösung."

tags:
- macOS

authors:
- harry
---
## Keine lokalen Ergebnisse

Gestern hat Spotlight beim suchen nur Ergebnisse in 'Vorgeschlagene Website' angezeigt. Lokale Ergebnisse gab es nicht.

Im Terminal den Status abgefragt

```sh
# sudo mdutil -s /
/:
    Error: unexpected indexing state.  kMDConfigSearchLevelTransitioning
```

Hmm...

<!-- more -->

## Have you tried turning it off and on again?

![Image Description](../images/20161026-Have-you-tried-turning-it-off-and-on-again.png)

```sh
# sudo mdutil -i off /
/:
Error: Index is already changing state.  Please try again in a moment.
```

Das funktioniert also auch nicht. Dann hilft nur den Daemon komplett neu zu starten.

```sh
# sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
# sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
```

Und siehe da, wenn man den Status erneut abfrägt

```sh
# sudo mdutil -s /
/:
    Indexing enabled.
```
