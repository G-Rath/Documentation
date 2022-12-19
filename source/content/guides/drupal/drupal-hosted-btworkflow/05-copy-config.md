---
title: Migrate a Site Created With the Pantheon Dashboard to Drupal + Build Tools
subtitle: Copy Existing Configuration
description: 
cms: "Drupal"
tags: [code, launch, migrate, site, updates, D8, D9]
contributors: [wordsmither]
layout: guide
permalink: docs/guides/drupal-hosted-btworkflow/copy-config
anchorid: copy-config
editpath: drupal/drupal-hosted-btworkflow/05-copy-config.md
reviewed: "2022-12-12"
contenttype: [guide]
categories: [migrate, git]
newcms: [drupal]
audience: [development]
product: [terminus]
integration: [--]
---

Copy any existing configuration from the source site and update the source path as needed to match your configuration folder:

```bash{promptUser: user}
rsync -avz $SOURCE/config/ $DESTINATION/config/ --delete --delete-after
# From $DESTINATION:
git add config -A
git commit -m "Pull in configuration from source site"
```

It is possible that the Drupal site might have relocated the configuration path to a different location. You can find where your configuration files are via:

```bash{promptUser: user}
terminus drush $SOURCE_SITE_NAME.dev -- status --fields=config-sync
```

In some cases no files are copied through this step. This is not cause for concern.