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