---
title: 'Clone all repository hosted on Gitlab'
date: 2021-03-08
permalink: /posts/2021/03/clone-all-repository-hosted-on-gitlab/
tags:
- gitlab
- linux
---

## Gitlab APIs
We will use [personal access token](https://docs.gitlab.com/ee/api/README.html#personalproject-access-tokens) with [projects API](https://docs.gitlab.com/ee/api/projects.html#list-all-projects) to get projects info.

## Usage

```shell
for repo in $(curl -H "PRIVATE-TOKEN: <ACCESS-TOKEN>" "http://<HOST-NAME>/api/v4/projects" | jq '.[].ssh_url_to_repo' | tr -d '"'); do git clone $repo; done;
```

There is another tool is `Glab`
-----------------------------------

Thank for reading :blush:.