# Find Files by Date and Time in Linux

Use `find` to search for files by modification date or time, then list, delete, or run other commands on them with `-exec`.

> Pass the directory path (e.g. `/var/opt/mssql/data/mssqllinux/`) rather than `/path/*`. A `*` is expanded by the shell, which skips hidden files and can fail with "Argument list too long".

## Search by specific date and time (`-newermt`)

`-newermt` matches files modified **after** the given date/time. Add `!` in front to match files modified **at or before** it.

### Files modified after a date and time (excluding `.trn` files)

```bash
find /var/opt/mssql/data/mssqllinux/ -type f ! -name "*.trn" -newermt '2022-06-26 23:00:00' -exec ls -la {} +
```

### Files modified after a date (from midnight)

```bash
find /var/opt/mssql/data/mssqllinux/ -type f ! -name "*.trn" -newermt '2022-06-26' -exec ls -la {} +
```

### Files modified at or before a date and time

```bash
find /var/opt/mssql/data/mssqllinux/ -type f ! -newermt '2022-06-26 23:00:00' -exec ls -la {} +
```

### Files modified between two dates

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -newermt '2022-06-26' ! -newermt '2022-06-27' -exec ls -la {} +
```

### Search the current directory

```bash
find . -type f ! -newermt '2022-06-27 22:00:00' -exec ls -la {} +
```

> Dates like `'26 Jun 2022'` also work, but `YYYY-MM-DD` is unambiguous and recommended.

## Search by number of days (`-mtime`)

`-mtime` counts 24-hour periods back from the current time, not calendar days.

| Option | Meaning |
|--------|---------|
| `-mtime 0` | Modified less than 24 hours ago |
| `-mtime -1` | Modified less than 24 hours ago (same as above) |
| `-mtime +0` | Modified more than 24 hours ago |
| `-mtime +7` | Modified more than 8 days ago (age of 8 or more full days) |
| `-mtime -7` | Modified less than 7 days ago |

### `.bak` files older than 24 hours

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime +0 -print
```

### `.bak` files modified in the last 24 hours

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime -1 -exec ls -la {} +
```

### Use calendar days instead of 24-hour periods

Add `-daystart` to measure from the start of today. With `-daystart`, `-mtime 0` means **today** and `-mtime 1` means **yesterday**.

```bash
find /var/opt/mssql/data/mssqllinux/ -daystart -type f -name "*.bak" -mtime 1 -exec ls -la {} +
```

## Search by minutes (`-mmin`)

| Option | Meaning |
|--------|---------|
| `-mmin -30` | Modified less than 30 minutes ago |
| `-mmin +760` | Modified more than 760 minutes ago |

### `.log` files modified more than 760 minutes ago

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.log" -mmin +760 -print
```

## Time types: modified, accessed, changed

| Days | Minutes | Checks |
|------|---------|--------|
| `-mtime` | `-mmin` | Modification time (file contents changed) |
| `-atime` | `-amin` | Access time (file was read) |
| `-ctime` | `-cmin` | Status change time (permissions, owner, or contents changed) |

> Access time is often updated only occasionally (`relatime` mount option), so `-atime`/`-amin` may be unreliable. Use `-mtime`/`-mmin` in most cases.

## Delete files

> **Always run the command with `-print` first** to check exactly which files will be deleted. Deleted files cannot be recovered.

### Step 1: Preview

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime +7 -print
```

### Step 2: Delete

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime +7 -delete
```

Or with `-exec`:

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime +7 -exec rm -f {} +
```

> Put `-delete` at the **end** of the command. If placed before the filters, it deletes everything `find` reaches.

## Using `-exec`

| Ending | Behavior |
|--------|----------|
| `-exec command {} \;` | Runs the command once per file |
| `-exec command {} +` | Runs the command once with many files (faster) |

`{}` is replaced with each file path found.

### Examples

Compress old log files:

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.log" -mtime +7 -exec gzip {} +
```

Move old backups to another directory:

```bash
find /var/opt/mssql/data/mssqllinux/ -type f -name "*.bak" -mtime +7 -exec mv {} /var/opt/mssql/backup/archive/ \;
```

## Quick Reference

| Task | Command |
|------|---------|
| Modified after a date/time | `find dir/ -type f -newermt 'YYYY-MM-DD HH:MM:SS'` |
| Modified at or before a date/time | `find dir/ -type f ! -newermt 'YYYY-MM-DD HH:MM:SS'` |
| Modified between two dates | `find dir/ -type f -newermt 'DATE1' ! -newermt 'DATE2'` |
| Modified in last 24 hours | `find dir/ -type f -mtime -1` |
| Older than 7 days | `find dir/ -type f -mtime +7` |
| Modified yesterday | `find dir/ -daystart -type f -mtime 1` |
| Modified in last 30 minutes | `find dir/ -type f -mmin -30` |
| Exclude a file type | `find dir/ -type f ! -name "*.trn"` |
| Delete old files | `find dir/ -type f -mtime +7 -delete` |