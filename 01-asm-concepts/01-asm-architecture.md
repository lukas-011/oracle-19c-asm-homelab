# Oracle ASM Architecture & Concepts

## Overview
ASM sits between the database instance and disk, simplifying, automating, and reducing costs and overhead compared to traditional volume managers and filesystems. In ASM, database files are striped and mirrored automatically (when configured) across all ASM disks, which simplifies managing files and storage. This allows a single storage system to serve all databases on a server rather than requiring separate filesystems for each database.

Oracle introduced ASM to eliminate the dependency on third-party volume managers and OS filesystems for database storage. Before ASM, DBAs had to manually manage striping, mirroring, and file placement across multiple filesystems ASM automates all of that at the database layer.

## Oracle ASM Instance

The Oracle ASM instance is built around the same technology as a regular Oracle database instance it has its own SGA and background processes, but it is not a database itself. Unlike a database instance, the ASM instance has no datafiles of its own; it manages storage for other databases but does not store data directly.

The ASM instance is a shared resource, meaning multiple database instances on the same server can connect to it. This is why it is installed under Oracle Grid Infrastructure rather than being baked into the database it needs to be independent, available before any database starts, and accessible to all databases on the system.

When a database instance starts up and detects that its storage is on ASM, it automatically spawns the **ASMB** background process. ASMB maintains a persistent connection between the database instance and the ASM instance. This connection is facilitated over Oracle Net through the **Grid Infrastructure Listener** another reason why Grid Infrastructure is a dependency for ASM.

The ASM instance uses its own initialization parameter file, either an SPFILE or PFILE, similar to a regular database. One important distinction: if you use an SPFILE for the ASM instance, it must be stored inside a disk group rather than on the OS filesystem.

## ASM Background Processes

| Process | Description |
|---------|-------------|
| **RBAL** | Coordinates rebalancing operations. When disks are added or dropped from a disk group, RBAL is what kicks off the rebalance. |
| **ARBn** | Worker processes that perform the actual rebalancing I/O. RBAL coordinates, ARBn does the heavy lifting. Multiple workers can run simultaneously (ARB0, ARB1, etc.). |
| **GMON** | Monitors disk group membership and disk health. If a disk goes offline, GMON is involved in detecting and handling that event. |
| **ASMB** | Maintains the persistent connection between the database instance and the ASM instance. Spawned automatically when a database connects to ASM storage. |
| **DBWR/LGWR** | Not ASM-specific, but worth noting the database write processes send their I/O through ASM rather than directly to disk when ASM storage is in use. |

## ASM Striping

ASM striping is the process of dividing a file into extents and spreading them evenly across all disks in a disk group. This distribution improves I/O performance and disk utilization by ensuring all disks in the disk group are actively serving I/O rather than concentrating load on a single disk. This automatic striping removes the need for manual I/O performance tuning.

ASM uses two stripe sizes depending on the file type:

- **Coarse Striping (1MB)** — used for datafiles and archive logs where large sequential reads benefit from larger extents
- **Fine Striping (128KB)** — used for redo logs and control files where small, frequent writes require lower latency

Two parameters define how striping behaves:

- **Stripe Depth** — the size of each stripe unit
- **Stripe Width** — the product of the stripe depth and the number of drives in the striped set

For workloads that require more throughput than a single disk can provide, extent striping distributes that load automatically across all disks in the disk group.

## ASM Mirroring

ASM mirroring maintains identical copies of data across multiple disks. Traditional mirroring is typically handled at the OS level, but ASM makes it more flexible certain files can be mirrored while others in the same disk group are not, allowing mixed redundancy within the same storage pool.

ASM offers three redundancy levels, defined at the disk group level:

| Redundancy | Mirror Copies | Min. Failure Groups | Description |
|------------|--------------|---------------------|-------------|
| **External** | None | 1 | ASM performs no mirroring. Assumes the underlying hardware (e.g. SAN with RAID) is already handling redundancy. |
| **Normal** | 2-way | 2 | ASM maintains 1 mirror copy of the data across 2 failure groups. |
| **High** | 3-way | 3 | ASM maintains 2 mirror copies across 3 failure groups for maximum protection. |

## ASM Rebalancing

ASM rebalancing is the process of evenly redistributing data across all disks in a disk group. Rebalancing is triggered automatically when disks are added or dropped from a disk group when a new disk is added, ASM moves extents onto it so no single disk is carrying a disproportionate amount of data, and when a disk is dropped, ASM redistributes its extents across the remaining disks before removing it.

Oracle ASM uses the parameter **asm_power_limit** to control how aggressively the rebalance operation consumes system resources. The value ranges from 0 to 1024:

- **0** — suspends the rebalance entirely
- **1–4** — low resource consumption, minimal impact on database workloads
- **5–10** — moderate speed, suitable for off-hours maintenance
- **11+** — maximum speed, use only during maintenance windows as it can adversely impact database performance

The rebalance will complete on its own, but tuning this parameter gives you control over the tradeoff between rebalance speed and the impact on running workloads.

---

## Resources

[Oracle ASM Introduction - Official Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/21/ostmg/asm-intro.html#GUID-34A732CD-CC55-4A25-982A-209FDF6134BE)