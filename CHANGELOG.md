# Changelog

All notable changes to Niripedia are listed here, newest first. The format follows [Keep a Changelog](https://keepachangelog.com).

## Unreleased

### Added

- License ([MIT](LICENSE)), [code of conduct](CODE_OF_CONDUCT.md), [contributing guide](CONTRIBUTING.md) and [security policy](SECURITY.md).
- Issue form for wrong or outdated steps, and a pull request template.
- Troubleshooting entries for first-boot network problems, Niri config errors, Polkit prompts, OBS screen capture and Spicetify after Spotify updates.
- A section on updating the system and removing each piece of the setup.

### Changed

- Rewrote the guide in a plainer style, with an explanation at the start of every section.
- Niri steps now use `niri msg outputs` and `niri validate` in place of tools that don't work with Niri.
- The Polkit agent now starts from the Niri config, because Niri doesn't read XDG autostart files.
- Merged the duplicate app lists and corrected package names (`fuzzel`, `gnome-disk-utility`, `udisks2`).
- Spotify scaling now uses the `spotify-launcher` desktop file.

### Removed

- All Hyprland content.
- The Spotify path settings from the Spicetify steps.

### Notes

- The Noctalia section covers the old shell, v4. A newer Noctalia has been released and isn't covered yet.
