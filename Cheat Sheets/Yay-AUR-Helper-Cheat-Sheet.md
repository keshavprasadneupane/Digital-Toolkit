# Yay Cheat Sheet — Arch Linux AUR Helper

`yay` is an AUR helper for Arch Linux. It searches, installs, upgrades, and removes packages from official repositories and the Arch User Repository (AUR).

> **Important:** AUR packages are community-maintained, not officially supported by Arch Linux. Review their `PKGBUILD` files and installation scripts before building or installing them.

## 1. Installing Yay

| Command | Name | Description |
|---|---|---|
| `sudo pacman -S --needed git base-devel` | Install build tools | Install Git and the tools required to build AUR packages. |
| `git clone https://aur.archlinux.org/yay.git` | Clone Yay | Download Yay's build files from the AUR. |
| `cd yay` | Enter directory | Move into the cloned repository. |
| `less PKGBUILD` | Review build script | Inspect the build instructions before running them. |
| `makepkg -si` | Build and install | Build Yay and install its required dependencies and package. Run as a normal user, not with `sudo`. |
| `yay --version` | Verify installation | Display the installed Yay version. |

**Note:** Run the cloning commands from a suitable working directory. Once installed, you can delete the cloned source directory if you no longer need it.

## 2. Searching for Packages

| Command | Name | Description |
|---|---|---|
| `yay` | Interactive search | Start an interactive package search and installation workflow. |
| `yay package` | Search by keyword | Search for packages matching a term; Yay may offer interactive choices. |
| `yay -Ss package` | Search repositories and AUR | Search for matching package names and descriptions. |
| `yay -Si package` | Remote package information | Display metadata for a package available from a configured repository or AUR. |
| `yay -Qi package` | Installed package information | Display metadata for an installed package. |
| `yay -Sl` | List repository packages | List packages available in configured sync repositories. |
| `pacman -Q` | List installed packages | Show installed packages, regardless of whether they came from official repositories or the AUR. |

Replace `package` with the actual package name or search term.

## 3. Installing Packages

| Command | Name | Description |
|---|---|---|
| `yay -S package` | Install package | Install a package from an official repository or the AUR. |
| `yay -S --needed package` | Install if needed | Skip reinstalling a package that is already up to date. |
| `yay -S package1 package2` | Install multiple packages | Install several packages in one transaction. |
| `yay -S --noconfirm package` | Skip confirmations | Automatically answer prompts with their default responses. Use cautiously. |
| `yay -S --answerclean None package` | Preserve build files | Avoid automatically cleaning existing build files when the relevant Yay prompt is encountered. Check your installed Yay version's help for supported options. |

**AUR-specific installation:** `yay -S package` can install AUR packages as well as official repository packages. You generally do not need `yay -Sa package` to install from the AUR; `-Sa` is not a standard, necessary AUR-installation command.

Avoid `--noconfirm` for unfamiliar packages because it prevents you from reviewing interactive choices and confirmations.

## 4. Updating the System

| Command | Name | Description |
|---|---|---|
| `yay` | Full system upgrade | Refresh package databases, upgrade installed packages, and process available AUR updates. |
| `yay -Syu` | Explicit full upgrade | Perform a full system upgrade, including supported AUR update workflows. |
| `yay -Sua` | AUR-only upgrade | Attempt to upgrade AUR packages without upgrading official repository packages. Use with care; keeping the system fully upgraded is generally safer. |
| `yay --devel -Syu` | Include development updates | Check for updates to development packages, such as Git-based packages, and perform a full upgrade. |
| `yay -Qu` | Check available updates | List available upgrades for installed packages when the relevant package metadata is current. |

**Recommended routine:** Use `yay` or `yay -Syu` for normal updates. Avoid updating only the repository databases and then installing packages without upgrading the system, as this can create a partial upgrade.

Development packages may not have conventional release versions, so checking them more frequently can be useful if you depend on Git-based AUR packages.

## 5. Removing Packages

| Command | Name | Description |
|---|---|---|
| `yay -R package` | Remove package | Uninstall the package while retaining dependencies and configuration files not otherwise removed. |
| `yay -Rs package` | Remove with unused dependencies | Also remove dependencies that are no longer required by another installed package and were installed as dependencies. |
| `yay -Rns package` | Remove package and dependencies | Remove the package, eligible unused dependencies, and system-level configuration files managed by the package manager. |
| `yay -Yc` | Remove unused dependencies | Run Yay's orphan-dependency cleanup workflow. Review the proposed removal list before confirming. |

`-Rns` does not guarantee deletion of every user configuration file in your home directory. Back up any important data before uninstalling software.

## 6. Inspecting AUR Build Files

| Command | Name | Description |
|---|---|---|
| `yay -G package` | Download build files | Download an AUR package's build files, including its `PKGBUILD`, without installing it. |
| `cd package` | Enter build directory | Open the downloaded package directory. |
| `less PKGBUILD` | Inspect PKGBUILD | Review source URLs, dependencies, build instructions, and installation steps. |
| `less .SRCINFO` | Inspect package metadata | Review the package metadata generated for AUR submission. |
| `makepkg --printsrcinfo` | Generate metadata | Print package metadata derived from the `PKGBUILD`. |
| `makepkg -si` | Build and install manually | Build the package and install its dependencies and resulting package after reviewing the build files. |

**Recommended review workflow:**

