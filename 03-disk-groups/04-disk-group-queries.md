# Useful ASM Queries

## Overview

The following queries are commonly used by DBAs for day-to-day ASM monitoring and troubleshooting. These cover disk group usage, database connections, disk paths, rebalance status, disk group attributes, and file placement.

---

## Disk Group Usage

A high level view of disk group usage across all disk groups:

```sql
SELECT name, state, type,
       ROUND(total_mb / 1024, 2) "SIZE_GB",
       ROUND(free_mb / 1024, 2)  "AVAILABLE_GB",
       ROUND((total_mb - free_mb) / total_mb * 100, 2) AS "USED%"
FROM v$asm_diskgroup;
```

For a more granular view broken down by individual disk:

```sql
SELECT dg.name AS "Disk Group",
       d.name  AS "Disk Name",
       ROUND(d.total_mb / 1024, 2) AS "SIZE_GB",
       ROUND(d.free_mb / 1024, 2)  AS "AVAILABLE_GB",
       ROUND((d.total_mb - d.free_mb) / d.total_mb * 100, 2) AS "USED%"
FROM v$asm_disk d
JOIN v$asm_diskgroup dg ON d.group_number = dg.group_number
ORDER BY dg.name, d.name;
```

---

## Databases Using ASM

Shows all database instances currently connected to the ASM instance:

```sql
SELECT instance_name, db_name, status, software_version
FROM v$asm_client;
```

---

## ASM Disk Paths and Status

Shows the physical path, mount status, and header status of each ASM disk. Useful for verifying disks are visible to ASM and confirming their physical locations:

```sql
SELECT name, path, mount_status, header_status
FROM v$asm_disk;
```

---

## Rebalance Status

Shows any active rebalance operations. If no rows are returned, no rebalance is currently running:

```sql
SELECT *
FROM v$asm_operation;
```

---

## Disk Group Attributes

Shows the configuration attributes of a disk group such as AU_SIZE and compatibility settings. Useful when inheriting an environment and needing to understand how disk groups were originally configured:

```sql
SELECT dg.name AS "Disk Group",
       a.name  AS "Attribute",
       a.value AS "Value"
FROM v$asm_attribute a
JOIN v$asm_diskgroup dg ON a.group_number = dg.group_number
ORDER BY dg.name, a.name;
```

---

## Files Stored in a Disk Group

Shows all files stored within a specific disk group. Useful for verifying that database files landed in the correct disk group after creation:

```sql
SELECT dg.name AS "Disk Group",
       f.type  AS "File Type",
       a.name  AS "File Name",
       ROUND(f.bytes / 1048576, 2) AS "SIZE_MB"
FROM v$asm_file f
JOIN v$asm_alias a ON f.group_number = a.group_number
                   AND f.file_number = a.file_number
JOIN v$asm_diskgroup dg ON f.group_number = dg.group_number
ORDER BY dg.name, f.type;
```