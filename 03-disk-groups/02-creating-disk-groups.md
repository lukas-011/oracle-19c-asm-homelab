# Creating Disk Groups

## Overview

This section covers the SQL syntax for creating ASM disk groups under each redundancy level. Every `CREATE DISKGROUP` statement includes the following key attributes:

- **AU_SIZE** — the Allocation Unit size, which is the smallest unit of storage ASM allocates. 4MB is common for most workloads, though 1MB is the default
- **compatible.asm** — the minimum ASM version required to mount this disk group
- **compatible.rdbms** — the minimum database version that can use this disk group

These compatibility attributes are important — once set, they can only be raised, never lowered.

---

## External Redundancy

Use External Redundancy when the underlying hardware (such as a SAN with RAID) is already handling redundancy. ASM performs no mirroring of its own. This gives you the most usable storage but relies entirely on your hardware for data protection.

```sql
CREATE DISKGROUP DATA_EXT
  EXTERNAL REDUNDANCY
  DISK
    '/dev/sdc',
    '/dev/sdd',
    '/dev/sde'
  ATTRIBUTE
    'AU_SIZE'                 = '4M',
    'CELL_SMART_SCAN_CAPABILITY' = 'FALSE',
    'compatible.asm'          = '19.0',
    'compatible.rdbms'        = '19.0';
```

> **Note:** `CELL_SMART_SCAN_CAPABLE` is an Exadata-specific attribute that enables Exadata Smart Scan offloading. This can be omitted in non-Exadata environments.

---

## Normal Redundancy

Use Normal Redundancy when you want ASM to handle 2-way mirroring across two independent failure groups. This is the most common choice for standard production environments. Each failure group should represent a separate physical controller or storage path so that losing one controller does not affect both copies of the data.

```sql
CREATE DISKGROUP DATA
  NORMAL REDUNDANCY
  FAILGROUP CTRL1 DISK
    '/dev/sdc' NAME DATA_01,
    '/dev/sdd' NAME DATA_02
  FAILGROUP CTRL2 DISK
    '/dev/sde' NAME DATA_03,
    '/dev/sdf' NAME DATA_04
  ATTRIBUTE
    'AU_SIZE'          = '4M',
    'compatible.asm'   = '19.0',
    'compatible.rdbms' = '19.0';
```

---

## High Redundancy

Use High Redundancy when maximum data protection is required. ASM maintains 3-way mirroring across three independent failure groups. This requires at least three failure groups and consumes more raw storage, but ensures the database remains online even if two failure groups are lost simultaneously.

```sql
CREATE DISKGROUP RECO
  HIGH REDUNDANCY
  FAILGROUP CTRL1 DISK
    '/dev/sdg' NAME RECO_0001
  FAILGROUP CTRL2 DISK
    '/dev/sdh' NAME RECO_0002
  FAILGROUP CTRL3 DISK
    '/dev/sdi' NAME RECO_0003
  ATTRIBUTE
    'compatible.asm'   = '19.0',
    'compatible.rdbms' = '19.0';
```