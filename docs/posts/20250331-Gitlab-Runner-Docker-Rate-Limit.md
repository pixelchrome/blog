---
title: "Gitlab-Runner Docker Rate Limit - use DOCKER_AUTH_CONFIG now!"
date:
  created: 2025-03-31

linktitle: "Gitlab-Runner Docker Rate Limit - use DOCKER_AUTH_CONFIG now!"
slug: "gitlab-runner-and-the-docker-rate-limit"

tags:
- GitLab
- Docker

authors:
  - harry
---
## Docker is changing the rate limits starting 1st April 2025

See [the announcement](https://www.docker.com/blog/revisiting-docker-hub-policies-prioritizing-developer-experience/)

> - **Updated Pull Rate Limits for Free & Unauthenticated Users** – To ensure a reliable and seamless experience for all users, we are updating authenticated and free pull limits:
    - **Unauthenticated users**: Limited to 10 pulls per hour (as announced previously)
    - **Free authenticated users**: Increased to 100 pulls per hour (up from 40 pulls / hour)

<!-- more -->
The rate limit for *Unauthenticated Users* might be a problem for CI/CD pipelines. Therefore you might to use a  *Free authenticated users*. Here is an example how this can work for a GitLab-Runner.
### 1. Create a personal Token -> docker.com
	- Go to *Account Settings*
	- Go to *Personal access tokens*
	- Create a *Generate new Token*
		- I've generated a *Public Repo read-only* Token
### 2. Generate the base64 encoded Key with the username and the new token

```shell
# The use of printf (as opposed to echo) prevents encoding a newline in the password.
printf "username:new_token" | openssl base64 -A

# Example output to copy
dXNlcm5hbWU6bmV3X3Rva2Vu#
```

### 3. Configure the gitlab-runner `gitlab-runner/config/config.toml`

```toml
[[runners]]
	environment = ["DOCKER_AUTH_CONFIG={\"auths\":{\"https://index.docke
r.io/v1/\":{\"auth\":\"dXNlcm5hbWU6bmV3X3Rva2Vu#\"}}}"]
```

In my case:
```toml
[[runners]]
	environment = ["GIT_CONFIG_COUNT=1", "GIT_CONFIG_KEY_0=safe.directory", "GIT_CONFIG_VALUE_0=*", "DOCKER_AUTH_CONFIG={\"auths\":{\"https://index.docke
r.io/v1/\":{\"auth\":\"dXNlcm5hbWU6bmV3X3Rva2Vu#\"}}}"]
```

### 4. Restart the runner
### 5. Test: re-run a job

Look for `Authenticating with credentials from $DOCKER_AUTH_CONFIG`

```sh
...
Running with gitlab-runner 16.5.0 (853330f9)

on docker-gitlab-runner bKPaazsy, system ID: r_CqZoZTdn7dJr

Preparing the "docker" executor00:03

Using Docker executor with image hugomods/hugo:exts-non-root ...

Authenticating with credentials from $DOCKER_AUTH_CONFIG

Pulling docker image hugomods/hugo:exts-non-root ...

Using docker image sha256:50f9f67f97a199e0bff8f908eb855249c58508cb6247545c784b85d3e7958768 for hugomods/hugo:exts-non-root with digest hugomods/hugo@sha256:6db28d013944a8fa0916fa5d61abec5d71203993a1907813ae77890133bdbab6
...
```

[Doc](https://docs.gitlab.com/ci/docker/using_docker_images/)
