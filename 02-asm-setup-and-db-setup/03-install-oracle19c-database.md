# Installing Oracle 19c Database

## Overview

This section covers the installation of the Oracle 19c Database software and the creation of a database using DBCA. Note that Oracle Grid Infrastructure must be installed and the ASM instance must be running before proceeding with the database installation.

---

## Oracle User Environment Setup

Switch to the oracle user and edit the `.bash_profile` to set the required environment variables:

```bash
su - oracle
vi .bash_profile
```

Add the following to the file:

```bash
# .bash_profile
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# User specific environment and startup programs
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/19c/db_home

PATH=$PATH:$HOME/.local/bin:$ORACLE_HOME/bin
export PATH
```

Once saved, execute the profile to apply the changes to your current session:

```bash
. .bash_profile
```

---

## Downloading the Database Software

Download the Oracle 19c Database software from the [Oracle Software Delivery Cloud](https://edelivery.oracle.com). An Oracle account is required to access the download.

Once downloaded, copy the zip file to `$ORACLE_HOME` and unzip it:

```bash
cd $ORACLE_HOME
unzip LINUX.X64_193000_db_home.zip
```

---

## Running the Installer

Launch the database installer:

```bash
./runInstaller
```

This will open the GUI installer. Walk through the prompts to configure your database software installation. When prompted for the installation type, select **Install Database Software Only** — the database itself will be created separately using DBCA after the software is installed.

---

## Running the Root Scripts

Near the end of the installation, the GUI will prompt you to open a separate terminal and run the root scripts as the `root` user:

```bash
/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/19c/db_home/root.sh
```

> **Important:** Run these as `root`, not as the `oracle` user, and always run `orainstRoot.sh` before `root.sh`. Return to the installer GUI and click **OK** once both scripts have completed.

---

## Creating the Database with DBCA

Once the database software is installed, use the Database Configuration Assistant (DBCA) to create the database. Launch it as the oracle user:

```bash
dbca
```

This will open the DBCA GUI. When walking through the configuration, make sure to select **Oracle Automatic Storage Management (ASM)** as the storage type. This ensures all database files — datafiles, redo logs, and the control file — are placed on ASM disk groups rather than the OS filesystem, which is the whole point of this lab.

---

## Verifying the Installation

Once DBCA completes, verify the database is running and confirm it is using ASM storage:

```bash
# Switch to the oracle user and connect to the database
sqlplus / as sysdba

# Verify the database is open
SQL> SELECT name, open_mode FROM v$database;

# Confirm database files are on ASM
SQL> SELECT name FROM v$datafile;
SQL> SELECT member FROM v$logfile;
```

All file paths should begin with `+`, indicating they are stored on ASM disk groups rather than the OS filesystem.