1. Download the package with `yay -G package`.
2. Inspect the `PKGBUILD` and related scripts.
3. Verify unfamiliar download sources and installation commands.
4. Build and install only if you trust the package.

`yay -S package` normally gives you an opportunity to review AUR build files during installation, depending on your Yay configuration. For a deliberate manual review, use `yay -G` first.

## 7. Querying AUR and Foreign Packages

Pacman's term **foreign package** means a package that is not found in the currently configured sync repositories. A foreign package is often installed from the AUR, but it may also have been installed manually or come from another source.

| Command | Name | Description |
|---|---|---|
| `pacman -Qm` | List foreign packages | Show installed packages that are not present in configured sync repositories. |
| `pacman -Qqm` | List foreign package names | Print only the names of foreign packages. |
| `pacman -Qme` | Explicit foreign packages | Show foreign packages marked as explicitly installed. |
| `pacman -Qqme` | Explicit foreign package names | Print only the names of explicitly installed foreign packages. |
| `pacman -Qqe` | Explicitly installed packages | List packages marked as explicitly installed, including official and foreign packages. |
| `pacman -Qd` | Dependency-installed packages | List packages marked as installed as dependencies. |
| `pacman -Qdt` | Orphan dependencies | List packages marked as dependencies that are no longer required. |
| `yay -Ps` | Yay system statistics | Display Yay's package and system-related statistics. |

These queries use Pacman's local package database. They do not prove that every foreign package came from the AUR or is currently maintained by Yay.

## 8. Cache Cleanup and Maintenance

| Command | Name | Description |
|---|---|---|
| `du -sh /var/cache/pacman/pkg` | Check cache size | Display the disk space used by the package cache. |
| `sudo paccache -r` | Clean package cache | Retain a default number of recent cached versions and remove older ones. Requires `pacman-contrib`. |
| `sudo paccache -rk1` | Keep one version | Retain one cached version of each package and remove older versions. |
| `sudo paccache -ruk0` | Remove uninstalled-package cache | Remove cached package versions that are no longer installed. |
| `yay -Sc` | Clean cache | Run Yay's cache-cleanup workflow; review its prompts before proceeding. |
| `yay -Scc` | Aggressive cache cleanup | Remove cached package files more extensively. This can eliminate convenient rollback and reinstall options. |
| `yay -Y --gendb` | Generate Yay development database | Generate Yay's development-package database for tracking packages that need development-version checks. |

Prefer `paccache` for routine cache management. Do not clear every cached package unnecessarily: keeping recent versions can help with recovery or reinstallations.

## 9. Troubleshooting

| Command | Purpose |
|---|---|
| `yay --version` | Check the installed Yay version. |
| `yay --help` | Review available options for your installed version. |
| `pacman -Qi yay` | Inspect the installed Yay package. |
| `pacman -Qm` | Check foreign packages. |
| `pacman -Dk` | Check local package database consistency. |
| `sudo pacman -Syu` | Perform a full system upgrade if your system is behind; review the proposed transaction first. |

If an AUR build fails, inspect the complete build output before retrying. Common causes include outdated build instructions, changed upstream sources, missing dependencies, or incompatibility with a recent system update.

Do not immediately delete build files or use force options without understanding the error.

## 10. Recommended Maintenance Workflow

Use this as a general routine:

```bash
# 1. Check the installed Yay version
yay --version

# 2. Review and apply system updates
yay

# 3. Review foreign packages, including AUR packages
pacman -Qm

# 4. Review orphaned dependencies
pacman -Qdt

# 5. Remove unused dependencies only after reviewing the list
yay -Yc

# 6. Clean older cached package versions
sudo paccache -r

# 7. Check package cache size
du -sh /var/cache/pacman/pkg
```

The development database command is optional; use it when you want Yay to track updates for development packages:

```bash
yay -Y --gendb
yay --devel -Syu
```

## 11. Best Practices

- **Keep the system fully upgraded.** Prefer `yay` or `yay -Syu` for routine updates.
- **Review AUR build files.** AUR packages are user-submitted build recipes, not officially vetted Arch packages.
- **Avoid unattended installations by default.** Use `--noconfirm` only when you understand the package and transaction.
- **Inspect removals.** Review the list of packages before confirming dependency cleanup.
- **Keep useful cache files.** Retaining recent versions can help with recovery.
- **Check foreign packages periodically.** Some may be manually installed, abandoned, or no longer available.
- **Use official repositories when practical.** Prefer official packages when they meet your needs.
- **Back up important data.** AUR build scripts and package updates can introduce problems.
- **Use Yay as a normal user.** Let it invoke `sudo` when elevated privileges are needed; do not run `yay` itself with `sudo`.

## 12. Quick Reference

| Task | Command |
|---|---|
| Search packages | `yay -Ss package` |
| Install package | `yay -S package` |
| Install without reinstalling current versions | `yay -S --needed package` |
| Update the system | `yay` |
| Check package information | `yay -Si package` |
| Inspect an installed package | `yay -Qi package` |
| Remove package | `yay -Rns package` |
| Download AUR build files | `yay -G package` |
| List foreign packages | `pacman -Qm` |
| List orphan dependencies | `pacman -Qdt` |
| Clean unused dependencies | `yay -Yc` |
| Clean old cached versions | `sudo paccache -r` |
| Check cache size | `du -sh /var/cache/pacman/pkg` |
| Show Yay statistics | `yay -Ps` |
| Review Yay options | `yay --help` |
