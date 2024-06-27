---
title: 'Paperless NGX: Conversion between UTF8 and SQL_ASCII is not supported'
date: 2024-05-08T11:17:02+02:00
#author: ["mightyjens"]
categories: ["paperless"]
tags: ["Open Source","Selfhosted","Paperless"]
categories_weight: 20
comments: true
showReactions: 1
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

Maybe it will help some of you, but I just had an error updating 2.8.0 on my bare metal Paperless NGX instance.
When trying to migrate the changes, the following error occured:

> Conversion between UTF8 and SQL_ASCII is not supported

In fact, my Postgres database was set to SQL_ASCII instead of UTF-8, so it helped to rebuild the database. 
Make sure to backup/snapshot everything in case something goes wrong, I managed to kill my instance while fiddling around with this :smile:

1. Stop paperless services:

``` shell
systemctl stop paperless-consumer paperless-webserver paperless-scheduler paperless-task-queue
```
2. Create an encoded dump of your existing data and push it into a newly created, UTF-8 encoded database:

``` shell
# Switch to postgres user context
su - postgres

# Create the dump
pg_dump --encoding utf8 paperlessdb -f paperless.sql

# Create the new database (*)
# createdb -E utf8 paperlessdb_new
createdb -T template0 -E utf8 paperlessdb_new

# Restore the dump
psql -f paperless.sql -d paperlessdb_new
```

(*) You might need to create a new database template according to this <a href="https://stackoverflow.com/a/16737776" target="_blank">accepted solution on stack overflow</a> in case you face the error *"New encoding (UTF8) is incompatible with the encoding of the template database (SQL_ASCII)"* when trying to create the new database.

3. Rename the old database and replace it with the new one:

``` sql
-- Switch to postgres interactive terminal 
psql

-- Rename the databases
ALTER DATABASE paperlessdb RENAME TO paperlessdb_old;
ALTER DATABASE paperlessdb_new RENAME TO paperlessdb;

-- Don't forget the permissions
ALTER DATABASE paperlessdb OWNER TO paperless;
```

4. Now we can restart our services and continue updating:

```  shell
systemctl start paperless-consumer paperless-webserver paperless-scheduler paperless-task-queue
```
