# Failure Groups

## Overview

A failure group is a named set of disks within a disk group that share a common point of failure — typically the same storage controller, HBA, or SAN fabric. ASM uses failure groups to determine where to place mirror copies of data, ensuring that a single hardware failure cannot take down both the primary and mirror copy simultaneously.

The key design principle is simple: **disks that could fail together should be in the same failure group.** ASM will then guarantee that mirror copies always land on disks in different failure groups.

---

## Designing Failure Groups

Before creating a disk group, you need to understand your physical storage topology. Common failure group boundaries are:

| Boundary | Example |
|----------|---------|
| Storage controller | Two HBAs in the same server |
| SAN fabric | Fabric A vs Fabric B in a dual-fabric SAN |
| Storage array | Two separate disk arrays |
| Physical shelf | Different disk shelves in the same array |

In this lab, failure groups are simulated using virtual disks to represent two separate storage controllers. In production, you would map each failure group to a genuine physical boundary in your storage infrastructure.

> **Important:** Keep failure groups balanced in disk count and capacity. ASM mirrors at the failure group level, so if one failure group has significantly more capacity than the other, you will end up with uneven mirror placement and wasted space.

---

## Querying Failure Group Assignments

To verify which disks belong to which failure group:

```sql
SELECT dg.name   AS "Disk Group",
       d.name    AS "Disk Name",
       d.failgroup AS "Failure Group",
       d.path    AS "Path",
       d.state   AS "State",
       ROUND(d.total_mb / 1024, 2) AS "SIZE_GB"
FROM v$asm_disk d
JOIN v$asm_diskgroup dg ON d.group_number = dg.group_number
ORDER BY dg.name, d.failgroup, d.name;
```

---

## What Happens When a Failure Group Goes Offline

When a failure group goes offline — for example, a storage controller fails — ASM behavior depends on your redundancy level:

| Redundancy | Failure Groups Lost | Result |
|------------|--------------------|----|
| External | Any | Data loss — no mirroring |
| Normal | 1 of 2 | Database stays online, running on mirror copies |
| Normal | 2 of 2 | Disk group taken offline |
| High | 1 of 3 | Database stays online |
| High | 2 of 3 | Database stays online on remaining copy |
| High | 3 of 3 | Disk group taken offline |

When a failure group comes back online, ASM will automatically resync the disks in that failure group by reapplying any writes that occurred while it was offline. You can monitor this process the same way you monitor a rebalance:

```sql
SELECT group_number, pass, state, est_minutes
FROM v$asm_operation;
```