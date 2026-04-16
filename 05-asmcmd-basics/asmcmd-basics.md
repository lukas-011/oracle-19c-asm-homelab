# ASMCMD Basics

## Overview

ASMCMD is the command line interface for managing and navigating Oracle ASM. It provides a shell-like experience for interacting with ASM disk groups, similar to how you would navigate a regular OS filesystem. ASMCMD is run as the `grid` OS user and connects directly to the ASM instance.

To enter ASMCMD:

```bash
su - grid
asmcmd
```

Your prompt will change to `ASMCMD>` once you are inside.

---

## File Management

These commands allow you to navigate and manage files stored within ASM disk groups, similar to standard Linux filesystem commands.

| Command | Description |
|---------|-------------|
| `ls` | List contents of the current ASM directory |
| `ls -l` | List contents with detailed file information |
| `cd` | Change directory within a disk group |
| `pwd` | Print current working directory |
| `mkdir` | Create a directory within a disk group |
| `rm` | Remove a file from a disk group |
| `cp` | Copy a file between ASM and the OS filesystem |
| `mv` | Move or rename a file within ASM |
| `find` | Search for files within a disk group by name pattern |

### Examples

```bash
# List all disk groups
ASMCMD> ls

# Navigate into a disk group
ASMCMD> cd +DATA

# List contents recursively with detail
ASMCMD> ls -l --recurse +DATA

# Find all datafiles in a disk group
ASMCMD> find +DATA -name '*.dbf'

# Copy a file out of ASM to the OS filesystem
ASMCMD> cp +DATA/ORCL/DATAFILE/system.256.1001 /tmp/system.dbf

# Copy a file from the OS filesystem into ASM
ASMCMD> cp /tmp/system.dbf +DATA/ORCL/DATAFILE/system.256.1001

# Remove a file from ASM
ASMCMD> rm +DATA/ORCL/DATAFILE/temp.dbf
```

---

## Disk Group Management

These commands allow you to manage and monitor ASM disk groups without needing to connect to a SQL session.

| Command | Description |
|---------|-------------|
| `lsdg` | List all disk groups and their status |
| `mkdg` | Create a new disk group |
| `dropdg` | Drop a disk group |
| `mountdg` | Mount a disk group |
| `umountdg` | Unmount a disk group |
| `lsdsk` | List all ASM disks and their attributes |
| `lsattr` | List attributes of a disk group |
| `setattr` | Set an attribute on a disk group |

### Examples

```bash
# List all disk groups with state and usage
ASMCMD> lsdg

# List all ASM disks with statistics
ASMCMD> lsdsk --statistics

# Check attributes of a specific disk group
ASMCMD> lsattr -l -G DATA

# Mount a disk group
ASMCMD> mountdg DATA

# Unmount a disk group
ASMCMD> umountdg DATA
```

---

## Instance Management

These commands allow you to check the status of the ASM instance and its connectivity without needing to open a SQL session.

| Command | Description |
|---------|-------------|
| `dsget` | Display the ASM instance the current session is connected to |
| `lsct` | List all database instances currently connected to ASM |
| `spget` | Display the location of the ASM SPFILE |
| `spcopy` | Copy the ASM SPFILE to a specified location |
| `pwget` | Display the location of the ASM password file |

### Examples

```bash
# Check which ASM instance you are connected to
ASMCMD> dsget

# List all databases connected to the ASM instance
ASMCMD> lsct

# Check where the ASM SPFILE is located
ASMCMD> spget
```