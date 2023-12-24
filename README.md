![Flushback](https://github.com/MdAhosanHabib/DatabaseFlashback_19c/assets/43145662/20ff2cf9-d892-4a6e-99ea-4d8ae3bd06f1)

# Oracle 19c Database Flashback

This guide provides step-by-step instructions for setting up and using the Oracle 19c Database Flashback feature to recover data or the entire database to a specific point in time or SCN.

**Note**: Before performing any of the following actions, always make sure to take a backup of your database to avoid data loss.

## Enable Database Flashback

### Check Recovery Area for Flashback Log
```bash
$ sqlplus / as sysdba
SQL> SHOW RECYCLEBIN;
SQL> show parameter db_recovery_file_dest;
```

### Create a Destination for Recovery Area & Configure Recovery File Destination and Size
```bash
$ mkdir /db_backup/db_recovery_file_dest

SQL> alter system set db_recovery_file_dest='/db_backup/db_recovery_file_dest' SCOPE=spfile;
SQL> alter system set db_recovery_file_dest_size=10G SCOPE=spfile;
```

### Set Undo Retention
```bash
-- Check undo retention (in seconds)
SQL> SELECT TO_NUMBER(VALUE) / 60 AS "Undo Retention (minutes)"
FROM V$PARAMETER
WHERE NAME = 'undo_retention';

-- Set undo retention (e.g., 259,200 seconds for 3 days)
SQL> ALTER SYSTEM SET undo_retention = 259200 SCOPE=BOTH;
```
**Note**: To better accommodate Oracle Flashback features, you can either set the UNDO_RETENTION parameter to a value 
equal to the longest expected Oracle Flashback operation

### Set Flashback Retention Target & Enable Flashback
```bash
-- Set flashback retention in minutes (e.g., 4,320 for 3 days)
SQL> alter system set db_flashback_retention_target=4320;
SQL> select flashback_on from v$database;

SQL> shutdown immediate;
SQL> startup mount;
SQL> alter database flashback on;
SQL> alter database open;
SQL> select flashback_on from v$database;
```

## Flashback by TIMESTAMP or SCN

### Obtain Current SCN and Timestamp for test
```bash
SQL> SELECT current_scn, SYSTIMESTAMP FROM v$database;
```

### Insert Test Data (for demonstration)
```bash
SQL> create user flash_usr identified by flash_usr;
SQL> grant connect, resource to flash_usr;
SQL> GRANT UNLIMITED TABLESPACE TO flash_usr;
SQL> ALTER USER flash_usr QUOTA UNLIMITED ON users;

SQL> conn flash_usr/flash_usr;
SQL> create table test_ahosan (
  id number(10) Primary key
);
SQL> insert into test_ahosan (id) values (1);
SQL> select * from test_ahosan;
```

### Perform Database Flashback (to SCN or Timestamp)

Flashback to SCN (before user creation)
```bash
SQL> SELECT oldest_FLASHBACK_scn, oldest_FLASHBACK_time from v$flashback_database_log;

SQL> shutdown immediate;
SQL> startup mount;

SQL> Flashback database to scn 209817435;

SQL> Alter database open read only;
SQL> select open_mode from v$database;

-- Check the expected changes
SQL> conn flash_usr/flash_usr;
SQL> select * from test_ahosan;
```

Flashback to Timestamp (before user creation)
```bash
SQL> shutdown immediate;
SQL> startup mount;

SQL> FLASHBACK DATABASE TO TIMESTAMP
TO_TIMESTAMP('2023-12-23 04:15:40', 'YYYY-MM-DD HH:MI:SS');

SQL> Alter database open read only;
SQL> select open_mode from v$database;

-- Check the expected changes
SQL> conn flash_usr/flash_usr;
SQL> select * from test_ahosan;
```

### Finalize Database State (Resetlogs)
```bash
-- After performing the flashback, you may need to reset the logs.
SQL> shutdown immediate;
SQL> startup mount;
SQL> Alter database open resetlogs;
SQL> conn flash_usr/flash_usr;
SQL> select * from test_ahosan;
```

That's it! We've successfully set up and performed Oracle 19c Database Flashback operations. 
Remember to adapt the settings and timestamps as needed for your specific use case.

Medium: https://medium.com/@ahosanhabib.974/oracle-19c-database-flashback-460e817a5546

Thanks from 
### Ahosan
