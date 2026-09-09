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