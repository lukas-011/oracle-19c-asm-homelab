# Disk Group Types

## Overview

Oracle ASM disk groups are typically separated by purpose. While you can technically store everything in a single disk group, separating by purpose gives you more control over redundancy, sizing, and performance tuning per workload type.

---

## Standard Disk Groups

The three standard disk groups in an Oracle environment are:

| Disk Group | Full Name | Stores |
|------------|-----------|--------|
| **+CRS** | Cluster Registry Storage | Oracle Cluster Registry (OCR) and Voting Disks — critical files that Grid Infrastructure needs to manage cluster resources. Required for Grid Infrastructure installations. |
| **+DATA** | Data | Database files — datafiles, control files, and redo logs. This is the primary storage for your database. |
| **+FRA** | Fast Recovery Area | Recovery-related files — archived redo logs, RMAN backups, and flashback logs. Keeping this separate from +DATA protects your recovery files if the data disk group fills up. |

> **Note:** These names are conventions, not requirements. Oracle does not enforce these names — you could name your disk groups anything. However, these are the standard naming conventions you will see in almost every Oracle environment.

---

## Sizing Considerations

Each disk group has different storage demands:

- **+CRS** only requires a small amount of space — typically 2-4GB. It stores cluster management files rather than database data, so it stays relatively small
- **+DATA** sizing depends entirely on your database size and growth projections. This is where your actual data lives so it will typically be your largest disk group
- **+FRA** should generally be sized larger than +DATA. Archived redo logs and RMAN backups accumulate over time and can grow quickly, especially in busy environments. Oracle recommends sizing +FRA at a minimum of twice the size of +DATA

---

## How Oracle Uses Disk Groups

When you create a database and set the following initialization parameters, Oracle automatically places the correct files in the correct disk group without you having to manually specify file locations:

```sql
DB_CREATE_FILE_DEST    = '+DATA'
DB_RECOVERY_FILE_DEST  = '+FRA'
```

With these set, Oracle will automatically place datafiles, control files, and redo logs in +DATA, and archived logs, RMAN backups, and flashback logs in +FRA. This is one of the key operational benefits of separating disk groups by purpose — you define the destination once and Oracle handles the rest.