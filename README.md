### Commands to create a backup of the Heroku app "test-postgres-backup"

**create a backup of test-postgres-backup**

 `heroku pg:backups capture --app test-postgres-backup`

**get a url of the backup (this is temporary and will expire after 10min)**

`heroku pg:backups public-url b001 --app test-postgres-backup`

** restore backup **

`heroku pg:backups restore 'https://xfrtu.s3.amazonaws.com/example' DATABASE -a test-postgres-backup`

** Database restored. Cool! **

### Scheduling backups

** schedule a backup for 3am GMT **

`heroku pg:backups schedule DATABASE_URL --at '19:00 America/Los_Angeles' --app test-postgres-backup`

** not sure what DATABASE_URL is being used for here? Works without this i.e.**

`heroku pg:backups schedule --at '19:00 America/Los_Angeles' --app test-postgres-backup`

** View the scheduled backups **

`heroku pg:backups schedules --app test-postgres-backup`

** Deleting backups **

`heroku pg:backups delete b101 --app test-postgres-backup`

** How to delete multiple backups? Can we set old backups to delete automatically? **

### Notes

The types of backups available for Postgres are divided into physical and logical backups. Physical backups may be snapshots of the file-system, a binary copy of the database cluster files or a replicated external system. Logical backups are a SQL-like dump of the schema and data of certain objects within the database. Physical backups can be less expensive computationally but limited in how they may be restored. Logical backups are much more flexible, but can be very slow and require substantial computational resources during a backup of a large db.

Backup interval - How often should we backup?
Backup retention - How long should we keep a backup?
What are the performance impacts?

Time spent making a back up is not important
Time it takes to restore a backup is critical!
Consider multi-stage solutions

**Options in Postgres**

Logical backups
- pg_dump
Physical backups
- Filesystem snapshots
- pg_basebackup
- "Manual" base backups

**Logical backups:** Backups at the SQL level of the DB. What is typically referred to as a dump, hence the name pg_dump

Physical backups: Some sort of local file system backup, well beneath level of SQL.

** More on pg_dump **

Before 8.0 pg_dump was the only backup postgres offered.
When you pg_dump you get a big SQL script + data. The script will have the schemas: create table, create column, create index, create function... and then it loads data with a normal copy command. Same as if you were loading in a csv file. They are a regular Postgres client - you could have a node, python.. script that does the same thing. This is not necessarily great for performance.

**Practical stuff**

pg_dump supports multiple output formats. Always use custom format (-Fc)
For backups, always dump the whole database
You can filter when you restore e.g. only restore one table.

**Using pg_restore**

- Reads "custom" format dumps
- Regular connection
- Supports full databse restore: "recover from backups"
- Partial database restore: "restore single/multiple table"

In summary, pg_restore: opens a standard Postgres connection, runs all the standard commands on the data to populate the database.

Can be very slow for large data sets.

**Improving restore time**
- use -1 flag: Full restore as a single transaction
....

### pg_dumpall
Important: pg_dumpall -g

pg_dump backs up a single database. There are objects in postgres which are not inside databases: Users, groups, tablespaces
pg_dumpall -g dumps only the global objects
pg_dumpall is all the databases + global in plain text. -g say only take global objects

**Backup strategy**
- You definitely want online physical backups
- You almost certainly want PITR (point in time recovery)
- You want pg_dump (if db is less than terabyte)

# Physical backups
"All Heroku Postgres databases are protected through continuous physical backups."
**Need to look into this**


### Useful links:
- https://devcenter.heroku.com/articles/heroku-postgres-backups - Heroku PGBackups
- https://devcenter.heroku.com/articles/heroku-postgres-data-safety-and-continuous-protection#the-different-types-of-backups - Heroku Postgres Data Safety and Continuous Protection
- https://www.youtube.com/watch?v=FyR3TD11hlc - great talk by one of Postgres contributors
- https://blog.heroku.com/archives/2015/3/11/pgbackups-levels-up - Heroku blog piece on PGBackups
