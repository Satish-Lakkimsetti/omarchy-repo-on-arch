# Add Omarchy Repository to Arch Linux

A beginner-friendly guide to adding the official Omarchy package repository to a regular Arch Linux installation.

All steps use copy-pasteable terminal commands. No manual configuration editing is required.

## Table of Contents

- Introduction
- Requirements
- 1\. Update Arch Linux
- 2\. Install Required Packages
- 3\. Check Existing Repository Configuration
- 4\. Add the Omarchy Repository
- 5\. Import the Signing Key
- 6\. Synchronize Package Databases
- 7\. Verify the Repository
- 8\. Install Omarchy Packages
- Troubleshooting
- Official Links

## Introduction

Omarchy provides its own package repository for Arch Linux.

By adding this repository, you can access packages maintained by Omarchy through Pacman, the Arch Linux package manager.

This guide explains how to configure the Omarchy repository on a regular Arch Linux installation.

**Note:** Adding the repository does not automatically install the complete Omarchy desktop environment.

## Requirements

- A working Arch Linux installation.
- An active internet connection.
- Sudo privileges.
- Basic terminal access.

## 1\. Update Arch Linux

First, update your Arch Linux installation.

bash

```bash
sudo pacman -Syu
```

**Explanation:**

- `sudo` executes the command with administrator privileges.
- `pacman` is the Arch Linux package manager.
- `-Syu` synchronizes package databases and upgrades installed packages.

## 2\. Install Required Packages

Install the tools required for repository and signing-key management.

bash

```bash
sudo pacman -S --needed curl gnupg
```

**Explanation:**

- `curl` is a command-line tool for communicating with web servers.
- `gnupg` provides GPG utilities for cryptographic key management.
- `--needed` avoids reinstalling packages that are already installed.

## 3\. Check Existing Repository Configuration

Before adding the repository, check whether it is already configured.

bash

```bash
grep -n '^\[omarchy\]' /etc/pacman.conf
```

**Explanation:**

- `grep` searches for matching text.
- `-n` displays the line number.
- `^\[omarchy\]` searches for the Omarchy repository section.

If the command displays `[omarchy]`, the repository is already configured. Do not append another copy.

If there is no output, continue to the next step.

## 4\. Add the Omarchy Repository

Run the following command to automatically append the Omarchy repository configuration to `/etc/pacman.conf`.

bash

```bash
sudo tee -a /etc/pacman.conf <<'EOF'

[omarchy]
SigLevel = Required DatabaseOptional
Server = https://pkgs.omarchy.org/stable/$arch
EOF
```

**Explanation:**

- `sudo` executes the command with administrator privileges.
- `tee` writes the provided text to a file.
- `-a` appends the configuration without deleting existing contents.
- `/etc/pacman.conf` is the Pacman configuration file.
- `<<'EOF'` allows multiple lines to be written directly into the file.

No text editor is required\!

**Important:** Run this command only once. Running it multiple times will create duplicate repository entries.

### Verify the configuration

bash

```bash
grep -A3 '^\[omarchy\]' /etc/pacman.conf
```

Expected output:

ini

```ini
[omarchy]
SigLevel = Required DatabaseOptional
Server = https://pkgs.omarchy.org/stable/$arch
```

## 5\. Import the Signing Key

Import the official Omarchy repository signing key.

bash

```bash
sudo pacman-key --recv-keys 40DFB630FF42BCFFB047046CF0134EE680CAC571 --keyserver keys.openpgp.org
```

**Explanation:**

- `pacman-key` manages Pacman's trusted signing keys.
- `--recv-keys` retrieves the specified public key.
- `--keyserver` specifies the keyserver.
- The fingerprint identifies the signing key.

Verify the fingerprint against the official Omarchy documentation before trusting it.

### Locally sign the key

bash

```bash
sudo pacman-key --lsign-key 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

**Explanation:**

This locally marks the imported key as trusted in your Pacman keyring.

## 6\. Synchronize Package Databases

Refresh the package databases.

bash

```bash
sudo pacman -Syy
```

**Explanation:**

- `-S` performs package synchronization.
- `-yy` forces a fresh download of repository databases.

This allows Pacman to retrieve package information from the newly configured Omarchy repository.

## 7\. Verify the Repository

Check whether the Omarchy repository is accessible.

bash

```bash
pacman -Sl omarchy
```

**Explanation:**

- `-S` performs synchronization-related operations.
- `-l` lists available packages.
- `omarchy` specifies the repository.

If the configuration is working correctly, you should see the packages available in the Omarchy repository.

## 8\. Install Omarchy Packages

You can now install packages provided by the Omarchy repository.

To install a package:

bash

```bash
sudo pacman -S PACKAGE_NAME
```

Replace `PACKAGE_NAME` with the actual package name available in the repository.

**Explanation:**

- `sudo` executes the command with administrator privileges.
- `pacman -S` installs the requested package.
- `PACKAGE_NAME` is the package you want to install.

**Note:** Installing individual packages is not the same as installing the complete Omarchy desktop environment. Follow the official installation documentation for a complete installation.

## Troubleshooting

### Repository database errors

Refresh the package databases:

bash

```bash
sudo pacman -Syy
```

### Check repository configuration

bash

```bash
grep -A3 '^\[omarchy\]' /etc/pacman.conf
```

### Check imported signing keys

bash

```bash
sudo pacman-key --list-keys
```

### Update the system

bash

```bash
sudo pacman -Syu
```

### Check repository availability

bash

```bash
pacman -Sl omarchy
```

**Important:** Never disable package signature verification to bypass signing errors. Verify the repository configuration and signing-key fingerprint instead.

## Official Links

- **Official Website:** [https://omarchy.org/](<https://omarchy.org/>)
- **Official Omarchy GitHub:** [https://github.com/omacom/omarchy](<https://github.com/omacom/omarchy>)
- **Official Omarchy Package Repository:** [https://github.com/omacom/omarchy-pkgs](<https://github.com/omacom/omarchy-pkgs>)
- **Official Omarchy Documentation:** [https://omarchy.org/manual/](<https://omarchy.org/manual/>)
- **Official Updates Documentation:** [https://omarchy.org/manual/updates/](<https://omarchy.org/manual/updates/>)
- **Omarchy Package Server:** https://pkgs.omarchy.org/
- **Arch Linux Wiki:** [https://wiki.archlinux.org/](<https://wiki.archlinux.org/>)
- **Pacman Documentation:** [https://man.archlinux.org/man/pacman.conf.5](<https://man.archlinux.org/man/pacman.conf.5>)

---

## Disclaimer

This is a community-written guide. Omarchy and its package repository are maintained by their respective developers.

Always refer to the official documentation for the latest repository configuration and installation instructions.

**Happy Linuxing\!** 🐧
