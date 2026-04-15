# Creating Partitions and ASM Setup

## Overview

This section covers the foundational steps of setting up ASM on Linux: installing the required software, configuring users and groups, initializing ASMLib, creating disk partitions, and marking those partitions as ASM disks.

---

## Package Installation

Start by installing the required packages using yum. If you are already familiar with Oracle database installation, the first package will look familiar.

```bash
yum -y install oracle-database-preinstall-19c.x86_64
yum -y install wget oracleasm kmod-oracleasm oracleasm-support
```

1. **oracle-database-preinstall-19c** — automatically configures all OS prerequisites for Oracle Database 19c, including kernel parameters, resource limits, and required dependencies
2. **oracleasm** — the ASMLib userspace library that provides tools to mark disks as ASM disks and manage them
3. **kmod-oracleasm** — the kernel module that ASMLib requires to communicate with the OS kernel. The userspace library cannot function without this kernel-level component
4. **oracleasm-support** — supporting utilities and scripts required by ASMLib to function properly

---

## User and Group Setup

Next we create the ASM-specific OS groups and the grid user. The grid user owns the Grid Infrastructure installation and the ASM instance.

```bash
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54329 asmadmin

useradd -u 54322 -g oinstall -G dba,asmdba,asmoper,asmadmin grid
```

Then add the oracle user to the ASM groups so it has the necessary access to ASM storage:

```bash
usermod -u 500 -g oinstall -G dba,oper,asmdba,asmoper,asmadmin,kmdba,dgdba,backupdba,racdba oracle
```

Finally, create the directories for the Grid home, Database home, and Oracle Inventory, and assign ownership to their respective users:

```bash
mkdir -p /u01/app/grid/19c/grid_home
mkdir -p /u01/app/oracle/19c/db_home
mkdir -p /u01/app/oraInventory

chown -R oracle:oinstall /u01
chown -R grid:oinstall /u01/app/grid
chown -R grid:oinstall /u01/app/oraInventory
```

---

## ASMLib Initialization and Configuration

Now we initialize ASMLib and configure it to start automatically on boot. Run the following and respond to each prompt as shown:

```bash
oracleasm init

oracleasm configure -i
```

```
Default user to own the driver interface:  grid
Default group to own the driver interface: oinstall
Start Oracle ASM library driver on boot (y/n): y
Scan for Oracle ASM disks on boot (y/n):   y
Writing Oracle ASM library driver configuration: done
```

Setting both boot options to `y` ensures that ASMLib loads automatically and scans for ASM disks whenever the server restarts — without this, ASM disks would need to be manually rescanned after every reboot.

---

## Partition Creation

Now we create the partitions on the OS. In a production environment this is typically handled by the storage team. For this demo we will be creating one partition per disk, where each partition spans the entire disk.

Use `fdisk` for each disk:

```bash
fdisk /dev/sdb
```

Once inside the fdisk prompt, enter the following commands in order:

| Command | Description |
|---------|-------------|
| `n` | Create a new partition |
| `p` | Set partition type to primary |
| `1` | Set partition number to 1 |
| `Enter` | Accept default first sector |
| `Enter` | Accept default last sector (uses the entire disk) |
| `w` | Write changes and exit |

Accepting the default first and last sector is intentional — it makes the partition span the entire disk, which is what we want since each disk is fully dedicated to ASM storage.

Repeat this process for each disk (`/dev/sdc`, `/dev/sdd`, etc.). Once complete, verify the partitions were created:

```bash
lsblk
```

You should see a partition listed under each disk — for example `/dev/sdb1` under `/dev/sdb`.

---

## ASM Disk Creation

Now that the partitions exist, we use the `oracleasm` utility to mark each partition as an ASM disk and assign it a name. This is what makes the partition visible to the ASM instance.

```bash
oracleasm createdisk CRS1  /dev/sdb1
oracleasm createdisk DATA1 /dev/sdc1
oracleasm createdisk FRA1  /dev/sdd1
```

Once the disks are created, scan for them and verify they are listed:

```bash
oracleasm scandisks
oracleasm listdisks
```

You should see `CRS1`, `DATA1`, and `FRA1` returned by `listdisks`, confirming they are registered and ready to be added to a disk group.