# Comprehensive Guide: Fix Unity Hub Infinite Sign-In Loop on Arch Linux

An infinite Unity Hub sign-in loop can occur when the browser cannot return the authentication response to Unity Hub, when Unity Hub cannot persist its authentication state, or when the application has a separate session or installation problem.

The two main areas to investigate are:

1. **Deep-link handling (`unityhub://`)** — Linux must recognize the URL scheme and launch Unity Hub with the authentication callback.
2. **Credential storage** — Unity Hub may rely on Electron's secure-storage integration, which can use a desktop keyring such as KWallet or GNOME Keyring. If no suitable backend is available, credential persistence may fail. <sup>1</sup>

Other causes include stale session data, outdated Hub builds, and browser or installation-specific issues. Diagnose each layer instead of assuming both are broken.

---

## Step 1: Locate Your Unity Hub Installation

First, identify how Unity Hub is installed and which executable is available.

```bash
command -v unityhub
```

If that returns a path, inspect it:

```bash
readlink -f "$(command -v unityhub)"
```

Check whether Pacman owns the executable:

```bash
pacman -Qo "$(command -v unityhub)"
```

If `command -v unityhub` returns nothing, inspect the installed package or launcher:

```bash
pacman -Qs unityhub
find /usr/share/applications ~/.local/share/applications -iname '*unity*hub*.desktop' 2>/dev/null
```

Common executable paths depend on how Unity Hub was installed. Examples include `/usr/bin/unityhub` and `/opt/unityhub/unityhub`; do not assume either path exists.

**Record the actual executable path.** You will use it when inspecting or correcting the desktop launcher.

## Step 2: Inspect the Existing Desktop Launcher

Before creating a new launcher, check whether one already exists.

```bash
grep -Ril 'Unity Hub' \
    /usr/share/applications \
    ~/.local/share/applications 2>/dev/null
```

Inspect the relevant `.desktop` file:

```bash
grep -E '^(Name|Exec|TryExec|MimeType|StartupWMClass)=' \
    /usr/share/applications/unityhub.desktop
```

The system launcher may have a different filename. Substitute the actual path returned by the previous command.

The important entries are:

- `Exec` — must point to the correct executable and accept URL arguments.
- `TryExec` — should identify the available executable.
- `MimeType` — should include `x-scheme-handler/unityhub;`.

If the existing launcher is incorrect, create a user-local copy so package updates do not overwrite your changes.

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/unityhub.desktop \
   ~/.local/share/applications/
```

If the system launcher has a different name, substitute that filename in the copy command.

Edit your user-local copy:

```bash
nano ~/.local/share/applications/unityhub.desktop
```

A minimal example is:

```ini
[Desktop Entry]
Name=Unity Hub
Comment=Unity Hub
Exec=/path/to/unityhub %U
TryExec=/path/to/unityhub
Terminal=false
Type=Application
Icon=unityhub
Categories=Development;
MimeType=x-scheme-handler/unityhub;
```

Replace `/path/to/unityhub` with the real executable path. Preserve any useful existing entries, including a valid `StartupWMClass`, rather than replacing a working launcher unnecessarily.

The `%U` field is important: it allows the desktop environment to pass URLs to Unity Hub.

Save the file and exit Nano with `Ctrl+O`, `Enter`, then `Ctrl+X`.

## Step 3: Register the `unityhub://` URL Handler

Install the desktop utilities if necessary:

```bash
sudo pacman -S --needed desktop-file-utils xdg-utils
```

Refresh the local desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

Register the handler:

```bash
xdg-mime default unityhub.desktop x-scheme-handler/unityhub
```

Check the selected default handler:

```bash
xdg-mime query default x-scheme-handler/unityhub
```

Expected output:

```text
unityhub.desktop
```

If the result differs, check for competing launcher entries and inspect:

```bash
nano ~/.config/mimeapps.list
```

The following associations can be present in that file:

```ini
[Default Applications]
x-scheme-handler/unityhub=unityhub.desktop;

[Added Associations]
x-scheme-handler/unityhub=unityhub.desktop;
```

Do not erase unrelated entries in `mimeapps.list`. The command-line registration is normally sufficient; edit the file manually only when necessary.

## Step 4: Test the Protocol Handler

Test whether Linux launches Unity Hub through the registered URL scheme:

```bash
xdg-open 'unityhub://test'
```

Possible outcomes:

- **Unity Hub launches or comes into focus:** the basic protocol handoff is likely working.
- **The browser or another application opens:** inspect the default MIME handler and desktop entry.
- **Nothing happens or an error appears:** inspect the terminal output and verify that the executable path is correct.

The test URL is not a real authentication callback. A successful test confirms only basic URL dispatch; it does not prove that Unity Hub can complete a login.

You can also verify the handler independently:

```bash
xdg-mime query default x-scheme-handler/unityhub
```

## Step 5: Check Credential Storage

Electron applications can use desktop secret-storage services to protect persistent credentials. The selected backend depends on the application build and desktop environment; installing `libsecret` alone does not create or unlock a keyring. <sup>1</sup>

Install the library:

```bash
sudo pacman -S --needed libsecret
```

Then check which secret-storage system your desktop environment uses.

### KDE Plasma / Wayland

KDE Plasma normally uses KWallet. Electron versions that support the relevant integration can use KWallet for secure storage. <sup>1</sup>

1. Open **System Settings**.
2. Find **KWallet** or **KDE Wallet**.
3. Check that the wallet subsystem is enabled and that the wallet can be opened.
4. If you use automatic unlocking, verify that your login configuration supports it.
5. Log out and back in after changing wallet or session settings.

If KWallet is missing or unavailable, inspect the installed components:

```bash
pacman -Qs kwallet
```

