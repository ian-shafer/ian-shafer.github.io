---
layout: post
title: Roll Your Own Database Migrations
---

I once worked at a startup where database migration scripts distributed via email. This is not necessarily a horrible thing -- I don't remember a time where it caused a major issue (especially when compared to the bug that charged customers 10x the purchase price). But, if you want to make your database migrations a little more formal than email, you can read on about how I do this.

By the way, there are many frameworks that manage database migrations for you. I've used some of these frameworks before and they're great. Many of the techniques that I use in my day-to-day development come from things I've seen in other pieces of software -- Ruby on Rails and Drupal are two examples. If you're using one of these frameworks and are curious about how they might work, read on.

The Problem
--------------------------------------------------------------------------------

Over the life of a software project that uses a relational database, the schema will need to change. How should these schema changes be tracked and applied in all environments (dev, qa, prod, etc.)?

The simple answer is that one would want to track the changes just like we track code changes in any SCMS. This gives us the ability to see what the schema was like at any given time, and it gives us the ability to replay schema updates in an isolated environmet.

Also, we'd want the updates applied automatically, or at least with just a few keypresses.

One Solution
--------------------------------------------------------------------------------

When faced with this problem about four years ago (I was building an e-commerce platform for a now defunct startup), I came up with a simple solution that has served me well since then.

The idea is simple:

* Store SQL migration files in a `/sql-migrations` directory
* Store the current database version in the database
* Use a shell script to apply necessary migrations

There are some limitations:

* Migrations are limited to pure SQL -- there is not (easy) access to application code
* I have not implemented a rollback mechanism (I'll talk more about this below)

Show Me the Code
--------------------------------------------------------------------------------

First, your database must have a table like this:

~~~ sql
create table database_versions (
  version varchar(32) not null,
  is_active boolean not null default false,
  creation_date timestamp not null default current_timestamp
);
~~~

Logic to automatically create this table could be added to my migration script below. I have a `db-init` script that creates this table for me.

The `is_active` field is optional. You could infer the active version by assuming that it's the greatest version. I use it because it makes the code simpler and I have not had a problem with it being out of sync.

Now let's look at the SQL migration files. There are two requirements for these files

* The schema version must be available via the file name
* The files must be sortable by name e.g. `sort -V`, or `sort -t. -k 1,1n -k 2,2n, -k 3,3n`, or just make the names lexically ordered

I use file name like `000.000`, `000.001`, `001.000`, etc. because it makes the migration shell script easier to code.

Now for the meat. Here's the script I use to apply the migrations:

~~~ bash
#!/bin/bash
set -e
source $(dirname $0)/../.include

function add-version() {
  psql $PSQL_OPTS -c "update database_versions set is_active = false where is_active = true" $DB_NAME
  psql $PSQL_OPTS -c "insert into database_versions (version, is_active) values ('$1', TRUE)" $DB_NAME
}

curv=$(psql $PSQL_OPTS -Atc "select version from database_versions where is_active = true" $DB_NAME)

for f in $(ls -1 $BASEDIR/migration/\*.sql | sort); do
  filev=$(basename $f .sql)
  if [[ $filev > $curv ]]; then
    psql $PSQL_OPTS -vON_ERROR_STOP= -1f $f $DB_NAME
    # No need to check return code because of the 'set -e'
    add-version $filev
  fi
done
~~~

This script

1. Retrieves the current database version
1. Iterates through the migration files (in order) and applies versions that are greater than the current version

This script is pretty simple. When I first wrote it, I was surprised at how short it was.

This script is for a postgres  databases, but I've used similar versions for MySQL.


Applying the Updates
--------------------------------------------------------------------------------

In development environments, I apply these updates manually. This means that a dev would have to see that a migration file was added and run the migration script by hand. This could be handled a little better using a test at application startup time that does one of the following if the databse is not up-to-date.

* Fails to start
* Automatically applies the migrations
* Prints a warning

In production, I apply the migrations as part of my deploy process.

What About Rollbacks?
--------------------------------------------------------------------------------

Like I said above, this database migration technique does not handle rollbacks. It could. All one would have to do is include a rollback SQL file for each migration file and write a script to rollback to a particular version. In my experience, I've never had a need for this. Maybe I'm just lucky, but I'd like to think that it's more that I've tested and planned properly.
