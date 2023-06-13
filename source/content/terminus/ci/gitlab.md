---
title: Terminus Guide - CI Specific - GitLab
subtitle: Authenticating Terminus in a GitLab Pipline
description: How to authenticate terminus properly in a CI pipline that avoids errors from authenticating too many times.
permalink: docs/terminus/ci/gitlab
contributors: [stovak]
terminuspage: true
type: terminuspage
layout: terminuspage
tags: [reference, cli, local, terminus, workflow]
contenttype: [guide]
innav: [false]
categories: [automate]
cms: [drupal, wordpress]
audience: [development]
product: [terminus]
integration: [--]
reviewed: "2023-06-08"
---

Here's a `.gitlab-ci.yml` file which accomplishes authentication of terminus in a CI Environment. Please ensure that you have defined `TERMINUS_TOKEN` in GitLab's CI/CD Environment Variables.

```yaml
image: ubuntu:latest

before_script:
  - apt-get update -yq
  - apt-get install -y php curl perl sudo git

stages:
  - build

cache:
  paths:
    - ~/.terminus

install_terminus:
  stage: build
  script:
    - export TERMINUS_RELEASE=$(curl --silent "https://api.github.com/repos/pantheon-systems/terminus/releases/latest" | perl -nle'print $& while m#"tag_name": "\K[^"]*#g')
    - mkdir ~/terminus && cd ~/terminus
    - echo "Installing Terminus v$TERMINUS_RELEASE"
    - curl -L https://github.com/pantheon-systems/terminus/releases/download/$TERMINUS_RELEASE/terminus.phar --output terminus
    - chmod +x terminus
    - sudo ln -s ~/terminus/terminus /usr/local/bin/terminus
    - export TERMINUS_TOKEN=$TERMINUS_TOKEN
    - terminus auth:login || terminus auth:login --machine-token="${TERMINUS_TOKEN}"
    - terminus auth:whoami
```

This pipeline does the following:

1. Uses the `ubuntu:latest` Docker image.
2. Updates the system and installs necessary tools like PHP, curl, perl, sudo, and git before the script stages.
3. Defines a cache for the `$HOME/.terminus` directory. The pipeline system will save and restore the cache for subsequent runs.
4. Determines the latest release of Terminus from the GitHub API and stores it in the `TERMINUS_RELEASE` variable.
5. Creates a directory for Terminus, downloads it into that directory, makes it executable, and then creates a symbolic link to it in `/usr/local/bin` so that you can run it from anywhere.
6. Exports the `TERMINUS_TOKEN` environment variable (assuming that you've already set it in your pipeline settings) and uses it to authenticate Terminus.
7. Checks that Terminus is authenticated with `terminus auth:whoami`.

In this script, `${TERMINUS_TOKEN` needs to be replaced with the machine token provided by Terminus, and should be added to your variables in the GitLab CI/CD settings.