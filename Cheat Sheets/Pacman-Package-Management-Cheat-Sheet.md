# Pacman Cheat Sheet — Arch Linux Package Management

Pacman is Arch Linux's native package manager. It handles repository synchronization, package installation, system upgrades, removal, dependency management, package verification, and cache maintenance.

## 1. Package Discovery

| Command | Name | Description |
|---|---|---|
| `pacman -Ss package` | Search Repositories | Search enabled repositories for matching packages. |
| `pacman -Qs package` | Search Installed Packages | Search installed package names and descriptions. |
| `pacman -Si package` | Repository Package Info | Display repository metadata before installation. |
| `pacman -Qi package` | Installed Package Info | Show details about an installed package. |
| `pacman -Sl` | List Repository Packages | List packages available in enabled repositories. |
| `pacman -Q` | List Installed Packages | Display all installed packages. |
| `pacman -Qq` | Package Names Only | List installed package names without extra information. |
| `pacman -V` | Pacman Version | Display the installed Pacman version. |

## 2. Installation

| Command | Name | Description |
|---|---|---|
| `sudo pacman -S package` | Install Package | Install a package and its required dependencies. |
| `sudo pacman -S pkg1 pkg2` | Install Multiple Packages | Install several packages in one transaction. |
| `sudo pacman -S --needed package` | Avoid Reinstallation | Skip reinstalling a package that is already up to date. |
| `sudo pacman -Syu package` | Upgrade and Install | Perform a full system upgrade and install the requested package. |
| `sudo pacman -U file.pkg.tar.zst` | Install Local Package | Install a package archive from disk. |
| `sudo pacman -Sw package` | Download Only | Download a package without installing it. |
| `sudo pacman -Sg group` | List Package Group | List the members of a package group. |
| `sudo pacman -Sg` | List Package Groups | Display available repository package groups. |

**Important:** Prefer `pacman -Syu package` when installing a package on a system whose package databases may be out of date. Arch Linux is a rolling-release distribution; avoid partial upgrades.

## 3. System Updates

| Command | Name | Description |
|---|---|---|
| `sudo pacman -Sy` | Refresh Databases | Download repository database updates without upgrading installed packages. |
| `sudo pacman -Su` | Upgrade Installed Packages | Upgrade using the currently synchronized databases. |
| `sudo pacman -Syu` | Full System Upgrade | Synchronize databases and upgrade the entire system. |
| `sudo pacman -Syyu` | Force Database Refresh | Force-redownload repository databases and perform a full upgrade. |
| `checkupdates` | Check Available Updates | List available updates without synchronizing the main Pacman databases; requires `pacman-contrib`. |

**Recommended update command:**

```bash
sudo pacman -Syu
```

Use `-Syyu` only when you have a reason to force a database refresh, such as troubleshooting stale or inconsistent repository databases.

**Avoid running `sudo pacman -Sy` followed by installing packages without upgrading the system.** This can create an unsupported partial-upgrade state.

## 4. Package Removal

| Command | Name | Description |
|---|---|---|
| `sudo pacman -R package` | Remove Package | Remove the specified package while retaining dependencies and package-managed configuration files where applicable. |
| `sudo pacman -Rs package` | Remove With Dependencies | Also remove dependencies that are no longer required and were installed with the target. |
| `sudo pacman -Rns package` | Remove Package and Dependencies | Remove the package, eligible unneeded dependencies, and package-managed backup configuration files. |
| `sudo pacman -Rsc package` | Recursive Removal | Remove the package and recursively remove dependencies, potentially affecting many packages. Use caution. |
| `sudo pacman -Rdd package` | Skip Dependency Checks | Remove a package while bypassing dependency checks. Can break the system. |

**Warning:** Review Pacman's proposed transaction before confirming removal. Avoid `-Rsc` and `-Rdd` unless you understand their consequences.

## 5. Dependency Management

| Command | Name | Description |
|---|---|---|
| `pactree package` | Dependency Tree | Show dependencies required by a package. Requires `pacman-contrib`. |
| `pactree -r package` | Reverse Dependency Tree | Show packages that depend on the specified package. |
| `pacman -T package` | Check Dependencies | Check whether specified dependencies are missing. |
| `pacman -Qdt` | Find Orphans | List packages installed as dependencies that are no longer required by another installed package. |
| `pacman -Qtdq` | Orphan Package Names | Print only the names of orphaned packages. |
| `pacman -Qe` | Explicit Packages | List packages marked as explicitly installed. |
| `pacman -Qd` | Dependency Packages | List packages marked as dependencies. |

