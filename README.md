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

---

## CPU & Memory

CPU and memory utilization are important indicators of overall system health. When a production server becomes slow or unstable, understanding resource utilization helps determine whether the problem is related to CPU saturation, memory pressure, individual processes, or another underlying issue.

### `uptime`

Displays how long the system has been running, the number of logged-in users, and system load averages.

```bash
uptime
```

Example:

```text
20:15:32 up 12 days,  4:27,  2 users,  load average: 1.25, 1.10, 0.98
```

The load averages represent system load over approximately:

```text
1 minute
5 minutes
15 minutes
```

**When to use:**

Useful as an initial check when a server appears slow or unresponsive.

Load average should be interpreted relative to the number of available CPU cores. A load average of `4` may be significant on a 2-core system but much less concerning on a 16-core system.

---

### `free`

Displays memory and swap usage.

```bash
free -h
```

The `-h` option displays values in a human-readable format.

Example:

```text
               total        used        free      shared  buff/cache   available
Mem:            16Gi        8Gi         2Gi        512Mi        6Gi         7Gi
Swap:            2Gi        0Gi         2Gi
```

Important fields include:

- `total` — Total physical memory
- `used` — Memory currently being used
- `free` — Completely unused memory
- `buff/cache` — Memory used for buffers and filesystem cache
- `available` — Estimated memory available for new applications
- `Swap` — Disk space being used as virtual memory

**When to use:**

Useful when investigating memory-related performance problems or determining whether a server is experiencing memory pressure.

**Production consideration:**

Do not assume that low `free` memory automatically means the server is out of memory. Linux intentionally uses available memory for filesystem caching. The `available` value is generally more useful when assessing whether applications can allocate additional memory.

---

### `top`

`top` provides both CPU and memory information while continuously updating.

```bash
top
```

Useful information includes:

- Overall CPU utilization
- Memory utilization
- Load average
- Running processes
- Process CPU usage
- Process memory usage

**When to use:**

Useful during active incidents when you need to determine whether CPU or memory is contributing to system performance problems.

---

### Checking CPU Usage

List processes sorted by CPU utilization:

```bash
ps aux --sort=-%cpu | head
```

This can quickly identify processes consuming the most CPU.

For a specific process:

```bash
ps -o pid,ppid,user,%cpu,%mem,cmd -p PID
```

**When to use:**

Useful when system-level CPU utilization is elevated and you need to identify the process responsible.

---

### Checking Memory Usage

List processes sorted by memory utilization:

```bash
ps aux --sort=-%mem | head
```

For a specific process:

```bash
ps -o pid,ppid,user,%cpu,%mem,cmd -p PID
```

**When to use:**

Useful when investigating memory pressure and identifying processes consuming unusually large amounts of RAM.

---

### Understanding Load Average

Load average represents the average number of tasks waiting for or actively using CPU resources, along with tasks in certain uninterruptible states.

View load average with:

```bash
uptime
```

or:

```bash
cat /proc/loadavg
```

Example:

```text
1.25 1.10 0.98 2/315 12345
```

The first three values represent the 1-, 5-, and 15-minute load averages.

**Production consideration:**

Load average should always be considered alongside CPU count and other system metrics. A high load average does not automatically mean that CPU utilization is the root cause.

---

### Checking CPU Count

Determine the number of available processors:

```bash
nproc
```

Or:

```bash
lscpu
```

**When to use:**

Useful when interpreting load averages and determining how much CPU capacity is available to the system.

---

### Memory Pressure Troubleshooting

When investigating possible memory-related problems, start with:

```bash
free -h
```

Then identify memory-heavy processes:

```bash
ps aux --sort=-%mem | head
```

Check overall process activity:

```bash
top
```

A practical workflow is:

```text
Application appears slow
        ↓
Check system load
        ↓
Check memory availability
        ↓
Identify resource-heavy processes
        ↓
Determine whether behavior is expected
        ↓
Review application and system logs
        ↓
Investigate the underlying cause
        ↓
Monitor system behavior
```

---

### CPU Troubleshooting

When CPU utilization is unexpectedly high:

```text
High CPU detected
        ↓
Check load average
        ↓
Check CPU count
        ↓
Identify high-CPU processes
        ↓
Identify process owner
        ↓
Determine the application or service
        ↓
Review logs and recent changes
        ↓
Determine root cause
        ↓
Take the least disruptive corrective action
        ↓
Continue monitoring
```

Useful commands:

```bash
uptime
nproc
top
ps aux --sort=-%cpu | head
```

**Production consideration:**

Avoid treating CPU utilization as the root cause by itself. High CPU can be caused by legitimate workloads, application behavior, traffic spikes, scheduled jobs, inefficient queries, runaway processes, or other underlying conditions.

---

### CPU and Memory During Production Incidents

CPU and memory metrics should be evaluated together with application behavior and other system indicators.

Useful commands include:

```bash
uptime
free -h
top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

The objective is to establish whether the system is experiencing:

- CPU saturation
- Memory pressure
- Excessive process activity
- High system load
- An application-specific resource issue

Resource utilization should be correlated with logs, monitoring data, recent deployments, traffic patterns, and other available metrics before determining the root cause.

---

## Disk & Storage

Disk utilization is a common source of production issues. A filesystem approaching capacity can cause applications to fail, logs to stop writing, databases to behave unexpectedly, and services to become unstable.

When troubleshooting disk-related problems, the goal is to determine **which filesystem is affected, what is consuming the space, and whether the usage is expected**.

### `df`

Displays available and used filesystem space.

```bash
df -h
```

The `-h` option displays sizes in human-readable units.

Example:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   42G  5.5G  89% /
```

