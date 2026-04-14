# Oracle ASM Architecture & Concepts

## Overview
ASM sits between the database instance and disk, simplifying, automating, and reducing costs and overhead compared to volume managers and filesystems. In ASM, database files are striped and mirrored automatically (when configured) across all ASM disks which simplifies managing files and mount points. This allows us to have one storage systems for all databases on a system rather than multiple file systems for many databases.

## Oracle ASM Instance

The Oracle ASM instance is built around the same technology as a regular Oracle database instance — it has its own SGA and background processes, but it is not a database itself. Unlike a database instance, the ASM instance has no datafiles of its own; it manages storage for other databases but does not store data directly.

The ASM instance is a shared resource, meaning multiple database instances on the same server can connect to it. This is why it is installed under Oracle Grid Infrastructure rather than being baked into the database — it needs to be independent, available before any database starts, and accessible to all databases on the system.

When a database instance starts up and detects that its storage is on ASM, it automatically spawns the **ASMB** background process. ASMB maintains a persistent connection between the database instance and the ASM instance. This connection is facilitated over Oracle Net through the **Grid Infrastructure Listener** — another reason why Grid Infrastructure is a dependency for ASM.

The ASM instance uses its own initialization parameter file, either an SPFILE or PFILE, similar to a regular database. One important distinction: if you use an SPFILE for the ASM instance, it must be stored inside a disk group rather than on the OS filesystem.



## ASM Background Processes


## ASM Striping
...

## ASM Mirroring
...

## ASM Rebalancing
...

For more information, here is the documentation that I had used to create this overview
https://docs.oracle.com/en/database/oracle/oracle-database/21/ostmg/asm-intro.html#GUID-34A732CD-CC55-4A25-982A-209FDF6134BE