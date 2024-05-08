---
title: 'Paperless NGX: Conversion between UTF8 and SQL_ASCII is not supported'
date: 2024-05-08T11:17:02+02:00
categories: "linux"
categories_weight: 20
ShowToc: false
---

Maybe it will help some of you, but I just had an error updating 2.8.0 on my Paperless NGX instance.
When trying to migrate the changes, the following error occured:

***Conversion between UTF8 and SQL_ASCII is not supported***

In fact, my Postgres database was set to SQL_ASCII instead of UTF-8. For me it helped to rebuild the database:
```
systemctl stop paperless-consumer paperless-webserver paperless-scheduler
```
Create an encoded dump of your existing data and push it into a newly created, UTF-8 encoded database:
```
su - postgres

postgres pg_dump --encoding utf8 paperlessdb -f paperless.sql
createdb -E utf8 paperlessdb_new
psql -f paperless.sql -d paperlessdb_new
```
Now rename the old database and replace it with the new one…

```
ALTER DATABASE paperlessdb RENAME TO paperlessdb_old;
ALTER DATABASE paperlessdb_new RENAME TO paperlessdb;

ALTER DATABASE paperlessdb OWNER TO paperless;
```
```
systemctl start paperless-consumer paperless-webserver paperless-scheduler
```
Make sure to backup everything in case something goes wrong.