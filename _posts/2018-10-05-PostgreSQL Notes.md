---
layout: post
title: PostgreSQL Notes
category: 
- Database
tags: 
- PostgreSQL
---

### 1. Set port number

You can make multi servers listen to different ports so that they can exist on the same machine -- Just set port numbers in file `/etc/postgresql/<version>/main/postgresql.conf`

### 2. Default port number 

If you install multi versions of PostgreSQL using Linux package manager, their configuration files will be under `/etc/postgresql/`, in sub-folders named by the versions, that is,  `/etc/postgresql/<version>/`. And the default port numbers of different versions will be distinct.

For example, I installed version 10 first, then 9.6, then 9.4. Their default port numbers are 5432, 5433 and 5434. The port number is shown in file `/etc/postgresql/<version>/main/postgresql.conf` .

### 3. Starting mode 

Starting mode determines how the database server can be started.

There are 3 modes:

`auto`  -- automatically start the cluster when starting your machine

`manual` -- manual startup with pg_ctlcluster/postgresql@.service only

`disabled` -- refuse to start cluster

The mode can be set in file `/etc/postgresql/<version>/main/start.conf`

### 4. Client and server's versions 

psql is the standard command line client of PostgreSQL. So the ideal case is to use same version of psql and PostgreSQL server. But that is OK if client and server have different versions.

### 5. Specify version of PostgreSQL when starting a server 

These ways do not work on my PC:

```
(1) sudo service postgresql start <version>

(2) /etc/init.d/postgresql start <version>
```

Only way that works for me is :

```
sudo /usr/bin/pg_ctlcluster <version> main start
```
### 6. Specify a server to connect to

When you have multi servers running, you can connect one of them by specifying the server's port.

```shell
sudo -u postgres psql -p <port> postgres
```

or

```shell
psql -U <username> -p <port>
```

### 7. Set default schema

One way is to set search_path in psql command line, which **only works for current session**.

```sql
SET search_path = new_schema;
```

Another way is to change in **postgresql.conf** file like following:

```python
#---------------------------------------------------------------------------
# CLIENT CONNECTION DEFAULTS
#---------------------------------------------------------------------------
 
# - Statement Behavior -
 
#search_path = '"$user",public'		# schema names
search_path = '"$user",new_schema'	# NEW SCHEMA HERE
#default_tablespace = ''		# a tablespace name, '' uses
					# the default
#check_function_bodies = on
#default_transaction_isolation = 'read committed'
#default_transaction_read_only = off
```

Set search_path in **postgresql.conf** will make it permanent.

Then you need to restart PostgreSQL cluster.



### 8. SET only works for current session, not permanent

```
 SET only affects the value used by the current session.
```

### 9. View schema privileges

We can check `pg_catalog` to see current user's privileges on some schema:

```sql
SELECT pg_catalog.has_schema_privilege(current_user, '<schema>', '<privilege>');
```

For example:

```sql
SELECT
  pg_catalog.has_schema_privilege(
    current_user, 'public', 'CREATE') AS "create", 
  pg_catalog.has_schema_privilege(
    current_user, 'public', 'USAGE') AS "usage";
```

will give us:

```
create | usage 
--------+-------
 t      | t
```

which tells us that current user has privileges  `CREATE` and `USAGE` on schema "public".

(If user has a certain privilege, the response will show 't' for this privilege. Otherwise response will be 'f'.)

### 10. PostgreSQL strcture

```
PostgreSQL -> Database -> Schema -> Table
```

(1) A PostgreSQL server contains one or more databases.

​     A database contains one or more schemas.

​     A schema contains one or more tables.

(2) The privileges assigned to a database won't be assigned to its schemas automically;

​     The privileges assigned to a schema won't be assigned to its tables automically.

### 11. Default database and schema 

(1) PostgreSQL has a default database called "postgres".

(2) Each database has a default schema called "public" which is created when this database is created. So by default, any table we create will be in schema "public".

