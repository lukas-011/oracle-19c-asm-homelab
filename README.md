![Oracle ASM Lab](/f1-oracle-asm.jpg "Oracle ASM Lab")

# Oracle 19c ASM Homelab

> This repository documents an Oracle Database 19c Automatic Storage Management (ASM) homelab with a focus on **understanding the storage layer**, **hands-on configuration**, and **real-world DBA workflows** rather than just automated setup scripts.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Requirements](#requirements)
- [ASM Concepts](#asm-concepts)
- [ASM Setup and DB Setup](#asm-setup-and-db-setup)
- [Disk Groups](#disk-groups)
- [Failure Groups](#failure-groups)
- [ASMCMD Basics](#asmcmd-basics)
- [Database Integration](#database-integration)
- [License](#license)

---

## Project Overview

This homelab is designed to simulate a **real-world Oracle ASM environment** built from scratch.  
The goal is to explore:

- ASM architecture and how it sits between the database and raw storage
- Disk provisioning and ASMLib configuration on Linux
- Disk group creation across all three redundancy levels
- Failure group design and behavior during disk loss
- Day-to-day ASM administration using ASMCMD and SQL

It is **not intended for production automation**, but instead as a learning exercise to demonstrate practical knowledge of the Oracle storage layer.

## Requirements

- Oracle Database 19c
- Oracle Grid Infrastructure 19c
- Oracle Linux 8
- Raw block devices or virtual disks for ASM
- ASMLib (`oracleasm`, `kmod-oracleasm`, `oracleasm-support`)

> Note: This lab uses virtual disks to simulate separate storage controllers and failure groups. In production, ASM disks are typically raw LUNs from a SAN or NVMe devices.

---

## ASM Concepts

Located in `01-asm-concepts/`:

- ASM architecture and how it sits between the database instance and raw storage
- Oracle ASM instance overview — SGA, background processes, and Grid Infrastructure dependency
- ASM background processes — RBAL, ARBn, GMON, ASMB
- ASM striping — coarse vs fine striping and when each is used
- ASM mirroring — External, Normal, and High redundancy
- ASM rebalancing — how it is triggered and how to control it with `asm_power_limit`
- Architecture diagrams made in draw.io

---

## ASM Setup and DB Setup

Located in `02-asm-setup-and-db-setup/`:

- Package installation — `oracle-database-preinstall-19c`, ASMLib packages
- User and group setup for the `grid` and `oracle` OS users
- ASMLib initialization and boot configuration
- Partition creation using `fdisk`
- Marking partitions as ASM disks using `oracleasm createdisk`
- Installing Oracle 19c Grid Infrastructure
- Installing Oracle 19c Database software

---

## Disk Groups

Located in `03-disk-groups/`:

- Standard disk group types — `+CRS`, `+DATA`, `+FRA` and their purpose
- Sizing considerations for each disk group
- How Oracle automatically places files using `DB_CREATE_FILE_DEST` and `DB_RECOVERY_FILE_DEST`
- Creating disk groups across all three redundancy levels — External, Normal, and High
- Adding and dropping disks and monitoring rebalance progress
- Useful ASM monitoring queries against `V$ASM_DISKGROUP`, `V$ASM_DISK`, and `V$ASM_OPERATION`

---

## Failure Groups

Located in `04-failure-groups/`:

- Failure group design principles and how to map them to physical storage topology
- Querying failure group assignments using `V$ASM_DISK`
- Behavior and recovery when a failure group goes offline across each redundancy level

---

## ASMCMD Basics

Located in `05-asmcmd-basics/`:

- File management — navigating disk groups, copying files between ASM and the OS filesystem
- Disk group management — listing, mounting, and checking attributes of disk groups
- Instance management — checking ASM instance connectivity and SPFILE location

---

## License

This project is for **educational purposes**.  
Feel free to fork and adapt for your own learning homelabs.