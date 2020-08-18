```
888b     d888           .d8888b.   .d88888b.  888      
8888b   d8888          d88P  Y88b d88P" "Y88b 888      
88888b.d88888          Y88b.      888     888 888      
888Y88888P888 888  888  "Y888b.   888     888 888      
888 Y888P 888 888  888     "Y88b. 888     888 888      
888  Y8P  888 888  888       "888 888 Y8b 888 888      
888   "   888 Y88b 888 Y88b  d88P Y88b.Y8b88P 888      
888       888  "Y88888  "Y8888P"   "Y888888"  88888888 
                   888                   Y8b           
              Y8b d88P                                 
               "Y88P"               🐬 Awesome MySQL useful queries and commands   
```

# Awesome MySQL Queries and Commands [![](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/sindresorhus/awesome)


> A curated list of awesome [MySQL](https://www.mysql.com/) useful [queries](https://dev.mysql.com/doc/refman/8.0/en/examples.html) and [commands](https://dev.mysql.com/doc/refman/8.0/en/mysql-commands.html). Inspired by [awesome-mysql](https://github.com/shlomi-noach/awesome-mysql) and [awesome-bash-commands](https://github.com/joseluisq/awesome-bash-commands).

_🏅 Of course, this document needs your help, so [consider contributing](contributing.md)._

#### Table of Contents
- [Commands](#commands)
    - [Data import](#data-import)
    - [Data export](#data-export)
- [Queries](#queries)
    - [Users and privileges](#users-and-privileges)
    - [Select](#select)
    - [Update](#update)
    - [Delete](#delete)
    - [Triggers](#triggers)
    - [Store procedures](#store-procedures)
    - [Utilities](#utilities)

## Commands

### Data import

#### Script file

```sh
mysql -h host -P 3306 -u username -p --default_character_set utf8 database_name < mysql_script.sql
```

### Data export

#### Script file

```sh
mysqldump -h localhost -u username -p database_name > ./mysql_script.sql
```

Or

```sh
mysqldump \
    --user=username \
    --host=127.0.0.1 \
    --protocol=tcp \
    --port=3306 -p \
    --default-character-set=utf8 \
    --skip-triggers \
    "database_name" > database_script.sql
```

#### GZIP script file

```sh
mysqldump -h localhost -u username -p database_name | gzip -c > tables.sql.gz
```

Or

```sh
mysqldump \
    --user=username \
    --host=127.0.0.1 \
    --protocol=tcp \
    --port=3306 -p \
    --default-character-set=utf8 \
    --skip-triggers \
    "database_name" | gzip -c > tables.sql.gz
```

Use `--single-transaction` if you got an mysqldump error (because you lack privileges to lock the tables)

```sh
mysqldump -h localhost -u username -p database_name --single-transaction | gzip -c > tables.sql.gz
```

#### Script file with tables only

```sh
mysqldump -h localhost -u username -p database_name table_name1 table_name2 > mydb_tables.sql
```

Or

```sh
mysqldump \
    --user=username \
    --host=127.0.0.1 \
    --protocol=tcp \
    --port=3306 -p \
    --default-character-set=utf8 \
    --skip-triggers \
    "database_name" "table_name1" "table_name2" > mydb_tables.sql
```

## Queries

### Users and privileges

#### Create a root user equivalent for backward compatibility

```sql
CREATE USER `my_root_user`@`%` IDENTIFIED WITH mysql_native_password BY 'my_root_pwd';

GRANT Alter, Alter Routine, Create, Create Routine, Create Temporary Tables, 
    Create User, Create View, Delete, Drop, Event, Execute, File, Grant Option, 
    Index, Insert, Lock Tables, Process, References, Reload, Replication Client, 
    Replication Slave, Select, Show Databases, Show View, Shutdown, Trigger, Update,
    Super, Create Tablespace
    ON *.* TO `my_root_user`@`%`;
```

**Note:** The above query creates a user using [Native Pluggable Authentication](https://dev.mysql.com/doc/refman/8.0/en/native-pluggable-authentication.html). It can useful for backward compatibility MySQL clients. Due [Caching SHA-2 Pluggable Authentication](https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html) is the default authentication plugin on __MySQL 8__.

#### Create a user with specific database privileges

```sql
CREATE USER `my_user`@`%` IDENTIFIED WITH mysql_native_password BY 'my_password';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE TEMPORARY TABLES, EXECUTE
      ON `my_database`.* TO `my_user`@`%` WITH GRANT OPTION;
```

_Note: User above is an example-purpose only._

#### Modify specific user privileges

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'root2'@'%';
```

#### Change an user password

```sql
ALTER USER 'username'@'localhost' IDENTIFIED BY 'my_new_password';
```

### Select

#### Finding duplicated values

```sql
SELECT code, COUNT(code) duplicates FROM client GROUP BY code HAVING duplicates > 1;
```

#### Select one day ago records

```sql
SELECT * 
FROM users
WHERE
  `registered` >= CONCAT(SUBDATE(CURDATE(), 1), ' 00:00:00') AND 
  `registered` < CONCAT(CURDATE(), ' 00:00:00')
```

### Utilities

#### Clean all tables of one existing database

Those queries create a database if doesn't exist (optional) and then *removes all tables of one specified database*. No root privileges are required, only make sure that the user which executes those queries has [enough privileges](#create-a-user-with-specific-database-privileges) for that particular database.

_**Warning:** This process cleans up the database removing all existing tables permanently. So make sure to do all necessary tests in a development environment first._

```sql
-- -----------------------------------------------------
-- `my_database` clean up process
-- -----------------------------------------------------

-- -----------------------------------------------------
-- 1. Create a new `my_database` database if doesn't exits 
-- This is optional but requires extra privileges
-- -----------------------------------------------------
CREATE SCHEMA IF NOT EXISTS `my_database` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- -----------------------------------------------------
-- 2. Remove all tables of `my_database` database
-- -----------------------------------------------------
SET FOREIGN_KEY_CHECKS = 0;
SET GROUP_CONCAT_MAX_LEN=32768;

USE `my_database`;

SET @tables = NULL;
SELECT GROUP_CONCAT('`', table_name, '`') INTO @tables
FROM information_schema.tables
WHERE table_schema = (SELECT DATABASE());

SELECT IFNULL(@tables, 'dummy') INTO @tables;

SET @tables = CONCAT('DROP TABLE IF EXISTS ', @tables);
PREPARE stmt FROM @tables;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;

SET FOREIGN_KEY_CHECKS = 1;
```

#### Show databases size in MB or GB

**Databases size in GBs**

```sql
SELECT
    table_schema "DB_NAME",
    SUM(ROUND(((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024 / 1024), 2)) AS "GB"
FROM information_schema.tables
GROUP BY table_schema;
```

**Databases size in MBs**

```sql
SELECT
    table_schema "DB_NAME",
    SUM(ROUND(((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024), 2)) AS "MB"
FROM information_schema.tables
GROUP BY table_schema;
```

#### Show status and open database connections

```sql
SHOW GLOBAL STATUS LIKE "%conn%";
SHOW GLOBAL STATUS LIKE '%onn%';
SHOW GLOBAL STATUS LIKE '%Connection_errors%';
```

#### Show performance schema information per query digest

```sql
SELECT
    schema_name AS "Database",
    digest_text AS "Query diggest",
    count_star AS "Executed times",
    avg_timer_wait AS "Executed average (picoseconds)",
    ROUND((avg_timer_wait / 1000 / 1000 / 1000 / 1000), 2) AS "Executed average (seconds)",
    query_sample_text AS "Query sample",
    query_sample_seen AS "Query sample seen"
FROM performance_schema.events_statements_summary_by_digest
ORDER BY avg_timer_wait DESC
LIMIT 15;
```

## Other Awesome Lists
- [awesome-mysql](https://github.com/shlomi-noach/awesome-mysql)
- [awesome-bash-commands](https://github.com/joseluisq/awesome-bash-commands)

## Contributions
Please check out [the contribution file](contributing.md).

## License

[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, [Jose Quintana](http://git.io/joseluisq) has waived all copyright and related or neighboring rights to this work.
