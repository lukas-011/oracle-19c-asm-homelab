# Adding and Dropping Disks

## Overview

This section covers the SQL syntax for adding and dropping disks from ASM disk groups. Both operations automatically trigger a rebalance — when a disk is added, ASM redistributes extents onto it, and when a disk is dropped, ASM redistributes its extents across the remaining disks before removing it. Refer to the ASM Rebalancing section for more detail on controlling rebalance behavior.

---

## Adding a Disk

To add a disk to an existing disk group, use the `ALTER DISKGROUP` statement. ASM will automatically begin rebalancing once the disk is added.

```sql
-- Add a single disk to an existing disk group
ALTER DISKGROUP DATA
  ADD DISK '/dev/sdg' NAME DATA_05;
```

If you are adding a disk to a disk group that uses failure groups, specify which failure group the disk belongs to:

```sql
-- Add a disk to a specific failure group
ALTER DISKGROUP DATA
  ADD FAILGROUP CTRL1 DISK '/dev/sdg' NAME DATA_05;
```

You can also add multiple disks in a single statement:

```sql
-- Add multiple disks at once
ALTER DISKGROUP DATA
  ADD FAILGROUP CTRL1 DISK '/dev/sdg' NAME DATA_05
  ADD FAILGROUP CTRL2 DISK '/dev/sdh' NAME DATA_06;
```

---

## Dropping a Disk

Dropping a disk tells ASM to migrate all extents off that disk before removing it. The disk is not physically removed until the rebalance completes, ensuring no data is lost.

```sql
-- Drop a single disk from a disk group
ALTER DISKGROUP DATA
  DROP DISK DATA_05;
```

You can drop multiple disks in a single statement:

```sql
-- Drop multiple disks at once
ALTER DISKGROUP DATA
  DROP DISK DATA_05
  DROP DISK DATA_06;
```

> **Important:** Never physically remove a disk before the drop operation and rebalance have completed. Removing a disk while ASM is still migrating extents off it can result in data loss.

---

## Monitoring the Rebalance

After adding or dropping a disk, monitor the rebalance progress using the `V$ASM_OPERATION` view:

```sql
SELECT group_number, pass, state, est_minutes
FROM v$asm_operation;
```

If no rows are returned, the rebalance has completed. You can also control the speed of the rebalance using the `POWER` clause:

```sql
-- Add a disk and set rebalance power at the same time
ALTER DISKGROUP DATA
  ADD DISK '/dev/sdg' NAME DATA_05
  REBALANCE POWER 6;
```

Refer to the ASM Rebalancing section for guidance on choosing an appropriate power level.