Important columns include:

- `Filesystem` — Device or filesystem
- `Size` — Total filesystem capacity
- `Used` — Space currently in use
- `Avail` — Available space
- `Use%` — Percentage of space used
- `Mounted on` — Filesystem mount point

**When to use:**

Useful as an initial check when a server reports disk-space problems or when an application begins failing unexpectedly.

---

### `du`

Displays disk usage for files and directories.

Check the size of a directory:

```bash
du -sh /path/to/directory
```

Check the size of items in the current directory:

```bash
du -sh *
```

**When to use:**

Useful when `df` indicates that a filesystem is filling up and you need to determine which directories are consuming the available space.

---

### Combining `du` with `sort`

Sort directory sizes from smallest to largest:

```bash
du -sh * | sort -h
```

Display the largest directories first:

```bash
du -sh * | sort -hr
```

**When to use:**

Useful when quickly identifying which directories are consuming the most disk space.

A common investigation might look like:

```bash
cd /path/to/filesystem
du -sh * | sort -hr
```

Then move into the largest directory and repeat the process.

---

### Finding Large Files

Use `find` to identify files larger than a specified size.

Find files larger than 1 GB:

```bash
find /path -type f -size +1G
```

Find files larger than 500 MB:

```bash
find /path -type f -size +500M
```

Include file sizes in the output:

```bash
find /path -type f -size +500M -exec ls -lh {} \;
```

**When to use:**

Useful when directory-level analysis identifies an area consuming significant disk space and you need to locate individual large files.

Potential sources of unexpected disk usage include:

- Application logs
- Backup files
- Temporary files
- Cache files
- Database dumps
- Uploaded media
- Core dumps
- Old archives

---

### Checking Individual File Sizes

Use `ls -lh` to display file sizes in human-readable format:

```bash
ls -lh
```

For a specific file:

```bash
ls -lh filename
```

**When to use:**

Useful when inspecting individual files identified during disk-space investigations.

---

### Inode Usage

Disk space and inode availability are separate resources.

Check inode utilization with:

```bash
df -i
```

Example:

```text
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/sda1      3276800  327000 2949800   10% /
```

Important fields include:

- `Inodes` — Total available inodes
- `IUsed` — Inodes currently in use
- `IFree` — Available inodes
- `IUse%` — Percentage of inodes used

**When to use:**

Useful when a filesystem reports that it cannot create new files even though `df -h` shows plenty of available disk space.

A system can run out of inodes when it contains an extremely large number of small files.

**Production consideration:**

If inode usage is at or near 100%, deleting or consolidating large numbers of unnecessary small files may be necessary. Always identify the source before removing files.

---

### Disk Space vs. Inode Exhaustion

These two conditions can look similar but have different causes.

#### Disk Space Exhaustion

Check:

```bash
df -h
```

Possible causes include:

- Large log files
- Backups
- Database dumps
- Uploaded files
- Application data
- Temporary files

#### Inode Exhaustion

Check:

```bash
df -i
```

Possible causes include:

- Extremely large numbers of small files
- Application-generated temporary files
- Cache directories
- Session files
- Mail queues
- Log or spool directories

The correct diagnostic command depends on the symptom being observed.

---

### Finding the Largest Directories

A common investigation pattern is to start at the affected filesystem and progressively narrow the search.

```bash
df -h
```

Identify the affected mount point, then:

```bash
du -sh /path/* | sort -hr
```

Move into the largest directory:

```bash
cd /path/to/largest-directory
```

Repeat:

```bash
du -sh * | sort -hr
```

Continue until the source of the disk usage is identified.

This approach avoids immediately scanning the entire filesystem and helps narrow the investigation systematically.

---

### Deleted Files Still Using Disk Space

A file can be deleted from the filesystem while a running process continues to hold the file open.

In this situation:

```bash
df -h
```

may report high disk usage even though:

```bash
du -sh
```

does not appear to account for all of the space.

Open deleted files can be investigated with:

```bash
lsof +L1
```

**When to use:**

Useful when there is a discrepancy between filesystem usage reported by `df` and the space that appears to be accounted for by `du`.

**Production consideration:**

Do not immediately restart or terminate the process holding the deleted file. First identify the process and determine whether restarting it is safe.

---

### Disk Full Troubleshooting Workflow

When a production server reports a disk-space problem:

```text
Disk-space alert
        ↓
Check filesystem utilization
        ↓
df -h
        ↓
Identify affected filesystem
        ↓
Check inode utilization
        ↓
df -i
        ↓
Identify largest directories
        ↓
du -sh * | sort -hr
        ↓
Locate large files if necessary
        ↓
find /path -type f -size +500M
        ↓
Determine whether usage is expected
        ↓
Identify safe cleanup opportunities
        ↓
Monitor filesystem utilization
```

---

### Production Considerations

Disk cleanup should never begin with blindly deleting files.

Before removing anything:

1. Identify what is consuming the space.
2. Determine whether the files are expected.
3. Check whether the files are actively being used.
4. Determine whether the files are required for application operation, backups, auditing, or compliance.
5. Identify whether a retention or rotation policy should be adjusted.
6. Make the smallest safe change.
7. Verify that the application remains healthy afterward.

The goal is not simply to create free space. The goal is to identify **why the filesystem filled up and prevent the issue from recurring**.