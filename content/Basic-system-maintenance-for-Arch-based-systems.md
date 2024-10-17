# Basic system maintenance for Arch based systems

## Table of Contents

- [Introduction](#introduction)
- [Maintenance steps](#maintenance-steps)
  1. [Check for failed services](#1-check-for-failed-services)
  2. [System updates](#2-system-updates)
        - [Update the system](#update-the-system)
            - [Updating the package Database](#updating-the-package-database)
        - [Cleaning package cache](#cleaning-package-cache)
        - [Remove orphaned packages](#remove-orphaned-packages)
        - [Clean unused dependencies](#clean-unused-dependencies)
  3. [Cleaning Directories](#3-cleaning-directories-and-temporary-files)
        - [Cleaning directories](#cleaning-directories)
        - [Cleaning files](#cleaning-files)
  4. [Managing system logs](#4-managing-system-logs)
  5. [Update mirror list](#5-update-mirror-list)
  6. [Backup important files](#6-backup-important-files)
- [Optional tools for maintainance](#optional-tools-for-maintenance)

## Introduction

Maintaining an Arch-based Linux system requires regular upkeep to ensure optimal performance, security, and reliability.
\
This guide outlines key maintenance tasks, such as monitoring services, cleaning up package caches, and updating the system.  Here's a basic guide to performing these tasks efficiently.

## Maintenance steps

### 1. Check for failed services

Monitoring system services is crucial for ensuring the system operates smoothly. Failed services may indicate underlying issues that need to be addressed.
\
For systems running [systemd](https://wiki.archlinux.org/title/Systemd) You can use `systemctl` to check for any failed services:

```bash
systemctl --failed
```

This command will list all services that have failed to start or function correctly, along with details such as service names and error codes.

you can view the system log files using journalctl for more information

```bash
sudo journalctl -p 3 -xb
```

This provides a more detailed view of errors or critical issues that might be affecting your system.

---

### 2. System updates

### Update the system

Pacman (**Pac**kage **man**ager) is the Arch linux package manager.
To update both system and [AUR](https://aur.archlinux.org/) (Arch User Repository) packages, run the following:

```bash
sudo pacman -Syu # with pacman
yay -Syu         # with yay
```

- [pacman](https://wiki.archlinux.org/title/Pacman) is for the main repository
- [yay](https://github.com/Jguer/yay) manages AUR packages

use `--noconfirm` for not having to put yes every 2 second

example:

```bash
sudo pacman -Syu --noconfirm 
yay -Syu --noconfirm         
```

### Updating the package database

The package database stores information about available packages from repositories. To ensure your system has the latest package information, you should update the database regularly.

Update the package database:

```bash
sudo pacman -Sy
```

This command refreshes the package database, fetching the latest package information from the repositories without upgrading installed packages.

Force update the database:

```bash
sudo pacman -Syy
```

If you encounter issues with outdated databases or want to ensure a full refresh, you can force the system to re-download the database from scratch.

Note: Running `pacman -Sy` without upgrading your system (`pacman -Syu`) can lead to a "partial upgrade" scenario, which is generally discouraged on Arch-based systems due to possible dependency issues. Always follow up with a full system update:

### Cleaning package cache

Over time, your system accumulates cached packages in `/var/cache/pacman/pkg/`. While these caches allow you to downgrade or reinstall packages without re-downloading them, they can consume a significant amount of disk space.

You can remove unneeded cached packages by:

```bash
sudo pacman -Sc
yay -Sc
```

To delete **EVERYTHING** in the pacman cache, use:

```bash
sudo pacman -Scc
```

Use this with caution, as it removes all cached packages, including those you may want for reinstalling.

### Remove Orphaned Packages

Sometimes, dependencies of removed packages (orphan packages) remain installed. These can be safely removed.

To list all of the orphan packages

```bash
pacman -Qdtq
```

To remove all the orphan packages

```bash
sudo pacman -Rns $(pacman -Qtdq)
```

This command removes both the orphaned packages and any unneeded dependencies.

### Clean Unused Dependencies

Similar to orphans, unnecessary dependencies can accumulate.

To remove all unwanted dependencies using `yay`

```bash
yay -Yc
```

This command removes all unnecessary dependencies installed via AUR.

---

### 3. Cleaning directories and temporary files

### Cleaning directories

System directories, such as `.cache`, can grow large over time with temporary files and cached data. Cleaning these directories helps free up disk space.

To check the size of the .cache dir

```bash
du -sh .cache
```

To remove the contents of `.cache` dir but not the dir itself

```bash
rm -rf .cache/*
```

`-r`: This flag stands for "recursive." It tells the rm command to delete not just the specified directory, but also all of its contents, including subdirectories and their files. **Without** this option, `rm` will not delete directories; it will only delete individual files.

`-f`: This flag stands for "force." It forces the removal of files and directories without prompting for confirmation. This is particularly useful when you're trying to delete files that are write-protected or when you want to avoid the confirmation prompt that `rm` usually shows when deleting files.

### Cleaning files

The cleanup for most other temporary files are handled automatically, but some editors may leave files ending with a ‘~' character laying around. Cleaning these files is a pretty simple find command. You can clean them up under your HOME as a normal user or, if you can be/are root, you can do it for the entire system, but that can be extremely dangerous.

Before deleting any files, it's a good practice to preview which files will be affected. You can do this by running the following command:

```bash
find $HOME -type f -name "*~" -print
```

After that appears to do what you want, add the -exec part. Be extremely careful

```bash
find $HOME -type f -name "*~" -print -exec rm {} \; 
```

**Be Careful**: The -exec rm {} part of the command will permanently delete the files found by the find command. Always ensure you are targeting the correct files.

**System-Wide Cleanup**: If you have root access and wish to clean up backup files across the entire system, you can run a similar command

```bash
sudo find / -type f -name "*~" -print -exec rm {} \; 
```

but be extremely cautious. Accidental deletion of system files could render your system unstable.

---

### 4. Managing system logs

System logs are useful for troubleshooting, but they can also take up significant space if not managed regularly. Logs are stored in `/var/log/journal/` and can accumulate quickly, so regularly review and clean up logs.

To check the size of the journal files

```bash
du -sh /var/log/journal
```

To remove the journal files older than 2 weeks

```bash
sudo journalctl --vacuum-time=2weeks
```

To remove all journal files on your system, you can use the following command:

```bash
sudo journalctl --vacuum-size=1K
```

This command reduces the journal log size to 1 KB, which effectively clears all logs.

Alternatively, you can manually delete all journal files stored in the `/var/log/journal/` directory:

``` bash

sudo rm -rf /var/log/journal/*
```

This will immediately remove all the journal logs.

check the [man page](https://wiki.archlinux.org/title/Man_page) for more information

```bash
man journalctl
```

---

### 5. Update mirror list

Ensuring fast and reliable mirrors is important for smooth updates.

Use [reflector](https://wiki.archlinux.org/title/Reflector) to update your mirrors

```bash
sudo reflector -c China -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

This command fetches mirrors from China that have been updated in the last 6 hours, sorting them by download speed.  You can adjust the country (-c) and age (-a) parameters based on your location and preferences.

---

### 6. Backup Important Files

Regularly backing up your important files is crucial to protect against data loss. Use tools like [rsync](https://wiki.archlinux.org/title/Rsync) to automate backups:

example:

```bash
rsync -a /path/to/source /path/to/destination
```

You can also use cloud solutions or dedicated backup tools such as Timeshift for system snapshots.

---

### Optional Tools for Maintenance

- [stacer](https://aur.archlinux.org/packages/stacer-bin): A linux optimization and monitoring tool.
- [BleachBit](https://archlinux.org/packages/extra/any/bleachbit/): A graphical cleaner for temporary files, logs, and system caches.
- [htop](https://archlinux.org/packages/extra/x86_64/htop/): A more interactive version of the top command for monitoring system resource usage.
- [Timeshift](https://wiki.archlinux.org/title/Timeshift): A system snapshot tool for quick recovery from accidental changes.

---
For more detailed information on system maintenance, refer to the Arch Linux Wiki.

[Arch Linux Wiki on System Maintenance](https://wiki.archlinux.org/title/System_maintenance)

By regularly performing these tasks, you’ll keep your Arch system in good health, with minimal risk of performance degradation or instability.
