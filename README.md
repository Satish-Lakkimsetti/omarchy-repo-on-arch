# Add Omarchy Repository to Arch Linux

A beginner-friendly guide to adding the official Omarchy package repository to a regular Arch Linux installation.

All steps use copy-pasteable terminal commands. No manual configuration editing is required.

## Table of Contents

- [Introduction](#introduction)
- [Requirements](#requirements)
- [1. Update Arch Linux](#1-update-arch-linux)
- [2. Install Required Packages](#2-install-required-packages)
- [3. Check Existing Repository Configuration](#3-check-existing-repository-configuration)
- [4. Add the Omarchy Repository](#4-add-the-omarchy-repository)
- [5. Import the Signing Key](#5-import-the-signing-key)
- [6. Synchronize Package Databases](#6-synchronize-package-databases)
- [7. Verify the Repository](#7-verify-the-repository)
- [8. Install Omarchy Packages](#8-install-omarchy-packages)
- [Troubleshooting](#troubleshooting)
- [Official Links](#official-links)
- [Disclaimer](#disclaimer)

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

## 1. Update Arch Linux

First, update your Arch Linux installation.

```bash
sudo pacman -Syu
```

**Explanation:**

- `sudo` executes the command with administrator privileges.
- `pacman` is the Arch Linux package manager.
- `-Syu` synchronizes package databases and upgrades installed packages.

**Important:** Arch Linux does not support partial system upgrades. Always perform a full system upgrade when synchronizing package databases.

## 2. Install Required Packages

Install GnuPG, which provides the cryptographic tools required for signing-key management.

```bash
sudo pacman -S --needed gnupg
```

**Explanation:**

- `gnupg` provides GPG utilities for cryptographic key management.
- `--needed` avoids reinstalling packages that are already installed.

## 3. Check Existing Repository Configuration

Before adding the repository, check whether it is already configured.

```bash
grep -n '^\[omarchy\]' /etc/pacman.conf
```

**Explanation:**

- `grep` searches for matching text.
- `-n` displays the line number.
- `^\[omarchy\]` searches for the Omarchy repository section.

If the command displays `[omarchy]`, the repository is already configured. Do not append another copy.

If there is no output, continue to the next step.

## 4. Add the Omarchy Repository

Run the following command to automatically append the Omarchy repository configuration to `/etc/pacman.conf`.

```bash
if ! grep -q '^\[omarchy\]' /etc/pacman.conf; then
    sudo tee -a /etc/pacman.conf <<'EOF'

[omarchy]
SigLevel = Required DatabaseOptional
Server = https://pkgs.omarchy.org/stable/$arch
EOF
else
    echo "Omarchy repository is already configured."
fi
```

**Explanation:**

- `grep -q` checks whether the repository section already exists without displaying output.
- `if !` runs the configuration command only when the repository is not already configured.
- `sudo` executes the command with administrator privileges.
- `tee` writes the provided text to a file.
- `-a` appends the configuration without deleting existing contents.
- `/etc/pacman.conf` is the Pacman configuration file.
- `<<'EOF'` allows multiple lines to be written directly into the file.

No text editor is required!

**Important:** This command prevents adding another repository section if one already exists. If the repository was previously configured, inspect its existing settings rather than appending another copy.

### Verify the configuration

```bash
grep -A2 '^\[omarchy\]' /etc/pacman.conf
```

The output should contain the following three configuration lines:

- `[omarchy]`
- `SigLevel = Required DatabaseOptional`
- `Server = https://pkgs.omarchy.org/stable/$arch`

## 5. Import the Signing Key

Import the official Omarchy repository signing key.

```bash
sudo pacman-key --recv-keys 40DFB630FF42BCFFB047046CF0134EE680CAC571 --keyserver keys.openpgp.org
```

**Explanation:**

- `pacman-key` manages Pacman's trusted signing keys.
- `--recv-keys` retrieves the specified public key.
- `--keyserver` specifies the keyserver.
- The fingerprint identifies the signing key.

### Verify the signing key fingerprint

Before trusting the imported key, verify its fingerprint.

```bash
sudo pacman-key --finger 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

Compare the displayed fingerprint with the official Omarchy security documentation:

https://omarchy.org/manual/security/

Do not trust or locally sign the key if the fingerprint does not match.

### Locally sign the key

After verifying the fingerprint, locally sign the key.

```bash
sudo pacman-key --lsign-key 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

**Explanation:**

This locally marks the imported key as trusted in your Pacman keyring.

**Important:** Only locally sign the key after independently verifying its fingerprint against the official documentation.

## 6. Synchronize Package Databases

After adding the repository and importing its signing key, synchronize the package databases and upgrade the system.

```bash
sudo pacman -Syu
```

**Explanation:**

- `-S` performs package synchronization.
- `-y` refreshes package databases.
- `-u` upgrades installed packages.

This allows Pacman to retrieve package information from the newly configured Omarchy repository while keeping the system fully upgraded.

**Note:** A full system upgrade is important because Arch Linux does not support partial upgrades.

## 7. Verify the Repository

Check whether the Omarchy repository is accessible.

```bash
pacman -Sl omarchy
```

**Explanation:**

- `pacman` is the Arch Linux package manager.
- `-S` performs synchronization-related operations.
- `-l` lists available packages.
- `omarchy` specifies the repository.

If the configuration is working correctly, Pacman will display the packages available in the Omarchy repository.

If no packages are displayed or an error occurs, check the repository configuration and signing-key setup.

## 8. Install Omarchy Packages

You can now install packages provided by the Omarchy repository.

To install a package:

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

Refresh the package databases and perform a full system upgrade.

```bash
sudo pacman -Syu
```

If you encounter stale or corrupted repository database errors, try forcing a fresh database download.

```bash
sudo pacman -Syyu
```

### Check repository configuration

```bash
grep -A2 '^\[omarchy\]' /etc/pacman.conf
```

### Check the imported signing key

```bash
sudo pacman-key --finger 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

### Update the system

```bash
sudo pacman -Syu
```

### Check repository availability

```bash
pacman -Sl omarchy
```

**Important:** Never disable package signature verification or use `TrustAll` to bypass signing errors. Verify the repository configuration and signing-key fingerprint instead.

## Official Links

- **Official Website:** [Omarchy](https://omarchy.org/)
- **Official Omarchy GitHub:** [omacom/omarchy](https://github.com/omacom/omarchy)
- **Official Omarchy Package Repository:** [omacom/omarchy-pkgs](https://github.com/omacom/omarchy-pkgs)
- **Official Omarchy Documentation:** [Omarchy Manual](https://omarchy.org/manual/)
- **Official Security Documentation:** [Security](https://omarchy.org/manual/security/)
- **Official Updates Documentation:** [Updates](https://omarchy.org/manual/updates/)
- **Omarchy Package Server:** https://pkgs.omarchy.org/
- **Arch Linux Wiki:** [ArchWiki](https://wiki.archlinux.org/)
- **Pacman Documentation:** [pacman.conf](https://man.archlinux.org/man/pacman.conf.5)

---

## Disclaimer

This is a community-written guide. Omarchy and its package repository are maintained by their respective developers.

Always refer to the official documentation for the latest repository configuration and installation instructions.

**Happy Linuxing!** 🐧
