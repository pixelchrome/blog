---
title: "Ghost NGINX und SSL"
date:
  created: 2016-02-18

linktitle: "Ghost, Mac OS X und homebrew"
slug: "ghost-nginx-und-ssl"

tags:
- SSL
- NGINX
- IPv6

authors:
- harry
---

Ich hatte mit meinem Setup das Problem, dass man beim Aufruf in einer 'Redirect Loop' hängen blieb, sobald ich SSL enabled habe.
<!-- more -->
Folgende Info hat geholfen:

[Bug: Setting url: to https in config.js causes a redirect loop #2796](https://github.com/TryGhost/Ghost/issues/2796) von [jcones](https://github.com/jcjones)

> On Nginx I added `proxy_set_header X-Forwarded-Proto https;` to my site configuration

Und es stimmt! So sieht nun der Redirect Teil der NGINX Config aus:

```nginx
location ^~ /blog {
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header X-Forwarded-Proto https;
     proxy_set_header Host $http_host;
     proxy_set_header X-NginX-Proxy true;
     proxy_pass http://127.0.0.1:2368;
     proxy_redirect off;
}
```

Die config.js muss folgendermassen aussehen:

```nginx
production: {
     url: 'https://pixelchrome.org/blog',
     mail: {
         transport: 'SMTP',
         options: {
             host: '127.0.0.1',
             service: '23mail'
     }
},
```