### Remove orphaned packages

First, inspect the list:

```bash
pacman -Qtdq
```

If the list is nonempty and the packages are genuinely unnecessary, remove them:

```bash
sudo pacman -Rns $(pacman -Qtdq)
```

This command fails or produces an unhelpful invocation if no orphans exist. A safer Fish-compatible check is:

```fish
set orphans (pacman -Qtdq)
if test (count $orphans) -gt 0
    sudo pacman -Rns $orphans
else
    echo "No orphaned packages found."
end
```

Review the removal list before confirming.

## 6. Package Files and Ownership

| Command | Name | Description |
|---|---|---|
| `pacman -Ql package` | List Package Files | List files installed by a package. |
| `pacman -Qk package` | Check Package Files | Check whether expected files exist. |
| `pacman -Qkk package` | Detailed File Check | Perform more detailed checks of package-owned files and metadata. |
| `pacman -Qo /path/to/file` | Local File Owner | Identify which installed package owns a file. |
| `sudo pacman -Fy` | Refresh File Database | Download the repository file database. |
| `pacman -F filename` | Search Repository Files | Find repository packages that provide a matching file. |
| `pacman -Fl package` | Repository File List | List files belonging to a package in the repository file database. |

### Example: Find which package provides a command

```bash
pacman -F /usr/bin/example
```

Refresh the file database first if needed:

```bash
sudo pacman -Fy
```

`pacman -Qo` searches locally installed package ownership; `pacman -F` searches the synchronized repository file database.

## 7. Package Cache Management

Pacman caches downloaded package archives in `/var/cache/pacman/pkg/` by default.

| Command | Name | Description |
|---|---|---|
| `du -sh /var/cache/pacman/pkg` | Cache Size | Show the disk space used by the package cache. |
| `ls -lh /var/cache/pacman/pkg` | Inspect Cache | List cached package archives. |
| `sudo paccache -r` | Remove Old Versions | Retain the three most recent cached versions of each package by default. Requires `pacman-contrib`. |
| `sudo paccache -rk1` | Keep One Version | Retain only the newest cached version of each package. |
| `sudo paccache -ruk0` | Clean Uninstalled Packages | Remove cached versions of packages that are no longer installed. |
| `sudo pacman -Sc` | Clean Cache | Remove cached packages that are no longer installed and unused repository databases. |
| `sudo pacman -Scc` | Clear Entire Cache | Remove all cached package archives and cached package files. |

**Recommendation:** Prefer `paccache -r` for routine cleanup. Keeping older package archives can help with downgrades or recovery.

Be cautious with `pacman -Scc`: clearing the entire cache removes locally cached package versions that might otherwise be useful for recovery.

## 8. Package Queries and Classification

| Command | Name | Description |
|---|---|---|
| `pacman -Qe` | Explicit Packages | List packages marked as explicitly installed. |
| `pacman -Qd` | Dependency Packages | List packages marked as dependencies. |
| `pacman -Qm` | Foreign Packages | List installed packages absent from synchronized repository databases, including many AUR packages and locally installed packages. |
| `pacman -Qmq` | Foreign Package Names | Print names of foreign packages only. |
| `pacman -Qme` | Explicit Foreign Packages | List explicitly installed foreign packages. |
| `pacman -Qqe` | Explicit Package Names | List explicitly installed package names. |
| `pacman -Qdt` | Orphaned Dependencies | List dependency-marked packages that are no longer required. |

**Note:** A foreign package is not necessarily an AUR package. Pacman uses this classification for packages that are not present in its currently synchronized repositories.

## 9. Verification and Database Maintenance

| Command | Name | Description |
|---|---|---|
| `pacman -Qk package` | Basic Package Check | Check whether package-owned files are present. |
| `pacman -Qkk package` | Detailed Package Check | Check files and metadata more thoroughly. |
| `sudo pacman -Dk` | Database Consistency Check | Check the local package database for dependency consistency. |
| `sudo pacman -Dkk` | Detailed Database Check | Perform additional database consistency checks. |
| `pacman -Q` | Query Installed Database | Check which packages Pacman records as installed. |

