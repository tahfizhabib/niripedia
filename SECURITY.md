# Security

Niripedia is a guide, not software, but readers run its commands on their own machines, often as root. A bad command can damage a system or expose it.

## What counts

Please report any of the following:

- A command that could erase data or break the system when it's run as written, such as a wrong device name or an unsafe `rm`.
- A script the guide pipes into a shell (`curl ... | sh`) whose address has moved, been taken over or changed what it does.
- A package, repository or link that now points to something unexpected, such as a renamed AUR package taken over by someone else.
- A step that weakens the system's security without saying so, such as a permission change or a disabled protection.

A wrong package name, a typo or an outdated step isn't a security issue. Open a normal issue for those, as [CONTRIBUTING.md](CONTRIBUTING.md) describes.

## How to report

Open an issue that says you have a security concern and leave out the details. A maintainer will reply with a private way to send them. If the problem is simple and already public, such as a broken installer link, a normal issue with the section name is fine.

## What to expect

The maintainers aim to reply within a few days and to remove or fix a dangerous step as soon as it's confirmed. The fix will be listed in the [changelog](CHANGELOG.md).

## A note for readers

The guide installs software from the Arch repositories and the AUR. AUR packages are written by users and aren't reviewed by the Arch team, so open a package's page and read its `PKGBUILD` before you build it. Read any script before you pipe it into your shell.
