# Add the Omarchy Repository to Arch Linux

A beginner-friendly guide to adding the official Omarchy **stable** package repository to an existing, regular Arch Linux installation.

This guide covers repository configuration and package discovery. It does **not** install the complete Omarchy desktop environment.

> [!WARNING]
> Adding the Omarchy repository does **not** guarantee that every package in it is suitable for a regular Arch Linux installation. Some packages may be designed for Omarchy's own desktop environment, defaults, services, configuration, or package set. Review a package's description and dependencies before installing it. Installing individual Omarchy packages does not turn a regular Arch installation into a supported Omarchy installation.

All terminal code blocks below contain commands intended to be copied into a terminal. Explanations and descriptions of expected results are kept outside those command blocks.

## Table of Contents

- [Introduction](#introduction)
- [Compatibility and safety](#compatibility-and-safety)
- [Requirements](#requirements)
- [1. Update Arch Linux](#1-update-arch-linux)
- [2. Install required packages](#2-install-required-packages)
- [3. Check the existing repository configuration](#3-check-the-existing-repository-configuration)
- [4. Add the Omarchy repository](#4-add-the-omarchy-repository)
- [5. Import and verify the signing key](#5-import-and-verify-the-signing-key)
- [6. Synchronize package databases](#6-synchronize-package-databases)
- [7. Verify repository access](#7-verify-repository-access)
- [8. Install an Omarchy package](#8-install-an-omarchy-package)
- [Troubleshooting](#troubleshooting)
- [Official links](#official-links)
- [Disclaimer](#disclaimer)

## Introduction

Omarchy maintains a package repository for Arch Linux. Adding it to Pacman's configuration makes packages published in the repository available for discovery and installation through Pacman.

This guide configures the **stable** channel at `https://pkgs.omarchy.org/stable/$arch`.

**Note:** Adding the repository does not automatically install Omarchy or its complete desktop environment.

## Compatibility and safety

Before proceeding, keep the following points in mind:

- **Package compatibility varies.** A package being listed in the Omarchy repository does not mean it is appropriate for every regular Arch Linux setup.
- **Check dependencies and purpose.** Review the package description, dependencies, and any Omarchy-specific requirements before installing it.
- **Expect system-wide changes.** Pacman may install dependencies or update packages as part of a transaction. Read the proposed transaction before confirming it.
- **Understand repository priority.** Pacman gives preference to repositories appearing earlier in `/etc/pacman.conf` when the same package name exists in multiple repositories, regardless of version. This guide appends `[omarchy]` after existing repositories, but changing repository order or availability may affect future upgrades. Review the selected package sources and every proposed upgrade.
- **Keep signature verification enabled.** Do not disable package signature checks to work around key or signature errors.
- **Use full system upgrades.** Arch Linux does not support partial upgrades. When refreshing package databases, use a full upgrade rather than synchronizing databases and installing packages separately.

If you only need a particular application, consider whether it is available from the official Arch repositories or another source that better matches your system before adding a third-party repository.

## Requirements

- A working Arch Linux installation.
- An active internet connection.
- A user account with `sudo` privileges.
- Basic terminal access.

## 1. Update Arch Linux

Start by fully updating your existing Arch Linux installation.

```bash
sudo pacman -Syu
```

**Explanation:**

- `sudo` runs the command with administrator privileges.
- `pacman` is the Arch Linux package manager.
- `-Syu` synchronizes package databases, refreshes package information, and upgrades installed packages.

If the upgrade fails, resolve the error before continuing. Do not proceed with package installation while a system upgrade is incomplete.

## 2. Install required packages

Install GnuPG, which provides the cryptographic tools used by Pacman's key-management utilities.

```bash
sudo pacman -S --needed gnupg
```

**Explanation:**

- `gnupg` provides GPG utilities for cryptographic key management.
- `--needed` avoids reinstalling packages that are already installed.

The `curl` package is not required for the repository configuration steps in this guide.

## 3. Check the existing repository configuration

Check whether an Omarchy repository section is already present in `/etc/pacman.conf`.

```bash
grep -n '^\[omarchy\]' /etc/pacman.conf
```

**Explanation:**

- `grep` searches for matching text.
- `-n` displays the matching line number.
- `^\[omarchy\]` matches a repository section named `[omarchy]`.

If the command prints a matching line, the section already exists. Do not add a second copy. Inspect the existing section and confirm that it points to the intended Omarchy channel.

If there is no output, continue to the next step.

## 4. Add the Omarchy repository

### Back up the Pacman configuration

Before modifying `/etc/pacman.conf`, create a timestamped backup:

```bash
sudo cp -a /etc/pacman.conf "/etc/pacman.conf.backup.$(date +%Y%m%d-%H%M%S)"
```

This preserves the original configuration and its file attributes. The backup is saved next to `/etc/pacman.conf` with a timestamp in its filename. **If this command fails, stop and resolve the error before continuing.**

### Append the repository safely

The following command appends the Omarchy stable repository configuration only if a section named `[omarchy]` is not already present.

```bash
if ! grep -q '^\[omarchy\]' /etc/pacman.conf; then
    sudo tee -a /etc/pacman.conf <<'EOF'

[omarchy]
SigLevel = Required DatabaseOptional
Server = https://pkgs.omarchy.org/stable/$arch
EOF
else
    echo "An [omarchy] repository section already exists. Inspect it before making changes."
fi
```

**Explanation:**

- `grep -q` checks for the section without printing a match.
- The `if` statement adds the repository only when the section is absent.
- `sudo tee -a` appends the configuration to `/etc/pacman.conf`.
- `<<'EOF'` passes the configuration lines to `tee` without expanding `$arch`.
- `SigLevel = Required DatabaseOptional` requires package signatures while allowing the repository database itself to be unsigned.

**Important:** This check prevents a second section with the exact name `[omarchy]`. It does not validate an existing section's settings. If the section already exists, inspect it instead of blindly adding or changing another entry.

### Verify the configuration

```bash
grep -A2 '^\[omarchy\]' /etc/pacman.conf
```

The output should show the `[omarchy]` section followed by these settings:

- `SigLevel = Required DatabaseOptional`
- `Server = https://pkgs.omarchy.org/stable/$arch`

If the existing section has different settings, compare them with the official Omarchy documentation before proceeding.

## 5. Import and verify the signing key

Import the public signing key used by the Omarchy package repository.

```bash
sudo pacman-key --recv-keys 40DFB630FF42BCFFB047046CF0134EE680CAC571 --keyserver keys.openpgp.org
```

**Explanation:**

- `pacman-key` manages Pacman's keyring.
- `--recv-keys` retrieves the specified public key.
- `--keyserver` selects the keyserver.
- The long hexadecimal value is the signing-key fingerprint.

### Verify the fingerprint

Before trusting the key, display its fingerprint:

```bash
sudo pacman-key --finger 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

Compare the complete fingerprint with the one published in the [official Omarchy security documentation](https://omarchy.org/manual/security/) and verify it through the source linked by Omarchy, [OpenPGP.org](https://keys.openpgp.org/).

The expected fingerprint is:

`40DFB630FF42BCFFB047046CF0134EE680CAC571`

**Do not locally sign the key if the fingerprint does not match.** If you cannot independently verify it, stop here.

### Locally sign the verified key

Only after confirming the fingerprint, locally sign the key so Pacman trusts packages signed by it:

```bash
sudo pacman-key --lsign-key 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

**Explanation:**

This adds local trust for the verified key to your Pacman keyring. It does not replace the need to verify the fingerprint first.

## 6. Synchronize package databases

Refresh repository databases and perform a full system upgrade:

```bash
sudo pacman -Syu
```

**Explanation:**

- `-S` performs package synchronization.
- `-y` refreshes package databases.
- `-u` upgrades installed packages.

Arch Linux is a rolling-release distribution and does not support partial upgrades. Avoid running `pacman -Sy` followed by a separate package installation. Use a full upgrade with `pacman -Syu`.

## 7. Verify repository access

List the packages available in the Omarchy repository:

```bash
pacman -Sl omarchy
```

If the repository is configured and accessible, Pacman should list its available packages.

If the command reports a repository, database, network, or signature error, consult the [Troubleshooting](#troubleshooting) section before proceeding.

## 8. Install an Omarchy package

First, choose a package from the list shown in [Step 7](#7-verify-repository-access). To inspect its metadata, use `pacman -Si omarchy/PACKAGE_NAME`, replacing `PACKAGE_NAME` with the real package name.

When you have reviewed the package and confirmed it is suitable for your system, install it with `sudo pacman -S omarchy/PACKAGE_NAME`, again replacing `PACKAGE_NAME` with the actual name.

**These are command templates, not ready-to-run commands.** They are intentionally shown as inline text rather than terminal code blocks so that nobody accidentally copies an unmodified placeholder into a terminal. The `omarchy/` prefix explicitly selects the Omarchy repository if another repository has a package with the same name.

**Before confirming the transaction:**

- Review the package description and dependencies.
- Check whether it expects Omarchy-specific services, settings, or components.
- Read Pacman's proposed transaction carefully.
- Cancel if the changes are unexpected or you are unsure about compatibility.

**Note:** Installing individual packages is not the same as installing the complete Omarchy desktop environment. For a complete Omarchy installation, follow the [official installation documentation](https://omarchy.org/).

## Troubleshooting

### The repository is already configured

Check the existing section:

```bash
grep -A2 '^\[omarchy\]' /etc/pacman.conf
```

Do not add another `[omarchy]` section. If the server or settings differ, compare them with the official documentation before editing the configuration.

### Repository database or download errors

Check your internet connection and repository configuration. Then perform a full synchronization and upgrade:

```bash
sudo pacman -Syu
```

If you specifically need to force a fresh download of all repository databases, use:

```bash
sudo pacman -Syyu
```

Use `-Syyu` only when necessary, such as when troubleshooting stale or inconsistent repository databases. It forces database downloads and still performs a full upgrade.

### Signing-key or signature errors

Display the key fingerprint again:

```bash
sudo pacman-key --finger 40DFB630FF42BCFFB047046CF0134EE680CAC571
```

Compare it with the official Omarchy security documentation. Confirm that the key was locally signed only after successful verification.

Never disable signature verification to bypass an error.

### Check repository availability

```bash
pacman -Sl omarchy
```

If the repository cannot be reached, check the server address, network connection, and Pacman error message.

### Update the system

```bash
sudo pacman -Syu
```

If a full upgrade fails, resolve the reported issue before attempting further package operations.

## Official links

- **Official Website:** [Omarchy](https://omarchy.org/)
- **Official Omarchy GitHub:** [omacom/omarchy](https://github.com/omacom/omarchy)
- **Official Omarchy Package Repository:** [omacom/omarchy-pkgs](https://github.com/omacom/omarchy-pkgs)
- **Official Omarchy Manual:** [Omarchy Documentation](https://omarchy.org/manual/)
- **Official Security Documentation:** [Signing Keys and Security](https://omarchy.org/manual/security/)
- **Official Updates Documentation:** [Updates](https://omarchy.org/manual/updates/)
- **Omarchy Package Server:** [pkgs.omarchy.org](https://pkgs.omarchy.org/)
- **Arch Linux Wiki:** [ArchWiki](https://wiki.archlinux.org/)
- **Pacman Configuration Manual:** [pacman.conf](https://man.archlinux.org/man/pacman.conf.5)
- **Arch Linux System Maintenance:** [Partial upgrades](https://wiki.archlinux.org/title/System_maintenance#Partial_upgrades_are_unsupported)

## Disclaimer

This is a community-written guide and is not an official Omarchy installation guide.

Omarchy and its package repository are maintained by their respective developers. Repository availability, package contents, dependencies, and configuration may change over time.

Adding the repository does not guarantee that every package is compatible with, or appropriate for, a regular Arch Linux installation. Review packages carefully and consult the official documentation before installing them.

Always refer to the official documentation for the latest repository configuration and installation instructions.

**Happy Linuxing!** 🐧
