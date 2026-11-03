# Linux User and Group Management

## Manage users

### List all users

```bash
cut -d: -f1 /etc/passwd
```

### Create a user

```bash
sudo adduser username
```

### Set or change a user's password

```bash
sudo passwd username
```

### Change a user's home directory

`-m` moves the contents of the old home directory to the new location.

```bash
sudo usermod -d /var/opt/mssql/backup/ -m username
```

> If you see `usermod: user username is currently used by process 28091`, stop that process first (see below).

### Find and stop a process

Check the process:

```bash
ps -fp 28091
```

Or list all processes owned by the user:

```bash
ps -u username
```

Stop the process (try a normal kill first):

```bash
sudo kill 28091
```

Force-kill it only if it doesn't stop:

```bash
sudo kill -9 28091
```

Stop all processes owned by the user:

```bash
sudo pkill -u username
```

### Delete a user

Delete the user only:

```bash
sudo userdel username
```

Delete the user and their home directory:

```bash
sudo userdel -r username
```

## Manage groups

### Show a group and its members

```bash
getent group groupname
```

### Show a user's groups

```bash
groups username
```

### Add a user to a group

`-a` (append) is required. Without it, the user is removed from all other supplementary groups.

```bash
sudo usermod -a -G groupname username
```

> The user must log out and back in for the change to take effect.

### Remove a user from a group

```bash
sudo gpasswd -d username groupname
```