Do not install GNOME Keyring automatically if KWallet is already your working secret-storage provider. Multiple competing Secret Service providers can cause conflicts.

### GNOME Keyring

For systems configured to use GNOME Keyring, install:

```bash
sudo pacman -S --needed gnome-keyring libsecret
```

Check whether the Secret Service is available in your user session:

```bash
busctl --user status org.freedesktop.secrets
```

If the service is unavailable, investigate how your desktop session starts and unlocks GNOME Keyring. On supported sessions, systemd user services and login integration normally handle startup; adding a daemon command to a shell startup file is not universally required.

Avoid blindly adding this to `.bash_profile`, `.zprofile`, or other shell startup files:

```bash
eval "$(gnome-keyring-daemon --start --components=secrets)"
```

The correct startup and environment setup depends on the session manager. Incorrect manual startup can create duplicate or improperly initialized daemon instances.

### Important distinction

The existence of `libsecret` does not prove that Unity Hub is using it. The application may select KWallet, GNOME Keyring, another supported backend, or a fallback depending on its Electron version and launch environment. <sup>1</sup>

## Step 6: Retry Sign-In

After checking the URL handler and keyring:

1. Close Unity Hub completely.
2. Start Unity Hub normally from the desktop launcher.
3. Click **Sign In**.
4. Complete authentication in your browser.
5. If the browser asks whether to open Unity Hub, allow it.
6. Return to Unity Hub and wait for the login process to finish.
7. Close and reopen Unity Hub to see whether the session persists.

If sign-in now works, both the callback and session persistence appear to be functioning.

If it still loops, identify which stage fails:

| Symptom | Next area to investigate |
|---|---|
| Browser finishes login but Unity Hub never opens | URL handler and browser callback |
| Unity Hub opens but returns to the sign-in screen | Authentication callback, session state, or credential storage |
| Login works until Unity Hub closes | Persistent credential storage |
| Login works in one browser but not another | Browser protocol permissions or browser-specific behavior |
| Unity Hub fails before the browser opens | Installation, application startup, or network issues |

## Step 7: Check for Stale Session Data

If the protocol handler and keyring are working, stale application state may be involved.

First, close Unity Hub and confirm it is no longer running:

```bash
pgrep -af unityhub
```

Check the likely configuration directories:

```bash
ls -la ~/.config/UnityHub 2>/dev/null
ls -la ~/.cache/UnityHub 2>/dev/null
```

Directory names can vary with the package and application version. Do not assume every listed directory exists.

Before changing application state, make a backup. For example, if `~/.config/UnityHub` exists:

```bash
cp -a ~/.config/UnityHub \
      ~/.config/UnityHub.backup
```

Do not immediately delete the whole configuration directory. It may contain settings or other application state, and deleting it may not fix a broken URL handler or keyring.

Prefer these steps in order:

1. Restart Unity Hub.
2. Update Unity Hub using the method through which it was installed.
3. Check the desktop URL handler and credential-storage backend again.
4. Back up relevant application data before testing a reset.
5. Reset only the specific state identified as problematic.

## Step 8: Check Installation and Launch Environment

If the problem continues, confirm that Unity Hub is installed correctly and that you are launching the intended executable.

```bash
command -v unityhub
pacman -Qs unityhub
```

If the application was installed through an AUR helper, review the package status and rebuild or update it only when appropriate.

Avoid launching Unity Hub with `sudo`. Running a desktop application as root can create incorrect ownership and session-environment problems.

Also avoid adding undocumented command-line switches without confirming that the installed Unity Hub/Electron version supports them.

## Step 9: Handle Authentication URLs Safely

A browser callback may contain a URL similar to:

```text
unityhub://login/?code=REDACTED&locale=en_US
```

This is an illustrative example only. Actual callback formats vary.

**Do not publish, share, or paste a real authentication callback into a public issue or chat.** Its query parameters may contain a short-lived authentication code or other session-related information.

If you need to investigate a callback failure:

1. Verify the desktop handler first.
2. Allow the browser to open Unity Hub normally.
3. If automatic handoff fails, capture only the non-sensitive error details.
4. Avoid copying the complete callback into shell commands, terminal history, logs, or screenshots.
5. If a manual callback test is unavoidable, follow current official Unity guidance and treat the callback as a temporary credential.

Manually passing the full callback URL to the Unity Hub executable is not a universal fix. It may fail if the callback has expired, has already been consumed, or is not supported in that format.

## Step 10: Final Verification Checklist

- [ ] `command -v unityhub` identifies the expected executable.
- [ ] The desktop launcher points to the correct executable.
- [ ] The launcher's `Exec` entry accepts URL arguments using `%U`.
- [ ] `xdg-mime query default x-scheme-handler/unityhub` returns `unityhub.desktop`.
- [ ] `xdg-open 'unityhub://test'` launches or focuses Unity Hub.
- [ ] A suitable secret-storage backend is available in the active desktop session.
- [ ] Unity Hub completes browser authentication.
- [ ] The session persists after closing and reopening Unity Hub.
- [ ] No real authentication callback or token was exposed during troubleshooting.

## References

1. Electron documentation, [Safe Storage](https://www.electronjs.org/docs/latest/api/safe-storage) — explains Linux credential-storage backends and fallback behavior.
2. ArchWiki, [KDE Wallet](https://wiki.archlinux.org/title/KDE_Wallet) — KDE wallet setup and Secret Service integration.
3. ArchWiki, [GNOME Keyring](https://wiki.archlinux.org/title/GNOME/Keyring) — keyring startup, unlocking, and Secret Service support.
4. Unity documentation, [Install the Unity Hub on Linux](https://docs.unity.com/en-us/hub/install-hub-linux) — official Linux installation information.
