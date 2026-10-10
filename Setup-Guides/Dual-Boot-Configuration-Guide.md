# Arch Linux — GRUB Boot Menu Configuration

Configure GRUB to display the boot menu for a longer duration and detect other operating systems, such as Windows, using `os-prober`.

## 1. Open the GRUB Configuration

```bash
sudo nano /etc/default/grub
```

## 2. Increase the Boot Menu Timeout

Find the following line:

```bash
GRUB_TIMEOUT=5
```

Change it to:

```bash
GRUB_TIMEOUT=30
```

This gives you 30 seconds to select an operating system from the GRUB boot menu before the default entry starts.

**Note:** If `GRUB_TIMEOUT` is set to `0` or `-1`, its behavior differs. Use `30` for a 30-second countdown.

## 3. Enable OS Detection

Find this commented line:

```bash
#GRUB_DISABLE_OS_PROBER=false
```

Uncomment it by removing the `#`:

```bash
GRUB_DISABLE_OS_PROBER=false
```

This enables GRUB's OS-prober integration, allowing it to detect supported operating systems installed on other partitions.

## 4. Install `os-prober`

Install the package:

```bash
sudo pacman -S os-prober
```

## 5. Regenerate the GRUB Configuration

Apply the changes by regenerating the GRUB configuration file:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Review the command output for detected operating systems. If Windows is detected, its boot entry should be included in the generated configuration.

## 6. Verify the Result

Restart the system:

```bash
sudo reboot
```

The GRUB menu should now provide a longer selection window, and detected operating systems should appear as boot options.

---

## Quick Reference

```bash
# Edit GRUB settings
sudo nano /etc/default/grub

# Set the timeout
GRUB_TIMEOUT=30

# Enable OS detection
GRUB_DISABLE_OS_PROBER=false

# Install OS detection utility
sudo pacman -S os-prober

# Regenerate GRUB configuration
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Reboot to test
sudo reboot
```

**Important:** Only edit the settings inside `/etc/default/grub`; do not manually edit `/boot/grub/grub.cfg`, because regenerating it overwrites manual changes. The output path above assumes GRUB is installed with its configuration at `/boot/grub/grub.cfg`.

If another operating system is not detected, check that its partition is accessible and that `os-prober` runs successfully. Some installations may require additional configuration or have security-related restrictions on cross-disk detection.