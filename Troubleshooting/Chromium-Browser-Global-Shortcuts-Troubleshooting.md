# Arch Linux — Fix Chromium-Based Browser Global Shortcut Spam

This guide describes how to disable the `GlobalShortcutsPortal` feature in Brave and Google Chrome on Linux. It can help address repeated global-shortcut portal prompts or related shortcut behavior when launching Chromium-based browsers in desktop environments such as KDE Plasma.

## 1. Install the Browsers

Install the browsers you want to configure. You only need to follow the steps for browsers installed on your system.

## 2. Configure Brave

### Step 1: Copy the Desktop Entry

Copy the system-wide desktop entry into your user's local applications directory:

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/brave-browser.desktop ~/.local/share/applications/
```

### Step 2: Edit the Desktop Entry

Open the copied file:

```bash
nano ~/.local/share/applications/brave-browser.desktop
```

Find the line:

```ini
Exec=brave %U
```

Change it to:

```ini
Exec=brave %U --disable-features=GlobalShortcutsPortal
```

Save the file and exit the editor.

**Note:** If the original `Exec` line differs on your system, preserve its existing executable and arguments, and append the flag before any field-code arguments where appropriate. In the example above, `%U` is retained.

## 3. Configure Google Chrome

### Step 1: Copy the Desktop Entry

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
```

### Step 2: Edit the Desktop Entry

Open the copied file:

```bash
nano ~/.local/share/applications/google-chrome.desktop
```

Find the main `Exec` line and update it to:

```ini
Exec=/usr/bin/google-chrome-stable %U --disable-features=GlobalShortcutsPortal
```

Save the file and exit.

**Important:** Do not accidentally duplicate `Exec=`. The line should start with `Exec=` exactly once.

## 4. Restart the Browser

Close all running instances of the browser and launch it again from the application menu.

If the change does not take effect, verify that the application menu is using the copied desktop entry under `~/.local/share/applications/`.

## 5. Verify the Configuration

Inspect the modified entries:

```bash
grep '^Exec=' ~/.local/share/applications/brave-browser.desktop
grep '^Exec=' ~/.local/share/applications/google-chrome.desktop
```

Confirm that the relevant launch commands include:

```text
--disable-features=GlobalShortcutsPortal
```

## How It Works

- **`~/.local/share/applications/`** stores user-specific desktop entries that can override system-wide application entries.
- **`Exec=`** specifies the command used to launch the application from its desktop entry.
- **`--disable-features=GlobalShortcutsPortal`** disables the named Chromium feature for launches using that command.

This workaround is intended to address behavior associated with the global shortcuts portal. It is not a universal fix for every shortcut prompt or browser-launch issue, and the feature flag may change or stop working in future browser versions.

---

## Quick Reference

### Brave

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/brave-browser.desktop ~/.local/share/applications/
nano ~/.local/share/applications/brave-browser.desktop
```

Set the relevant `Exec` line to:

```ini
Exec=brave %U --disable-features=GlobalShortcutsPortal
```

### Google Chrome

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
nano ~/.local/share/applications/google-chrome.desktop
```

Set the relevant `Exec` line to:

```ini
Exec=/usr/bin/google-chrome-stable %U --disable-features=GlobalShortcutsPortal
```

**Remember:** Modify the copies in `~/.local/share/applications/`, not the original system-wide desktop files. Browser updates are less likely to overwrite your local copies, although you may need to revisit the configuration if the original desktop entry or launch command changes.