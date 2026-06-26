# steamos-shim

A single multi-call bash script that lets a non-SteamOS Linux distribution launch Steam in "SteamOS mode" via [Gamescope](https://github.com/ValveSoftware/gamescope), selectable from the display manager (SDDM, GDM, …) like any other session.

## What this is

The Steam client, when invoked with `-steamdeck -steamos3`, calls out to several scripts that only ship with SteamOS — `jupiter-biosupdate`, `steamos-update`, `steamos-select-branch`, `steamos-set-timezone`, `steamos-session-select`. On a regular distro, none of these exist and the corresponding features (first-run setup, "Switch to Desktop", update flow) refuse to proceed.

`steamos-shim` is a busybox-style multi-call binary: one real script at `/usr/bin/steamos-shim`, plus symlinks for each name the Steam client expects. The script dispatches on `basename "$0"` and returns whatever the Steam client wants to see (an exit code, a stub message, or — for `gamescope-session` — the actual `gamescope -e -- steam -steamdeck -steamos3` launch).

## Files installed

```
/usr/bin/steamos-shim                                 (real script)
/usr/bin/gamescope-session                            -> steamos-shim
/usr/bin/jupiter-biosupdate                           -> steamos-shim
/usr/bin/steamos-select-branch                        -> steamos-shim
/usr/bin/steamos-session-select                       -> steamos-shim
/usr/bin/steamos-update                               -> steamos-shim
/usr/bin/steamos-polkit-helpers/jupiter-biosupdate    -> ../steamos-shim
/usr/bin/steamos-polkit-helpers/steamos-set-timezone  -> ../steamos-shim
/usr/bin/steamos-polkit-helpers/steamos-update        -> ../steamos-shim
/usr/share/wayland-sessions/steam.desktop             (display-manager entry)
```

## Requirements

- A working Steam client for Linux (install from your distro, run it once so the launcher fetches the bootstrap).
- `gamescope` available.
- AMD GPU recommended; NVIDIA and Intel can work but with caveats — see [Gamescope's notes](https://github.com/ValveSoftware/gamescope#status-of-gamescope-packages).
- `mangohud` is optional but required for the in-game Performance Overlay.
- Display manager running under Wayland. SDDM users on X11 should set `DisplayServer=wayland` in `/etc/sddm.conf.d/`.

## Install

Clone this repository and build the package locally with `makepkg`:

```bash
git clone https://github.com/raspirin/steamos-shim steamos-shim
cd steamos-shim
makepkg -si
```

`makepkg` runs the `PKGBUILD`, packages everything into a `.pkg.tar.zst`, and `pacman` installs it. Uninstall is via `pacman -R steamos-shim`.

After install, log out, pick **Steam (Gamescope)** on the login screen, and proceed through the SteamOS first-run setup.

## Uninstall

```bash
sudo pacman -R steamos-shim
```

pacman removes every file the package owns, including all symlinks.

## Why a shim and not the real scripts?

Three reasons over the original approach (one independent script per command):

1. **One file to read, one file to change.** All the stub behavior — exit codes, stub messages, the actual `gamescope` invocation — lives in one ~30-line `case` statement.
2. **No drift between `/usr/bin/<name>` and `/usr/bin/steamos-polkit-helpers/<name>`.** Both paths are symlinks to the same dispatcher; they cannot disagree.
3. **pacman owns the install.** No `installer.sh`, no manual `chmod`/`cp`, clean uninstall, conflict detection against `gamescope-session-steam` and similar.

## Compatibility note

This package conflicts with `gamescope-session-steam` (ChimeraOS's equivalent). Pick one — both target `/usr/bin/gamescope-session`.

## License

MIT — see [LICENSE](LICENSE).

The shim concept and the per-command behavior follow the original guide at <https://github.com/shahnawazshahin/steam-using-gamescope-guide>; this package is an independent multi-call repackaging, not a fork.
