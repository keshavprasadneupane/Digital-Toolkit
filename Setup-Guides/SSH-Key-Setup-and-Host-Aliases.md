# SSH Key Setup & Host Aliases — Linux

A reusable guide for generating SSH keys in a custom directory, configuring host aliases, and setting up passwordless SSH authentication. It works with any reachable SSH server, whether accessed over a local network, public IP, VPN, or Tailscale.

## 1. Generate an SSH Key in a Custom Directory

Create a directory to store your SSH key pair:

```bash
mkdir -p "$HOME/.ssh/custom-keys"
chmod 700 "$HOME/.ssh/custom-keys"
```

Generate an Ed25519 key pair:

```bash
ssh-keygen -t ed25519 \
  -f "$HOME/.ssh/custom-keys/id_ed25519" \
  -C "server-access"
```

Set a passphrase when prompted if desired.

Secure the private key:

```bash
chmod 600 "$HOME/.ssh/custom-keys/id_ed25519"
```

The generated files are:

| File | Purpose |
|---|---|
| `id_ed25519` | Private key; keep it secret. |
| `id_ed25519.pub` | Public key; install it on the remote server. |

Display the public key so you can copy it:

```bash
cat "$HOME/.ssh/custom-keys/id_ed25519.pub"
```

## 2. Configure an SSH Host Alias

SSH host aliases let you connect using a short name instead of repeatedly entering the server address, username, and key path.

Create the SSH configuration directory and file if necessary:

```bash
mkdir -p "$HOME/.ssh"
chmod 700 "$HOME/.ssh"
touch "$HOME/.ssh/config"
chmod 600 "$HOME/.ssh/config"
```

Open the configuration file:

```bash
nano "$HOME/.ssh/config"
```

Append the following template, replacing the placeholders with your actual server details:

```sshconfig
Host <alias>
    HostName <server_ip_or_hostname>
    User <remote_username>
    IdentityFile <absolute_path_to_private_key>
    IdentitiesOnly yes
```

Example with generic values:

```sshconfig
Host myserver
    HostName 192.168.1.100
    User user
    IdentityFile ~/.ssh/custom-keys/id_ed25519
    IdentitiesOnly yes
```

Save the file.

### Configuration reference

| Option | Purpose |
|---|---|
| `Host` | Shortcut used when connecting. |
| `HostName` | Actual IP address or hostname of the server. |
| `User` | Username on the remote machine. |
| `IdentityFile` | Path to the local private key. |
| `IdentitiesOnly yes` | Restricts identity selection to explicitly configured identities rather than trying unrelated agent keys. |

You can create multiple `Host` entries in the same file, each with its own alias and key.

## 3. Install the Public Key on the Server

First, connect to the server using an existing authentication method, such as an existing SSH key or console access.

On the **remote server**, create the SSH directory:

```bash
mkdir -p "$HOME/.ssh"
chmod 700 "$HOME/.ssh"
```

Open the authorized keys file:

```bash
nano "$HOME/.ssh/authorized_keys"
```

Paste the complete contents of your local `.pub` file into a new line, then save.

Set the permissions:

```bash
chmod 600 "$HOME/.ssh/authorized_keys"
```

Ensure the `.ssh` directory and `authorized_keys` file are owned by the intended remote user.

**Security note:** Copy only the public key to the server. Never transfer your private key to the remote machine.

### Alternative: Install the key automatically

If `ssh-copy-id` is available and the server already permits an initial login:

```bash
ssh-copy-id -i "$HOME/.ssh/custom-keys/id_ed25519.pub" <remote_username>@<server_ip_or_hostname>
```

This can install the public key without manually editing `authorized_keys`.

## 4. Connect Using the Alias

Once the host entry and public key are configured, connect with:

```bash
ssh <alias>
```

For example:

```bash
ssh myserver
```

SSH reads the matching host entry and uses its configured hostname, username, and private key.

Test a remote command without opening an interactive session:

```bash
ssh <alias> 'hostname'
```

Inspect the effective SSH configuration:

```bash
ssh -G <alias>
```

## 5. Troubleshooting

### Debug authentication

```bash
ssh -v <alias>
```

Verbose output helps identify which key SSH offers and whether the server accepts it.

### Check key and directory permissions

```bash
ls -ld "$HOME/.ssh"
ls -l "$HOME/.ssh/custom-keys"
```

### Common problems

- **Connection timeout:** Check network connectivity, firewall rules, VPN connectivity, and the server's SSH service.
- **Permission denied (publickey):** Confirm that the matching public key is installed for the correct remote user.
- **Wrong username:** Verify the `User` setting in the host entry.
- **Wrong key path:** Confirm that `IdentityFile` points to the private key on your local machine.
- **Host alias not working:** Check for spelling errors and duplicate or conflicting host entries.
- **Server uses a nonstandard port:** Add `Port <port_number>` to the corresponding `Host` block.

---

# Personal Configuration — My SSH Shortcut

This section records my actual configuration. The general instructions above can be reused for other machines and servers.

## 1. Create My Custom Key Directory

```bash
mkdir -p /home/keshav/git-cred
chmod 700 /home/keshav/git-cred
```

## 2. Generate My SSH Key

```bash
ssh-keygen -t ed25519 \
  -f /home/keshav/git-cred/id_ed25519 \
  -C "server-access"

chmod 600 /home/keshav/git-cred/id_ed25519
```

Display my public key:

```bash
cat /home/keshav/git-cred/id_ed25519.pub
```

## 3. Configure My SSH Alias

Open the SSH configuration:

```bash
nano ~/.ssh/config
```

Add the following entry:

```sshconfig
Host myserver
    HostName 100.x.y.z
    User keshav
    IdentityFile /home/keshav/git-cred/id_ed25519
    IdentitiesOnly yes
```

Replace `100.x.y.z` with my server's actual IP address or hostname. A `100.x.y.z` address is commonly used as a placeholder for a Tailscale IPv4 address, but the SSH configuration itself is not Tailscale-specific.

## 4. Add My Public Key to the Server

Connect to the server using an existing access method. On the remote server, run:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Paste the contents of my local public key file into a new line:

```text
/home/keshav/git-cred/id_ed25519.pub
```

The path above identifies the local file; paste its **contents**, not the path itself.

Save the file and run:

```bash
chmod 600 ~/.ssh/authorized_keys
```

## 5. Connect Using My Shortcut

From my local machine:

```bash
ssh myserver
```

To debug the connection if necessary:

```bash
ssh -v myserver
```

---

## Quick Reference

```bash
# Generate a custom SSH key
ssh-keygen -t ed25519 -f "$HOME/.ssh/custom-keys/id_ed25519" -C "server-access"

# Display the public key
cat "$HOME/.ssh/custom-keys/id_ed25519.pub"

# Edit SSH host aliases
nano ~/.ssh/config

# Install a public key, if ssh-copy-id is available
ssh-copy-id -i "$HOME/.ssh/custom-keys/id_ed25519.pub" <user>@<host>

# Connect through an alias
ssh <alias>

# Debug the connection
ssh -v <alias>
```

**Core idea:** Generate a key once, install its public half on the server, define a host alias in `~/.ssh/config`, and connect with `ssh <alias>` instead of typing the full connection details each time.