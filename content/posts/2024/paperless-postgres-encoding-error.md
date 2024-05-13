---
title: 'Paperless NGX: Conversion between UTF8 and SQL_ASCII is not supported'
date: 2024-05-08T11:17:02+02:00
author: ["mightyjens"]
categories: ["paperless"]
tags: ["Open Source","Selfhosted","Paperless"]
categories_weight: 20
comments: true
showToc: false
TocOpen: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
editPost:
    URL: "https://github.com/mightyjens/mightyjens.github.io/tree/main/content"
    Text: "Suggest Changes" 
    appendFilePath: true
---

Maybe it will help some of you, but I just had an error updating 2.8.0 on my Paperless NGX instance.
When trying to migrate the changes, the following error occured:

> Conversion between UTF8 and SQL_ASCII is not supported

In fact, my Postgres database was set to SQL_ASCII instead of UTF-8, so it helped to rebuild the database. 
Make sure to backup/snapshot everything in case something goes wrong, I managed to kill my instance while fiddling around with this :smile:

1. Stop paperless services:

``` shell
systemctl stop paperless-consumer paperless-webserver paperless-scheduler
```
2. Create an encoded dump of your existing data and push it into a newly created, UTF-8 encoded database:

``` shell
su - postgres

postgres pg_dump --encoding utf8 paperlessdb -f paperless.sql
createdb -E utf8 paperlessdb_new
psql -f paperless.sql -d paperlessdb_new
```

(!) You might need to create a new database template according to this accepted solution on <a href="https://stackoverflow.com/questions/16736891/pgerror-error-new-encoding-utf8-is-incompatible" target="_blank">stack overflow</a> if you face the error *"Use the same encoding as in the template database, or use template0 as template"*.

3. Rename the old database and replace it with the new one:

``` sql
ALTER DATABASE paperlessdb RENAME TO paperlessdb_old;
ALTER DATABASE paperlessdb_new RENAME TO paperlessdb;

-- Don't forget the permissions
ALTER DATABASE paperlessdb OWNER TO paperless;
```

4. Now we can restart our services and continue updating:

```  shell
systemctl start paperless-consumer paperless-webserver paperless-scheduler
```
