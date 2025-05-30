---
title: Meine ersten Obsidian Tweaks
date:
  created: 2024-11-29

slug: "meine-ersten-obsidian-tweaks"
tags:
  - Obsidian
  - Blog

authors:
- harry
---
## Änderungen in den Obsidian Settings damit die Attachments in einem Subfolder gespeichert werden

So sieht die Struktur aus ohne Anpassung

![Image Description](../images/20241129-ObsidianTweaks1.png)

<!-- more -->

In den Einstellungen auf folgendes geändert
*Files and Links* -> *Default location for new attachments* -> *In subfolder under current folder*

![Image Description](../images/20241129-ObsidianTweaks2.png)

So sieht das dann im Finder aus

![Image Description](../images/20241129-ObsidianTweaks3.png)
<!-- more -->
## GIT

Siehe https://help.obsidian.md/getting-started/sync-your-notes-across-devices#Git

1. Git Repo auf Github oder Gitlab anlegen
2. `git init` in dem Verzeichnis des Vaults
3. `git remote add origin [URL]`
4. Commit your changes: `git add .` and `git commit -m "Your message"`.
5. Push the changes: `git push origin main`.

### Git Plugin

Siehe https://obsidian.md/plugins?id=obsidian-git#

1. Git Plugin installieren
2. Settings überprüfen
3. *Commit-and-sync*

![Image Description](../images/20241129-ObsidianTweaks4.png)
