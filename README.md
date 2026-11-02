<p align="center">
  <img src="https://img.shields.io/badge/distro-Arch%20Linux-1793D1" alt="Arch Linux">
  <img src="https://img.shields.io/badge/compositor-Niri-5B7FFF" alt="Niri">
  <img src="https://img.shields.io/badge/shell-Noctalia-8A2BE2" alt="Noctalia Shell">
  <img src="https://img.shields.io/badge/login-SDDM-3DDC84" alt="SDDM">
  <img src="https://img.shields.io/badge/setup-manual-lightgrey" alt="Manual setup">
</p>

# Niripedia

Niripedia is the guide to Nirvic, my daily Arch Linux setup, and it shows how I get from the Arch ISO to a working Niri desktop. You install Arch with the official installer, add Niri and a shell on top, and finish with the fonts, icons and apps I use every day. Everything is done by hand for now. My full dotfiles aren't finished, and I'd rather keep the system clean and easy to change than automate something half done.

A small dotfiles set called **niriha** is the lightweight version of this setup. Use it as a temporary environment or a starting point before moving to the full Nirvic configuration.

This is a personal setup, not a distribution. Monitor names, package choices and paths are mine, so change them to fit your machine. The sections up to [Niri](#niri) go in order, because each one needs the one before it. After that every section stands on its own, so skip the ones you don't use.

This guide has no connection to Arch Linux, Niri, Noctalia or any other project it mentions. None of them endorse it, and none of them wrote it.

[Install Arch](#install-arch) | [Set up Niri](#niri) | [Troubleshooting](#troubleshooting)

> [!IMPORTANT]
> `archinstall` can erase a whole disk. Back up anything you care about before you start, and read the disk step twice.

> [!WARNING]
> The Noctalia in this guide is the old shell, Noctalia v4. A newer Noctalia has been released since, and its install and setup steps differ. Check the [Noctalia documentation](https://docs.noctalia.dev) before you follow the [Noctalia Shell](#noctalia-shell) section, and don't assume the commands here still apply to the new version.

## Contents

1. [Before you start](#before-you-start)
2. [Overview](#overview)
3. [Install Arch](#install-arch)
4. [AUR helper](#aur-helper)
5. [Niri](#niri)
6. [Launchers](#launchers)
7. [Noctalia Shell](#noctalia-shell)
8. [Polkit and drives](#polkit-and-drives)
9. [Fonts](#fonts)
10. [Icons](#icons)
11. [Apps](#apps)
12. [OBS Studio](#obs-studio)
13. [LazyVim](#lazyvim)
14. [Spotify and Spicetify](#spotify-and-spicetify)
15. [Updating and removing](#updating-and-removing)
16. [Troubleshooting](#troubleshooting)
17. [Credits](#credits)

## Before you start

You need a PC that boots in UEFI mode, a USB stick big enough for the Arch ISO, and a working internet connection, because Arch downloads its packages during the install. A second device with a browser helps, so you can read this guide while the installer is running. You should be comfortable typing commands in a terminal. Nothing here is difficult, but Arch won't ask you twice before it does what you typed.

The install wipes at least one partition, and it wipes the whole disk if you pick that option. If this machine already has Windows or another system on it that you want to keep, shrink its partition first and choose **Use existing partition** in the installer.

## Overview

The setup is four layers. The login screen starts the compositor, the compositor runs your shell, and the shell gives you the bar and the notifications. A launcher opens apps on top of that.

| Piece | What it is |
|---|---|
| [Niri](https://github.com/YaLTeR/niri) | A scrollable-tiling Wayland compositor with smooth animations and a small, plain-text config. It's the foundation of this setup. |
| [Noctalia Shell](https://github.com/noctalia-dev/noctalia-shell) | A Wayland shell that draws the bar, panels, widgets and notifications, and handles theming. It replaces Waybar and Mako, so you don't run them together. |
| [Vicinae](https://github.com/vicinaehq/vicinae) | An application launcher that runs as a background server and opens with a keybinding. [Walker](#walker) is the lighter alternative. |
| SDDM | The login screen. It starts Niri as a session. |
| PipeWire | Audio and screen capture. Archinstall installs it when you pick it under Audio. |

When you finish, you have a clean Arch system that boots to a login screen and opens Niri, with a shell, a launcher, good fonts, an icon theme and the apps from the [Apps](#apps) section. Nothing else is installed behind your back.

## Install Arch

You install Arch with the official ISO and the built-in `archinstall` script. The script asks for your choices in menus and then installs the system in one go, so you don't partition or run `pacstrap` yourself.

1. Download the ISO from [archlinux.org/download](https://archlinux.org/download/).
2. Write it to a USB stick. On Windows and macOS use balenaEtcher. On Linux use `dd`, and check the device name with `lsblk` first, because the wrong one gets erased:

   ```bash
   sudo dd if=archlinux-x86_64.iso of=/dev/sdX bs=4M status=progress oflag=sync
   ```

3. Boot from the USB stick. Turn off **Secure Boot** in your firmware settings first, because the Arch ISO isn't signed for it. You open the firmware settings with a key at power-on, usually F2, F12 or Delete.
4. Connect to the network. Ethernet works by itself. For Wi-Fi, use `iwctl`:

   ```bash
   iwctl
   device list
   station wlan0 scan
   station wlan0 get-networks
   station wlan0 connect "<your network>"
   exit
   ```

   Check the connection with `ping -c 3 archlinux.org`. Replace `wlan0` with the name `device list` shows.
5. Run `archinstall` and fill it in with the table below.

| Section | What I choose |
|---|---|
| Mirrors and repositories | Select regions, then your country. A nearby mirror makes the install faster. |
| Disk configuration | **Use entire disk** erases everything. **Use existing partition** overwrites only the partition you pick. |
| Disk, file system | btrfs for the default subvolumes and compression, or ext4 for a plain layout. ext4 needs no extra partitioning. |
| Disk, encryption | None, or LUKS with a password and the partitions you select. With LUKS you type the password at every boot. |
| Hostname | Any name you like. |
| Bootloader | systemd-boot, GRUB or Limine. All three work with this guide. |
| Authentication, root password | Set one. |
| Authentication, user account | Add a user and give it superuser access. You'll use this user, not root, for everything after the install. |
| Profile | Minimal, so no desktop gets installed. Niri comes later by hand. |
| Applications, audio | pipewire |
| Additional packages | `git base-devel`, which the [AUR helper](#aur-helper) needs. |
| Network configuration | Copy the ISO network configuration, or NetworkManager if you switch Wi-Fi networks often. |
| Timezone | Your location. |

When it finishes, reboot, remove the USB stick and log in at the TTY with the user you created, not root. A TTY is the plain text login screen, and it's where the rest of this guide starts.

## AUR helper

The Arch User Repository (AUR) holds packages that the official repositories don't, such as Vicinae, Noctalia and Vesktop. A helper builds them for you, handles their dependencies and updates them along with everything else. You only need one, and you build it as your normal user, never as root. It needs the `git` and `base-devel` packages from the install step.

**yay**

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd .. && rm -rf yay
```

**paru**

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
cd .. && rm -rf paru
```

Both take the same arguments as pacman, so they install from the official repositories and the AUR with one command:

```bash
yay -S <package>   # install
yay -Syu           # update everything, official and AUR
```

The rest of this guide uses `yay`. If you chose paru, swap the command name. AUR packages are written by users and not checked by the Arch team, so skim a package's page before you build it.

## Niri

This section takes you from a bare TTY to a login screen and a working Niri session. Niri is a Wayland compositor, which means it draws your windows and handles input. It doesn't come with a bar, a launcher or a wallpaper, so the next sections add those.

### 1. Install Niri

```bash
sudo pacman -S niri
```

For the development build from the AUR, use `yay -S niri-git` instead. It gets new features sooner and breaks more often.

### 2. Install the base tools

```bash
sudo pacman -S kitty fuzzel swaybg otf-font-awesome ttf-jetbrains-mono xwayland-satellite
```

| Package | What it does |
|---|---|
| `kitty` | Terminal emulator. Niri's default config opens it with Mod+T. |
| `fuzzel` | Simple application launcher. Skip it if you use [Vicinae](#launchers). |
| `swaybg` | Wallpaper setter. Skip it if you use [Noctalia Shell](#noctalia-shell), which handles wallpapers itself. |
| `otf-font-awesome`, `ttf-jetbrains-mono` | Icon and coding fonts. |
| `xwayland-satellite` | Lets X11 apps run under Niri. |

### 3. Install and enable SDDM

```bash
sudo pacman -S sddm
sudo systemctl enable sddm.service --now
```

`--now` starts SDDM immediately and ends your TTY session. At the login screen, choose the **Niri** session, enter your password and log in. SDDM remembers the session you picked, so you only choose it once.

### 4. Set up your display

Niri writes a default config to `~/.config/niri/config.kdl` the first time it runs. List your outputs and their exact modes:

```bash
niri msg outputs
```

Then set yours in the config. This example is a 1920x1080 monitor at 144 Hz:

```kdl
output "DP-1" {
    mode "1920x1080@144.000"
    scale 1
}
```

Replace `DP-1` with your output name and the mode with one `niri msg outputs` lists. Use the refresh rate exactly as it's listed, because Niri matches it to the monitor's real modes. Niri reloads the config when you save it, so a display change shows up at once. `niri validate` checks the file for errors.

### 5. Start apps with Niri

Programs that should run at login go in the same file:

```kdl
spawn-at-startup "swaybg" "-i" "/path/to/wallpaper.png"
spawn-at-startup "waybar"
spawn-at-startup "mako"
```

Waybar is the status bar and Mako is the notification daemon. If you use [Noctalia Shell](#noctalia-shell), leave out all three lines, because it replaces them. Startup lines run once when Niri starts, so a new one waits for your next login. Log out and back in to try it.

## Launchers

A launcher is the box you open with a keybinding to start apps and run quick actions. You need one, so pick either Vicinae or Walker. The two do the same job, and the table shows where they differ.

| | Vicinae | Walker |
|---|---|---|
| How it runs | A single server, started with Niri. | A launcher on top of the Elephant backend, which supplies its providers. |
| Extras | Has its own extension system. | Features come from separate Elephant packages. |
| Weight | More features, more to load. | Lighter and modular. |

I use Vicinae. If you install it, you can skip the Walker part.

### Vicinae

```bash
yay -S vicinae-bin
```

The project's installer script also works, but read a script before you pipe it to your shell:

```bash
curl -fsSL https://vicinae.com/install.sh | bash
```

Add the server and a toggle key to `~/.config/niri/config.kdl`. The keybinding goes inside the existing `binds { }` block:

```kdl
spawn-sh-at-startup "vicinae server"

binds {
    Mod+Space repeat=false { spawn "vicinae" "toggle"; }
}
```

`spawn-sh-at-startup` runs the launcher server at login, and `Mod+Space` shows or hides it. The first press after a fresh login may be slow while the server starts. The [Vicinae documentation](https://docs.vicinae.com) covers themes and extensions.

### Walker

Skip this if you installed Vicinae. Install the Elephant providers, then Walker:

```bash
yay -S elephant elephant-bluetooth elephant-calc elephant-clipboard \
  elephant-desktopapplications elephant-files elephant-menus elephant-providerlist \
  elephant-symbols elephant-todo elephant-unicode elephant-websearch
yay -S walker
```

Each Elephant package adds one kind of result, such as apps, files, a calculator, clipboard history or emoji. Install only the ones you want. Enable Elephant as a user service and start it without a reboot:

```bash
elephant service enable
systemctl --user start elephant.service
```

Then bind Walker in the `binds { }` block of your Niri config:

```kdl
binds {
    Mod+Space repeat=false { spawn "walker"; }
}
```

## Noctalia Shell

> [!WARNING]
> This section covers the old Noctalia shell, v4. The new Noctalia isn't covered here, and its commands and settings location may be different.

Noctalia draws your bar, panels, widgets and notifications and handles theming. It replaces Waybar, Mako and a separate wallpaper tool, so don't start those next to it. Without it, Niri is just windows on a blank background, which some people prefer. Add Noctalia when you want a complete desktop.

### 1. Install

```bash
yay -S noctalia-shell
```

`noctalia-shell-git` is the development build. Pick one of the two.

### 2. Start it with Niri

Add this to `~/.config/niri/config.kdl`, then log out and back in:

```kdl
spawn-at-startup "qs" "-c" "noctalia-shell"
```

The launch command has changed between Noctalia releases, so check the [Noctalia documentation](https://docs.noctalia.dev) if the shell doesn't appear. If your package ships a user service, this works too, but use only one of the two methods:

```bash
systemctl --user enable --now noctalia.service
systemctl --user status noctalia.service
```

### 3. Configure it

Noctalia keeps its settings in `~/.config/noctalia/` and you change them from its own settings panel, not from `config.kdl`. Niri's config only decides how the shell starts. Colors, the bar layout and the widgets all live in the panel.

## Polkit and drives

Without a Polkit agent, graphical apps can't ask for your password. Mounting a drive, partitioning in GNOME Disks and similar tasks then fail without a clear error. Niri doesn't start an agent for you, so you add one.

### Polkit agent

```bash
sudo pacman -S polkit-gnome
```

Niri doesn't read XDG autostart files, so start the agent from its config:

```kdl
spawn-at-startup "/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1"
```

Log out and back in. To test it without logging out, run the same path in a terminal with `&` at the end. The next time an app needs admin rights, a password window appears.

### Secondary drives

```bash
sudo pacman -S udisks2 gvfs gnome-disk-utility
```

`udisks2` and `gvfs` let your file manager mount drives with one click. GNOME Disks lets you manage partitions. To mount a drive at every boot, find its UUID, make a mount point and add a line to `/etc/fstab`:

```bash
lsblk -f
sudo mkdir -p /mnt/data
sudo nano /etc/fstab
```

```text
UUID=<your-uuid>  /mnt/data  ext4  defaults,nofail  0  2
```

`nofail` lets the system boot even when the drive is missing. Use the right type for your file system, such as `btrfs` or `ntfs3`. Test the line with `sudo mount -a` before you reboot, because a broken fstab can stop the system booting.

## Fonts

Install a broad font set so text, symbols and emoji render properly. Wayland apps pick fonts through Fontconfig, and a missing font shows up as an empty box.

```bash
sudo pacman -S noto-fonts noto-fonts-emoji noto-fonts-extra
sudo pacman -S ttf-liberation ttf-roboto ttf-jetbrains-mono ttf-jetbrains-mono-nerd ttf-fira-code ttf-hack
yay -S ttf-symbola
```

`ttf-jetbrains-mono-nerd` supplies the icon glyphs that terminals and [LazyVim](#lazyvim) expect, and `ttf-symbola` covers symbols the Noto fonts miss.

Rebuild the font cache and check that your language is covered. The example checks Bangla, so use your own language code:

```bash
fc-cache -fv
fc-list :lang=bn
```

If your language's text still looks wrong, you can remove the default fonts that Fontconfig may pick first:

```bash
sudo pacman -Rns ttf-dejavu gnu-free-font
```

Check what pacman says before you confirm. Other packages can depend on these fonts, and pacman refuses to remove them in that case. Running apps keep the fonts they started with, so log out and back in after changing fonts.

## Icons

Papirus is the icon theme I use. It comes in light and dark variants, plus a tool that recolors folders. Install it first, then tell your apps to use it.

### 1. Install and apply

```bash
sudo pacman -S papirus-icon-theme nwg-look
```

Niri has no settings app, so use `nwg-look`. Open it, go to **Icon theme**, select **Papirus-Dark** or **Papirus-Light**, and apply. Apps that are already open pick up the change when you restart them.

### 2. Change folder colors

```bash
yay -S papirus-folders
papirus-folders -C <color> --theme Papirus-Dark
```

Use `Papirus-Light` for the light variant. Available colors:

```text
adwaita  black  blue  bluegrey  breeze  brown  carmine  cyan  darkcyan
deeporange  green  grey  indigo  magenta  nordic  orange  palebrown
paleorange  pink  red  teal  violet  white  yaru  yellow
```

See [previews of every color](https://github.com/PapirusDevelopmentTeam/papirus-folders) in the project's repository. To go back to the default blue, run `papirus-folders -D --theme Papirus-Dark`.

## Apps

These are the apps I install on a fresh system. `yay` pulls from both the official repositories and the AUR, so one command does it.

```bash
yay -S firefox zed neovim spotify-launcher vesktop-bin telegram-desktop \
  btop mpv vlc pacseek nwg-look nwg-displays \
  gnome-calendar gnome-calculator gnome-disk-utility
```

| App | What it does |
|---|---|
| [Firefox](https://archlinux.org/packages/extra/x86_64/firefox/) | Web browser. |
| [Zed](https://archlinux.org/packages/extra/x86_64/zed/) | Code editor. |
| [Neovim](https://archlinux.org/packages/extra/x86_64/neovim/) | Terminal editor. See [LazyVim](#lazyvim). |
| [spotify-launcher](https://archlinux.org/packages/extra/x86_64/spotify-launcher/) | Installs and starts the official Spotify client. See [Spotify and Spicetify](#spotify-and-spicetify). |
| [Vesktop](https://aur.archlinux.org/packages/vesktop-bin) | Discord client with Wayland screen sharing. |
| [Telegram](https://archlinux.org/packages/extra/x86_64/telegram-desktop/) | Telegram client. |
| [btop](https://archlinux.org/packages/extra/x86_64/btop/) | Terminal resource monitor. |
| [mpv](https://archlinux.org/packages/extra/x86_64/mpv/) | Light media player. |
| [VLC](https://archlinux.org/packages/extra/x86_64/vlc/) | Full-featured media player. |
| [Pacseek](https://aur.archlinux.org/packages/pacseek) | Terminal front end for searching packages. |
| [nwg-look](https://archlinux.org/packages/extra/x86_64/nwg-look/) | GTK theme, icon and font settings. |
| [nwg-displays](https://aur.archlinux.org/packages/nwg-displays) | Graphical display settings for Wayland. |
| [GNOME Calendar](https://archlinux.org/packages/extra/x86_64/gnome-calendar/) | Events and appointments. |
| [GNOME Calculator](https://archlinux.org/packages/extra/x86_64/gnome-calculator/) | Standard and scientific calculator. |
| [GNOME Disks](https://archlinux.org/packages/extra/x86_64/gnome-disk-utility/) | Disk and partition manager. Needs the [Polkit agent](#polkit-and-drives). |

OBS Studio has its own [section](#obs-studio), because screen capture needs extra setup.

## OBS Studio

OBS records and streams. On Wayland it can't grab the screen by itself. It asks the desktop portal, and the portal hands it a PipeWire stream, so those pieces need to be in place first.

### 1. Install

```bash
sudo pacman -S obs-studio pipewire pipewire-pulse wireplumber xdg-desktop-portal xdg-desktop-portal-gnome
```

| Package | What it does |
|---|---|
| `pipewire`, `pipewire-pulse` | Audio and video routing, with PulseAudio compatibility. |
| `wireplumber` | Session manager for PipeWire. |
| `xdg-desktop-portal`, `xdg-desktop-portal-gnome` | Let apps ask for screen capture. Niri uses the GNOME portal for this. |

### 2. Enable the services

These are usually on already, since archinstall sets up PipeWire. Running the command is harmless if so:

```bash
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

### 3. Capture the screen

Start OBS with `obs`, add a **Screen Capture (PipeWire)** source and pick a screen or window in the portal dialog that appears. The dialog shows up every time you add the source, and that's expected, because Wayland makes you approve each capture.

## LazyVim

[LazyVim](https://www.lazyvim.org) is a ready-made Neovim setup built on lazy.nvim. It gives you fast startup, lazy-loaded plugins and sensible defaults, so you get a working editor without writing a config first.

### 1. Install the requirements

```bash
sudo pacman -S --needed neovim git ripgrep fd lazygit
```

You also need a Nerd Font in your terminal. `ttf-jetbrains-mono-nerd` from the [Fonts](#fonts) section works.

### 2. Back up your current config

Skip this on a fresh install. If you already have a Neovim setup, this moves it aside, so nothing is lost:

```bash
mv ~/.config/nvim{,.bak}
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

### 3. Install LazyVim

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
nvim
```

The first launch installs the plugins, which takes a minute. Afterwards, run `:LazyHealth` inside Neovim to check that everything is in place.

## Spotify and Spicetify

### Install Spotify

```bash
sudo pacman -S spotify-launcher
```

`spotify-launcher` downloads the official client into your home folder on first run, so Spotify's files stay writable without root. Start it from your launcher the first time and give it a moment to download.

### Fix scaling

On high-DPI or unusual screens Spotify can come up too large or too small. You can fix that with a copy of its launcher entry, which leaves the system file alone.

1. Copy the entry:

   ```bash
   mkdir -p ~/.local/share/applications
   cp /usr/share/applications/spotify-launcher.desktop ~/.local/share/applications/
   ```

2. Open the copy and change the `Exec` line:

   ```bash
   nano ~/.local/share/applications/spotify-launcher.desktop
   ```

   ```text
   Exec=env GDK_SCALE=1 GDK_DPI_SCALE=0.6667 spotify-launcher %U
   ```

   The `0.6667` value is mine, so raise or lower it until the size suits your screen.
3. Save with **Ctrl+O**, press **Enter**, and exit with **Ctrl+X**.

Anything you start from your launcher now uses the new scale. Starting `spotify-launcher` by hand in a terminal doesn't read this file.

### Spicetify

[Spicetify](https://spicetify.app) is a command-line tool that customizes the official Spotify client. It can change colors, inject CSS and add extensions and custom apps. Install it and apply it to Spotify:

```bash
curl -fsSL https://raw.githubusercontent.com/spicetify/cli/main/install.sh | sh
spicetify backup apply
```

`backup apply` saves a copy of Spotify's original files and then applies Spicetify. Run it again after every Spotify update, because updates remove the changes. If Spicetify can't find Spotify, its [getting started guide](https://spicetify.app/docs/getting-started) explains the Linux setup, and the [basic usage](https://spicetify.app/docs/getting-started#basic-usage) page covers themes and extensions.

## Updating and removing

Arch is a rolling release, so you update the whole system at once. This updates the official packages and the AUR ones together:

```bash
yay -Syu
```

Never run `sudo pacman -Sy` on its own and then install a package. Arch doesn't support partial upgrades, and they can leave the system broken. After a kernel update, reboot so the new kernel runs. Before big updates, read the [Arch news](https://archlinux.org/news/), because manual steps are posted there.

Each piece of this setup comes out cleanly:

| To remove | Do this |
|---|---|
| Noctalia Shell | `yay -Rns noctalia-shell`, then delete its `spawn-at-startup` line from the Niri config. |
| Vicinae | `yay -Rns vicinae-bin`, then delete its startup line and keybinding. |
| Walker | `yay -Rns walker` and the `elephant` packages, then delete its keybinding. |
| Papirus folder colors | `papirus-folders -D --theme Papirus-Dark` resets them. `yay -Rns papirus-folders` removes the tool. |
| Spicetify | `spicetify restore` puts Spotify back to normal. |
| LazyVim | Restore the `.bak` folders from the install step, or delete `~/.config/nvim`. |

## Troubleshooting

### No network after the first reboot

Wi-Fi settings only carry over if you chose **Copy ISO network configuration**. In that case `iwctl` works the same way as it did on the ISO. Otherwise, pick NetworkManager in `archinstall` next time. Ethernet needs no setup either way.

### Niri isn't in the SDDM session list

Check that Niri is installed and that `/usr/share/wayland-sessions/niri.desktop` exists. Reinstall `niri` if it doesn't. Then restart SDDM with `sudo systemctl restart sddm`.

### Niri shows an error or ignores my config

Run `niri validate` in a terminal. It prints the line and the reason for the first error in `config.kdl`. Niri keeps your last working config until the file is valid again.

### My monitor doesn't use the resolution or refresh rate I set

The mode string has to match one from `niri msg outputs` exactly, refresh rate included, such as `1920x1080@144.000`. The output name in quotes must match too.

### Authentication prompts never appear

The Polkit agent isn't running. Check with `pgrep -a polkit-gnome` and see [Polkit and drives](#polkit-and-drives) for the startup line.

### OBS shows a black screen or no screen source

Check that the portal services are running with `systemctl --user status xdg-desktop-portal xdg-desktop-portal-gnome`. Restart them with `systemctl --user restart xdg-desktop-portal`, then reopen OBS and add the PipeWire screen source again.

### Two bars or two notification daemons show up

Noctalia Shell is running next to Waybar or Mako. Remove the `spawn-at-startup` lines for the older ones from `~/.config/niri/config.kdl`.

### Spicetify stopped working after a Spotify update

Updates restore Spotify's original files. Run `spicetify backup apply` again.

### An AUR build fails after a system update

Update the system and the helper's cache together with `yay -Syu`, then try again. If it still fails, open the package's AUR page and read the newest comments, since other users usually report the same error there.

## Credits

- [Arch Linux](https://archlinux.org) and the [Arch Wiki](https://wiki.archlinux.org), where most of the steps here are documented in more detail.
- [Niri](https://github.com/YaLTeR/niri) by YaLTeR.
- [Noctalia Shell](https://github.com/noctalia-dev/noctalia-shell).
- [Vicinae](https://github.com/vicinaehq/vicinae) and [Walker](https://github.com/abenz1267/walker) with Elephant.
- [Papirus Icon Theme](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) and [papirus-folders](https://github.com/PapirusDevelopmentTeam/papirus-folders).
- [Spicetify](https://spicetify.app), [spotify-launcher](https://github.com/kpcyrd/spotify-launcher) and [LazyVim](https://www.lazyvim.org).

Arch Linux, Spotify and the other names here belong to their owners.
