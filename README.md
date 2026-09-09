# Linux Administration & Troubleshooting Notes

Practical Linux administration notes, commands, and troubleshooting techniques based on real-world production support experience.

## Purpose

This repository documents Linux commands, system administration concepts, and troubleshooting methodologies commonly used when diagnosing issues in production environments.

The goal is not simply to document commands, but to understand **when and why to use them** during an incident.

## Contents

- [File & Directory Management](#file--directory-management)
- [Permissions & Ownership](#permissions--ownership)
- [Processes](#processes)
- [CPU & Memory](#cpu--memory)
- [Disk & Storage](#disk--storage)
- [Networking](#networking)
- [SSH](#ssh)
- [Log Analysis](#log-analysis)
- [Text Processing](#text-processing)
- [Production Troubleshooting](#production-troubleshooting)

---

## File & Directory Management

Understanding the Linux filesystem is fundamental to system administration and production troubleshooting. These commands are commonly used to navigate servers, locate files, inspect directories, and determine where disk space is being consumed.

### `pwd`

Prints the current working directory.

```bash
pwd
```

**When to use:**

Useful when working across multiple directories or environments and you need to confirm your current location before running commands.

**Example:**

```bash
pwd
```

```text
/nas/content/live/example
```

---

### `ls`

Lists files and directories.

```bash
ls
```

Commonly useful options:

```bash
ls -lah
```

- `-l` — Long listing format
- `-a` — Include hidden files
- `-h` — Human-readable file sizes

**When to use:**

Useful for quickly inspecting directory contents, checking file permissions and ownership, and identifying configuration or application files.

---

### `cd`

Changes the current working directory.

```bash
cd /path/to/directory
```

Move to the parent directory:

```bash
cd ..
```

Return to the previous directory:

```bash
cd -
```

**When to use:**

Used constantly when navigating a Linux server and moving between application, configuration, log, and system directories.

---

### `find`

Searches for files and directories based on specified criteria.

```bash
find /path -name "filename"
```

Find all files with a specific extension:

```bash
find /path -type f -name "*.log"
```

Find files modified within the last day:

```bash
find /path -type f -mtime -1
```

**When to use:**

Useful when you need to locate files on a server without knowing their exact location. This can be especially helpful when troubleshooting logs, configuration files, backups, or unexpectedly large directories.

---

### `cp`

Copies files or directories.

Copy a file:

```bash
cp source.txt destination.txt
```

Copy a directory recursively:

```bash
cp -r source_directory destination_directory
```

**When to use:**

Useful for creating backups of configuration files or application files before making changes.

---

### `mv`

Moves or renames files and directories.

Rename a file:

```bash
mv old-name.txt new-name.txt
```

Move a file:

```bash
mv file.txt /path/to/destination/
```

**When to use:**

Useful when reorganizing files, renaming configuration files, or temporarily moving files during troubleshooting.

---

### `rm`

Removes files or directories.

Remove a file:

```bash
rm filename.txt
```

Remove a directory and its contents:

```bash
rm -r directory/
```

**When to use:**

Used to remove files or directories that are no longer needed.

**Production consideration:**

`rm` permanently removes files and should be used carefully in production environments. Always verify the path and target before executing destructive commands.

---

### `mkdir`

Creates a new directory.

```bash
mkdir new-directory
```

Create nested directories:

```bash
mkdir -p /path/to/new/directory
```

**When to use:**

Useful when creating directory structures for applications, logs, backups, scripts, or temporary troubleshooting files.

---

### `du`

Displays disk usage for files and directories.

Check the size of a directory:

```bash
du -sh /path/to/directory
```

Find the largest items in the current directory:

```bash
du -sh * | sort -h
```

**When to use:**

Useful when investigating disk space issues and determining which directories or files are consuming the most storage.

---

### `Command Selection During Troubleshooting`

A common workflow when investigating files or directories is:

```text
pwd
  ↓
ls -lah
  ↓
find
  ↓
du
  ↓
Inspect the relevant files
```

The goal is to first establish **where you are**, understand **what exists**, locate the relevant resources, and then determine whether disk usage or file-level issues may be contributing to the problem.

---

## Permissions & Ownership

Linux uses file permissions and ownership to control who can access, modify, or execute files and directories. Understanding permissions is essential when troubleshooting application errors, failed deployments, inaccessible files, and web server issues.

### `ls -l`

Displays detailed information about files and directories, including permissions, ownership, group, size, and modification time.

```bash
ls -l
```

Example:

```text
-rw-r--r--  1 user  group  1234 Sep  1 12:00 example.txt
```

The first section represents the file type and permissions:

```text
-rw-r--r--
```

Breaking this down:

```text
- rw- r-- r--
  │   │   │
  │   │   └── Other users
  │   └────── Group
  └────────── Owner
```

Permission types:

- `r` — Read
- `w` — Write
- `x` — Execute
- `-` — Permission not granted

**When to use:**

Useful when troubleshooting permission-related errors or determining which user and group own a file.

---

### `chmod`

Changes file or directory permissions.

Using symbolic permissions:

```bash
chmod u+x script.sh
```

This adds execute permission for the file owner.

Using numeric permissions:

```bash
chmod 644 example.txt
```

Common permission values:

```text
4 = read
2 = write
1 = execute
```

For example:

```text
644

Owner: 6 = read + write
Group: 4 = read
Other: 4 = read
```

**When to use:**

Useful when a file or script does not have the permissions required for an application or user to access it.

**Production consideration:**

Avoid changing permissions more broadly than necessary. Permissions such as `777` grant read, write, and execute access to everyone and should generally be avoided unless there is a specific, understood reason.

---

### `chown`

Changes the owner and group of a file or directory.

Change the owner:

```bash
chown username file.txt
```

Change the owner and group:

```bash
chown username:groupname file.txt
```

Recursively change ownership:

```bash
chown -R username:groupname directory/
```

**When to use:**

Useful when an application or service cannot access files because they are owned by the wrong user or group.

**Production consideration:**

Use recursive ownership changes carefully. Running `chown -R` against the wrong directory can modify ownership of a large number of files and potentially create additional access problems.

---

### `chgrp`

Changes the group ownership of a file or directory.

```bash
chgrp groupname file.txt
```

**When to use:**

Useful when access needs to be shared with members of a specific Linux group without changing the file's primary owner.

---

### Permission Troubleshooting

When an application reports that it cannot read, write, or execute a file, start by inspecting the file's permissions and ownership.

```bash
ls -l /path/to/file
```

Then verify the directory permissions:

```bash
ls -ld /path/to/directory
```

Check the ownership of the relevant files:

```bash
ls -lah /path/to/directory
```

A useful troubleshooting workflow is:

```text
Identify the affected file
        ↓
Check ownership
        ↓
Check permissions
        ↓
Identify the application/service user
        ↓
Determine the minimum required access
        ↓
Make the smallest necessary change
        ↓
Retest
```

The goal is to restore the required access without unnecessarily weakening the system's security.

---

## Processes

Linux treats running applications and services as processes. Understanding how to identify, inspect, monitor, and manage processes is essential when troubleshooting high CPU usage, memory pressure, application failures, or unexpected system behavior.

### `ps`

Displays information about running processes.

Basic process listing:

```bash
ps
```

Display processes for all users:

```bash
ps aux
```

A common way to search for a specific process is:

```bash
ps aux | grep nginx
```

**When to use:**

Useful when you need to identify which processes are running, which user owns them, and how much CPU or memory they are consuming.

Useful columns from `ps aux` include:

- `USER` — User that owns the process
- `%CPU` — CPU utilization
- `%MEM` — Memory utilization
- `PID` — Process ID
- `STAT` — Process state
- `COMMAND` — Command used to start the process

---

### `top`

Provides a real-time view of running processes and system resource usage.

```bash
top
```

`top` can be used to monitor:

- CPU utilization
- Memory utilization
- Load average
- Running processes
- Process IDs
- Individual process resource consumption

**When to use:**

Useful when investigating a server that appears slow or is experiencing elevated CPU or memory usage.

A typical troubleshooting workflow is:

```text
System appears slow
        ↓
Run top
        ↓
Identify high CPU or memory processes
        ↓
Record the PID
        ↓
Identify the process owner
        ↓
Investigate the application or service
```

---

### `htop`

Provides an interactive alternative to `top` with an easier-to-read process view.

```bash
htop
```

Depending on the Linux distribution, `htop` may need to be installed separately.

**When to use:**

Useful when you need an interactive view of running processes and want to quickly sort or navigate through resource usage.

**Production consideration:**

Do not assume `htop` is available on every production server. `top` is generally more universally available.

---

### `pgrep`

Searches for processes by name or other attributes and returns their process IDs.

```bash
pgrep nginx
```

Include the process name in the output:

```bash
pgrep -a nginx
```

**When to use:**

Useful when you need to quickly identify the PID of a known application or service without manually searching through the entire process list.

---

### `kill`

Sends a signal to a process.

Terminate a process gracefully:

```bash
kill PID
```

For example:

```bash
kill 12345
```

If a process does not respond, a stronger signal can be used:

```bash
kill -9 12345
```

**When to use:**

Useful when a process needs to be stopped or restarted and normal service controls are not sufficient.

**Production consideration:**

`kill -9` should not be the first response. It immediately terminates the process and does not allow the application an opportunity to shut down cleanly.

Whenever possible, attempt a normal termination first and investigate why the process is unresponsive before using a forceful signal.

---

### `pkill`

Terminates processes based on their name or other matching criteria.

```bash
pkill process-name
```

**When to use:**

Useful when you need to terminate processes based on their name rather than locating individual PIDs.

**Production consideration:**

Use `pkill` carefully. A broad match can terminate multiple processes at once.

Always verify the matching processes before using a destructive command.

---

### Process Ownership

Identifying the user that owns a process can provide important context during troubleshooting.

```bash
ps aux
```

Example:

```text
USER       PID  %CPU  %MEM  COMMAND
www-data  1234  85.2   2.1  php-fpm
```

In this example:

```text
User: www-data
PID: 1234
CPU: 85.2%
Memory: 2.1%
```

**When to use:**

Process ownership can help determine which application, service, or user is responsible for unexpected resource consumption.

---

### High CPU Troubleshooting

When a server experiences unusually high CPU utilization, avoid immediately terminating processes.

Start by identifying what is consuming the CPU:

```bash
top
```

or:

```bash
ps aux --sort=-%cpu | head
```

Then identify the process:

```bash
ps -fp PID
```

Check which user owns it:

```bash
ps -o user,pid,ppid,%cpu,%mem,cmd -p PID
```

A practical troubleshooting workflow is:

```text
High CPU detected
        ↓
Identify highest CPU processes
        ↓
Record PID and process owner
        ↓
Determine application/service
        ↓
Review logs and recent changes
        ↓
Determine whether the behavior is expected
        ↓
Take the least disruptive corrective action
        ↓
Monitor CPU utilization
        ↓
Confirm system stability
```

**Production consideration:**

High CPU utilization is a symptom, not necessarily the root cause. Before terminating a process, determine whether it is performing legitimate work such as a backup, deployment, database operation, or scheduled task.

---

### Process Monitoring During an Incident

When investigating production performance issues, process information can be combined with other system metrics.

Useful commands include:

```bash
top
ps aux
free -h
df -h
uptime
```

This helps establish whether the issue is primarily related to:

- CPU
- Memory
- Disk space
- System load
- Individual processes

The objective is to establish a baseline, identify the abnormal behavior, and determine the underlying cause before taking corrective action.