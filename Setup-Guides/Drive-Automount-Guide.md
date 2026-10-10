# Arch Linux — NTFS Drive Setup Sheet

A reusable guide for mounting NTFS drives on Arch Linux, configuring automatic mounting through `/etc/fstab`, and setting Linux user permissions.

## 1. Install NTFS Support

Install `ntfs-3g` using Pacman:

```bash
sudo pacman -S ntfs-3g
```

## 2. Identify Available Drives

List the available partitions, filesystem types, labels, UUIDs, and mount points:

```bash
lsblk -f
```

Identify the NTFS partitions you want to mount and copy their UUIDs.

**Best practice:** Use filesystem UUIDs instead of `/dev/sda1` or `/dev/sdb1`, since device names can change between boots.

## 3. Create Mount Directories

Create a directory under `/mnt` for each drive:

```bash
sudo mkdir -p /mnt/<mount_name>
```

For example:

```bash
sudo mkdir -p /mnt/Games
sudo mkdir -p /mnt/Development
sudo mkdir -p /mnt/Multimedia
```

Choose descriptive directory names without spaces to simplify command-line usage.

## 4. Check User and Group IDs

Check your current Linux user's UID and primary GID:

```bash
id
```

Example output:

```text
uid=1000(user) gid=1000(user) groups=1000(user),...
```

Use the appropriate numeric IDs in your mount configuration. The values are often `1000`, but they can differ between systems.

## 5. Configure Automatic Mounting

Open `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Append one entry per drive, replacing the placeholders with the appropriate UUIDs, mount directories, and UID/GID values:

```fstab
UUID=<UUID_1> /mnt/<mount_name_1> ntfs-3g uid=<UID>,gid=<GID>,umask=002 0 0
UUID=<UUID_2> /mnt/<mount_name_2> ntfs-3g uid=<UID>,gid=<GID>,umask=002 0 0
```

### Mount option reference

| Option | Description |
|---|---|
| `UUID=...` | Identifies the partition to mount. |
| `/mnt/...` | Directory where the partition will be mounted. |
| `ntfs-3g` | Filesystem driver used for NTFS. |
| `uid=1000` | Presents files as owned by the specified Linux user. |
| `gid=1000` | Presents files as owned by the specified Linux group. |
| `umask=002` | Typically grants owner and group read/write access and other users read access. |
| First `0` | Disables the legacy `dump` backup flag. |
| Second `0` | Disables the traditional boot-time filesystem check for this entry. |

**Permissions note:** NTFS mount options generally determine how ownership and permissions are presented to Linux. Use a stricter mask if other local users should not have read access.

## 6. Apply and Test the Configuration

After saving `/etc/fstab`, validate its entries:

```bash
sudo findmnt --verify
```

Reload systemd's manager configuration and mount the configured filesystems:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

Check the results:

```bash
findmnt --target /mnt/<mount_name>
```

Repeat for each mount directory. If a command reports an error, correct the configuration before rebooting.

## 7. Troubleshooting

### UUID mismatch

```bash
lsblk -f
```

Compare the listed UUIDs with those in `/etc/fstab`.

### NTFS drive refuses to mount

- Windows Fast Startup or hibernation may leave the volume in an unsafe state.
- Boot into Windows and perform a full shutdown before trying again.
- Do not force read-write mounting if the filesystem is hibernated or potentially damaged.
- Back up important files before attempting filesystem repair.

### Missing drive delays boot

For nonessential drives, consider the `nofail` mount option if you want the system to continue booting when a drive is unavailable. Understand its implications before adding it.

## 8. Adding Another Drive

1. Identify its UUID using `lsblk -f`.
2. Create its mount directory.
3. Add its entry to `/etc/fstab`.
4. Validate the configuration with `sudo findmnt --verify`.
5. Run `sudo mount -a`.
6. Verify the mount with `findmnt --target /mnt/<mount_name>`.

A reboot is generally unnecessary when adding or changing mount entries.

---

# Personal Configuration — Existing NTFS Drives

This section preserves the actual drive UUIDs and mount directories from my current Arch Linux setup.

## 1. Install NTFS Support

```bash
sudo pacman -S ntfs-3g
```

## 2. Create Mount Directories

```bash
sudo mkdir -p /mnt/Game /mnt/Development /mnt/Multimedia
```

## 3. Edit `/etc/fstab`

```bash
sudo nano /etc/fstab
```

Append the following entries:

```fstab
UUID=2244A6F144A6C6BF /mnt/Game        ntfs-3g uid=1000,gid=1000,umask=002 0 0
UUID=BC5283AF52836CC8 /mnt/Development ntfs-3g uid=1000,gid=1000,umask=002 0 0
UUID=E454CE6854CE3D54 /mnt/Multimedia  ntfs-3g uid=1000,gid=1000,umask=002 0 0
```

**Note:** These are the UUIDs for my existing configuration. If a drive is replaced, reformatted, or its UUID changes, check it with `lsblk -f` and update the entry. The `uid=1000` and `gid=1000` values assume my Linux user and primary group use those IDs.

## 4. Reload and Mount Drives

```bash
sudo findmnt --verify
sudo systemctl daemon-reload
sudo mount -a
```

## 5. Verify the Mounts

```bash
findmnt --target /mnt/Game
findmnt --target /mnt/Development
findmnt --target /mnt/Multimedia
```

If the mounts appear without errors, the drives are mounted at their configured directories.

**Important:** Ensure the UUIDs are correct and the NTFS volumes are in a clean state. Windows Fast Startup or hibernation can prevent safe read-write mounting.

---

## Quick Reference

```bash
# Inspect drives
lsblk -f

# Check UID and GID
id

# Create a mount directory
sudo mkdir -p /mnt/<mount_name>

# Edit mount configuration
sudo nano /etc/fstab

# Validate, reload, and mount
sudo findmnt --verify
sudo systemctl daemon-reload
sudo mount -a

# Verify a mount
findmnt --target /mnt/<mount_name>
```

**Workflow:** Identify the partition → create the mount directory → configure `/etc/fstab` → validate → mount → verify.