### Verify a package

```bash
pacman -Qk package
```

### Verify all installed packages

```bash
pacman -Qk
```

For detailed checks across all packages:

```bash
pacman -Qkk
```

These checks help identify missing or altered package-owned files. A reported difference does not automatically mean the system is compromised; configuration files and intentionally modified files may differ.

## 10. Logs and Package History

Pacman records package transactions in `/var/log/pacman.log`.

| Command | Name | Description |
|---|---|---|
| `tail -100 /var/log/pacman.log` | Recent Log Entries | Show the last 100 lines of the Pacman log. |
| `grep '\[ALPM\]' /var/log/pacman.log` | Package Transactions | Filter log entries associated with package operations. |
| `grep '\[ALPM\].*installed' /var/log/pacman.log` | Installation History | Find package installation entries. |
| `grep '\[ALPM\].*removed' /var/log/pacman.log` | Removal History | Find package removal entries. |
| `grep '\[ALPM\].*upgraded' /var/log/pacman.log` | Upgrade History | Find package upgrade entries. |
| `grep '\[ALPM\]' /var/log/pacman.log \| tail -100` | Recent Package Activity | Show the latest package transaction log entries. |

### Follow the log live

```bash
tail -f /var/log/pacman.log
```

Press `Ctrl+C` to stop following the log.

## 11. Useful Package Management Workflows

### A. Search and install a package

```bash
pacman -Ss package
pacman -Si package
sudo pacman -Syu package
```

### B. Inspect an installed package

```bash
pacman -Qi package
pacman -Ql package
pacman -Qk package
```

### C. Find and inspect orphaned packages

```bash
pacman -Qtdq
```

Review the output before removing any packages.

### D. Perform routine maintenance

```bash
# Upgrade the system
sudo pacman -Syu

# Review orphaned dependencies
pacman -Qtdq

# Clean older cached packages
sudo paccache -r
```

### E. Find the package that owns a file

```bash
pacman -Qo /usr/bin/example
```

If the file is not installed locally, search the repository file database:

```bash
sudo pacman -Fy
pacman -F /usr/bin/example
```

## 12. Best Practices

1. **Perform full system upgrades.** Prefer `sudo pacman -Syu` over refreshing databases without upgrading.
2. **Review transaction prompts.** Check which packages will be installed, upgraded, or removed before confirming.
3. **Keep a useful package cache.** Retaining previous package archives can assist with troubleshooting and downgrades.
4. **Inspect orphaned packages before removing them.** Dependency classifications do not always reflect whether a package is still useful to you.
5. **Understand foreign packages.** Review AUR and locally installed packages separately when maintaining the system.
6. **Use file ownership queries.** `pacman -Qo` and `pacman -F` help distinguish locally owned files from files available in repositories.
7. **Avoid force-removal flags.** Options such as `-Rdd` bypass safeguards and may leave dependencies broken.
8. **Use an AUR helper only when needed.** Tools such as `paru` or `yay` extend package workflows for AUR packages, but they are not replacements for understanding Pacman.
9. **Read update announcements for major changes.** Arch Linux updates can occasionally require manual intervention.
10. **Keep backups of important data and configuration.** Package management is not a substitute for a recovery plan.

---

## Quick Reference

```bash
# Search repositories
pacman -Ss package

# Inspect a package
pacman -Si package
pacman -Qi package

# Install a package with a full system upgrade
sudo pacman -Syu package

# Update the system
sudo pacman -Syu

# Remove a package
sudo pacman -Rns package

# List orphaned dependencies
pacman -Qtdq

# List files installed by a package
pacman -Ql package

# Find which installed package owns a file
pacman -Qo /path/to/file

# Search repository files
sudo pacman -Fy
pacman -F filename

# Clean cached package versions
sudo paccache -r

# View recent package operations
grep '\[ALPM\]' /var/log/pacman.log | tail -100

# Check package files
pacman -Qk package
```

**Remember:** Pacman is both an installer and a system-maintenance tool. Use full upgrades, inspect removals, preserve useful cached packages, and avoid bypassing dependency checks unless you fully understand the consequences.