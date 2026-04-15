# Installing Oracle 19c Grid Infrastructure

## Overview

This section covers the installation of Oracle 19c Grid Infrastructure, which provides the foundation for running the ASM instance. The installation is performed as the `grid` OS user through the provided GUI installer.

---

## Grid User Environment Setup

Switch to the grid user and edit the `.bash_profile` to set the required environment variables:

```bash
su - grid
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
export ORACLE_BASE=/u01/app/grid
export ORACLE_HOME=/u01/app/grid/19c/grid_home

PATH=$PATH:$HOME/.local/bin:$ORACLE_HOME/bin
export PATH
```

Once saved, execute the profile to apply the changes to your current session:

```bash
. .bash_profile
```

---

## Downloading Grid Infrastructure

Download the Oracle 19c Grid Infrastructure software from the [Oracle Software Delivery Cloud](https://edelivery.oracle.com). Note that an Oracle account is required to access the download.

Once downloaded, copy the zip file to `$ORACLE_HOME` and unzip it:

```bash
cd $ORACLE_HOME
unzip LINUX.X64_193000_grid_home.zip
```

---

## Running the Installer

Launch the Grid Infrastructure installer:

```bash
./gridSetup.sh
```

This will open the GUI installer. Walk through the prompts to configure your Grid Infrastructure and ASM instance. When you reach the storage option, select **Oracle Automatic Storage Management (ASM)** as the storage option.

---

## Running the Root Scripts

Near the end of the installation, the GUI will prompt you to open a separate terminal and run two root scripts. These must be run as the `root` user in the following order:

```bash
/u01/app/oraInventory/orainstRoot.sh
/u01/app/grid/19c/grid_home/root.sh
```

> **Important:** Order matters here. Always run `orainstRoot.sh` before `root.sh`, and do not run these as the `grid` user — they must be executed as `root`. Once both scripts have completed, return to the installer GUI and click **OK** to finish the installation.

---

## Verifying the Installation

Once the installer completes, verify that the Grid Infrastructure and ASM instance are running correctly:

```bash
# Check the ASM instance status
$ORACLE_HOME/bin/crsctl status res -t

# Verify the ASM instance is up
ps -ef | grep asm
```

You should see the ASM instance and Grid Infrastructure resources listed as online.