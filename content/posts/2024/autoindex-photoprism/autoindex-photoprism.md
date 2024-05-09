---
title: 'Automating Photoprism Indexing'
date: 2024-04-08T12:33:51+02:00
author: ["mightyjens"]
categories: ["linux"]
tags: ["Open Source","Selfhosted"]
categories_weight: 10
comments: true
showToc: false
TocOpen: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

If you’re an avid user of the open-source image library Photoprism, you know how convenient and powerful it can be for managing and organizing your photo collection. However, keeping your library indexed and up to date manually can be a tedious task. Thankfully, with a simple cronjob in Linux, you can automate the indexing process and ensure your Photoprism library stays current without any manual intervention.

## Indexing with Cron
1. Create the shell-script auto_index.sh

``` bash
set -o allexport; source /opt/photoprism/config/.env; set +o allexport
cd /opt/photoprism/bin/
./photoprism index
```
2. Make it executable using chmod -x auto_index.sh
3. Add it to your crontabs using crontabs -e

``` bash
#Run Photoprism indexing every day
@daily /opt/photoprism/photoprism index
```
In this example, the cronjob is scheduled to run every day at 0:00 AM, triggering the Photoprism indexing process by invoking the index command.
