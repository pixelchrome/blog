---
title: "IOS10 / macOS Sierra kann keine Mails verschicken?"
date:
  created: 2016-09-15

linktitle: "macOS Sierra und IOS 10 benötigen IPv6 um Mails zu verschicken?"
slug: "ios10-macos-sierra-kann-keine-mails-verschicken"

description: "Wenn ein Mail Server auch per IPv6 erreichbar ist, aber keine Mails über IPv6 annimmt, kann es zu Problemen mit macOS Sierra und IOS 10 kommen"

tags:
- macOS
- iPhone
- IPv6
- Mail

authors:
- harry
---
Ich habe heute den Update auf IOS 10 bzw. macOS Sierra durchgeführt. Was ziemlich schnell auffiel, es können keine Mails mehr verschickt werden!

Naja... offensichtlich versucht das Mail Programm nun IPv6 zu bevorzugen, macht aber keinen Fallback auf IPv4.

Das ist mir aufgefallen als ich die Logs meines Mail-Servers angesehen habe. IMAP Connection kommt zustande (eben per IPv6), bei SMTP sah ich gar keine Einträge.

Hmm...

<!-- more -->

```sh
# netstat -a|grep -i listen|grep smtp
tcp4       0      0 *.smtps                *.*                    LISTEN
tcp4       0      0 *.smtp                 *.*                    LISTEN
```

Wie man sieht hört mein SMTP Server nur auf IPv4 :-(

```sh
# grep inet_protocols main.cf
inet_protocols = ipv4
```

Tatsächlich!

Also `inet_protocols = all`, Postfix neu starten, Fertig!
