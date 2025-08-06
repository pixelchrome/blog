---
title: NGIX Rewrite URL

date:
  created: 2025-05-31

authors:
  - harry

slug: "nginx-rewrite-url"

tags:
  - NGINX

draft: false
---

# NGIX Rewrite URL

I've updated my blog (again). Now I am using Material for MkDocs. With my old setup, I had a misconfiguration that caused the URL to be set to `/blog/posts/<article>`. The new URL is just `/blog/<article>`.

I want to fix that by using NGINX rewrite rules. Here's how I did it:

<!-- more -->

```nginx hl_lines="5"
location / {
    try_files $uri $uri/ =404 /index.php?$args;
    set_real_ip_from 172.22.0.1;
    proxy_set_header X-Forwarded-For $remote_addr;
    rewrite ^/blog/posts/(.*)$ /blog/$1 permanent;
}
```

Check the configuration file for errors with `nginx -t` and restart the server.
