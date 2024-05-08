---
title: 'Paperless NGX: Conversion between UTF8 and SQL_ASCII is not supported'
date: 2024-05-08T11:17:02+02:00
author: ["mightyjens"]
categories: ["paperless"]
tags: ["Open Source","Selfhosted","Paperless"]
categories_weight: 20
comments: false
showToc: false
TocOpen: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

Maybe it will help some of you, but I just had an error updating 2.8.0 on my Paperless NGX instance.
When trying to migrate the changes, the following error occured:

***Conversion between UTF8 and SQL_ASCII is not supported***

In fact, my Postgres database was set to SQL_ASCII instead of UTF-8. For me it helped to rebuild the database:
``` shell
systemctl stop paperless-consumer paperless-webserver paperless-scheduler
```
Create an encoded dump of your existing data and push it into a newly created, UTF-8 encoded database:
``` shell
su - postgres

postgres pg_dump --encoding utf8 paperlessdb -f paperless.sql
createdb -E utf8 paperlessdb_new
psql -f paperless.sql -d paperlessdb_new
```
Now rename the old database and replace it with the new one…

``` sql
ALTER DATABASE paperlessdb RENAME TO paperlessdb_old;
ALTER DATABASE paperlessdb_new RENAME TO paperlessdb;

ALTER DATABASE paperlessdb OWNER TO paperless;
```
Now we can restart our services
```  shell
systemctl start paperless-consumer paperless-webserver paperless-scheduler
```
Make sure to backup everything in case something goes wrong.