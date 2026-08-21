# UNIX & Oracle Reference

A collection of practical UNIX/Linux and Oracle tips, gathered from field notes and consolidated for quick reference.

---

## Table of Contents

- [Top Tips for UNIX Production Administration](#top-tips-for-unix-production-administration)
- [Shell & Command Line](#shell--command-line)
- [System Info & File Handling](#system-info--file-handling)
- [Networking](#networking)
- [Git](#git)
- [Scheduling (cron)](#scheduling-cron)
- [Web / Content Management](#web--content-management)
- [Oracle & SQL Reference](#oracle--sql-reference)
- [Sybase (isql)](#sybase-isql)

---

## Top Tips for UNIX Production Administration

- **Always use `rm -i`, never plain `rm`.** You'll be prompted to confirm each file being deleted — a small friction that's far better than deleting more than intended.
- **When pasting in PuTTY (right-click), keep a partial screen Notepad session open and paste there first**, so you can confirm the contents of your clipboard match what you expect. There's nothing worse than accidentally pasting hundreds of lines of commands straight into production.
- **When releasing files to production, always use SFTP — never copy and paste.** Accidental carriage returns can silently and significantly change the behaviour of a script.
- **Always keep a log of everything you do in PuTTY.** To set this up:
  1. In the PuTTY opening dialogue, single-click your saved session and press **Load**.
  2. In the left panel, go to **Session** → **Logging**.
  3. Select the **"All session output"** radio button.
  4. In the **Log file name** field, enter the full path with host and date parameters, e.g. `C:\putty_logs\&Y-&M-&D-&T_&H.log`.
  5. Select **"Always append to the end of it"**.
  6. The remaining defaults should be fine.
  7. Go back to **Session** and click **Save** to persist these settings against the saved session.

---

## Shell & Command Line

### Use vi-style command history recall

Set your shell to use `vi` key bindings for recalling and editing command history:

```sh
set -o vi
```

Once enabled, you can use:

| Command | Effect |
|---|---|
| `Esc k` | Scroll back through previous commands |
| `Esc /mySearchText` | Search for a command containing `mySearchText` |
| `Esc /^mySearchText` | Search for a command that **starts with** `mySearchText` |
| `Esc /mySearchText$` | Search for a command that **ends with** `mySearchText` |

### Set the erase key

```sh
stty erase <backspace>
```

### Custom shell prompt with hostname

```sh
export PS1=`hostname`':${PWD}> '
```

### Basic arithmetic in a shell script

```sh
MY_COUNT=`expr $MY_COUNT + 1`
```

### Set the default editor

```sh
define _editor=vi
```

---

## System Info & File Handling

### Check the Linux/UNIX version

```sh
uname -a
```

Alternatively:

```sh
cat /etc/system-release
```

### Disk usage and free space

```sh
du -sh     # summary disk usage
du -h      # detailed disk usage
df -h      # free disk space
```

### Display file contents in hex or octal

```sh
od -x      # hex
od -b      # octal
```

---

## Networking

### Copy files with scp

```sh
scp xstoragerule.hda user@127.0.0.1:/tmp
scp -r my_file user@127.0.0.1:/tmp
```

### Exit a telnet session

```
Ctrl-]
quit
```

### Flush and re-register DNS (Windows)

```
ipconfig /flush dns
ipconfig /register dns
```

---

## Git

### List untracked files, respecting .gitignore

```sh
git ls-files --others --exclude-standard
```

### View commit history

```sh
git log
```

### Revert to a previous commit, discarding changes

```sh
git reset --hard 0d1d7fc32
```

---

## Scheduling (cron)

Crontab field reference:

```
# +---------------- minute (0 - 59)
# | +------------- hour (0 - 23)
# | | +---------- day of month (1 - 31)
# | | | +------- month (1 - 12)
# | | | | +---- day of week (0 - 7) (Sunday = 0 or 7)
# | | | | |
30 5 * * 1-5 /home/run_script.ksh > /home/run_$(date "+%Y%m%d_%H%M%S").log 2>&1
```

---

## Web / Content Management

Useful for Oracle/Stellent (UCM) based content systems:

```
RemoteURL <HttpRelativeWebRoot$>idcplg?IdcService=GET_USER_INFO
```

Referenced asset: `StellentDot_edit.gif` – wcm-region.

---

## Oracle & SQL Reference

### Identify the current database

```sql
SELECT ora_database_name FROM dual;
SELECT name FROM v$database;
SELECT global_name FROM global_name;
```

### Flashback database log

```sql
SELECT * FROM v$FLASHBACK_DATABASE_LOG;
```

### List active sessions

```sql
-- all_sessions.sql
SELECT s.sid,
       s.serial#,
       s.osuser,
       s.program
FROM   v$session s;
```

### Generate a "kill session" command

```sql
SELECT 'ALTER SYSTEM KILL SESSION ''' || sid || ',' || serial# || ''';' AS "Command to kill a session:"
FROM   v$session;
```

### Show blocking locks

```sql
-- show_locks.sql
SELECT s1.username || '@' || s1.machine
       || ' ( SID=' || s1.sid || ' ) is blocking '
       || s2.username || '@' || s2.machine || ' ( SID=' || s2.sid || ' )' AS blocking_status
FROM   v$lock l1, v$session s1, v$lock l2, v$session s2
WHERE  s1.sid = l1.sid
AND    s2.sid = l2.sid
AND    l1.BLOCK = 1
AND    l2.request > 0
AND    l1.id1 = l2.id1
AND    l2.id2 = l2.id2;
```

### Scheduler job history

```sql
SELECT log_id, job_name, status, owner, job_subname, error#, additional_info,
       TO_CHAR(log_date, 'DD-MON-YYYY HH24:MI') log_date
FROM   dba_scheduler_job_run_details
WHERE  log_date > TRUNC(sysdate)
ORDER  BY log_id DESC;
```

```sql
SELECT log_id, owner, job_name, job_subname, job_class, operation, status,
       user_name, client_id, global_uid, additional_info,
       TO_CHAR(log_date, 'DD-MON-YYYY HH24:MI') log_date
FROM   dba_scheduler_job_log
WHERE  log_date > TRUNC(sysdate)
ORDER  BY log_id DESC;
```

### Look up a database parameter

```sql
SELECT value
FROM   v$parameter
WHERE  name = 'utl_file_dir';
```

### Useful session settings

```sql
SET SERVEROUTPUT ON SIZE 1000000 FORMAT WRAPPED
```

### Row counts for specific tables

```sql
SELECT table_name, num_rows
FROM   dba_tables
WHERE  table_name IN ('MY_TABLE');
```

### Change a user's password

```sql
ALTER USER ora_user_name IDENTIFIED BY ora_password;
```

### Audit queries

```sql
-- Statement-level audit trail for today
SELECT os_username, username,
       TO_CHAR(timestamp, 'DD-MON-YY HH24:MI:SS') timestamp,
       action_name
FROM   dba_audit_statement
WHERE  timestamp > TRUNC(sysdate)
AND    username != 'SPOT'
ORDER  BY 3;
```

```sql
-- Last DDL time for all tables
SELECT object_name, TO_CHAR(last_ddl_time, 'DD-MON-YYYY HH24:MI:SS')
FROM   dba_objects
WHERE  object_type = 'TABLE';
```

```sql
-- Object-level audit trail for a specific table
SELECT obj_name, action_name, TO_CHAR(timestamp, 'DD-MON-YYYY HH24:MI:SS')
FROM   dba_audit_object
WHERE  obj_name LIKE '%MY_TABLE%'
AND    timestamp > TRUNC(sysdate);
```

### Parsing listener logs

```sh
cd /oracle/product/9.2.0.6/network/log
tail -80000 listener.log | grep -i rwrbe60.exe | cut -d '(' -f 7 | sort -u
```

### Maestro / conman job scheduling

```sh
export MAESTROLINES=0
export MAESTRO_OUTPUT_STYLE=LONG
export MAESTROCOLUMNS=140

conman sj          # show jobs
conman ss           # show schedules only
conman "SHOWSCHEDULES @#@GB1@;KEYS"
```

Batch runtime report:

```sql
SET LINES 133
SET PAGES 1000
COL sched_name FORMAT A10
COL job_name FORMAT A8
BREAK ON REPORT;
BREAK ON batch_date SKIP 1;
COMPUTE SUM OF elapsed_mins ON batch_date

SELECT sched_name, job_name, log_date start_date,
       log_date + (elapsed_mins / 60 / 24) end_date,
       elapsed_mins, batch_date
FROM   maestro_logs
WHERE  log_date > sysdate - 5;
```

---

## Sybase (isql)

### Log in to Sybase isql from UNIX

```sh
isql -Umy_user_name
```

### Choose a database

```sql
1> use my_database_name
2> go
```

### Run a SQL statement

Note: no semi-colon at the end of the statement.

```sql
1> select * from my_table
2> go
```

### List all available databases

```sql
1> sp_helpdb
2> go
```

---

*Compiled from personal field notes accumulated over several years of UNIX/Linux systems administration and Oracle/Sybase database work.*
