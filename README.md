# steamos-shim

A single multi-call bash script that lets a non-SteamOS Linux distribution launch Steam in "SteamOS mode" via [Gamescope](https://github.com/ValveSoftware/gamescope), selectable from the display manager (SDDM, GDM, …) like any other session.

## What this is

The Steam client, when invoked with `-steamdeck -steamos3`, calls out to several scripts that only ship with SteamOS — `jupiter-biosupdate`, `steamos-update`, `steamos-select-branch`, `steamos-set-timezone`, `steamos-session-select`. On a regular distro, none of these exist and the corresponding features (first-run setup, "Switch to Desktop", update flow) refuse to proceed.

`steamos-shim` is a busybox-style multi-call binary: one real script at `/usr/bin/steamos-shim`, plus symlinks for each name the Steam client expects. The script dispatches on `basename "$0"` and returns whatever the Steam client wants to see (an exit code, a stub message, or — for `gamescope-session` — the actual `gamescope -e -- steam -steamdeck -steamos3` launch).

## Switching between gamescope and Plasma

`steamos-session-select` (the command Steam invokes for "Switch to Desktop") and a "Return to Gamemode" application entry let the user move between gamescope and a Plasma desktop without ever seeing the display manager.

**Precondition: autologin must be configured on your DM.** This package is DM-agnostic — pick SDDM, GDM, plasma-greeter, etc., and configure it to auto-log into your user account. Without autologin, every switch will land the user at the DM login prompt.

**How a switch works.** Both directions go through the same mechanism: a state file at `~/.config/steamos-shim/next-session` records "go to gamescope" or "go to plasma", then `loginctl terminate-session` ends the current session. The DM's autologin re-enters "Steam (Gamescope)", and the dispatcher reads the state file to decide whether to start gamescope or the Plasma session listed at `/usr/share/wayland-sessions/plasma.desktop`. The state file is reset to `gamescope` on every dispatch, so a power cycle from Plasma always returns to game mode.

**Steam in Plasma.** This package does not auto-start Steam inside Plasma. If you want SteamDeck-like behaviour where Steam appears on the desktop, add it via Plasma's own autostart configuration:

> System Settings → Autostart → Add Application → `steam` (optionally `steam -silent`)

**Desktop shortcut for "Return to Gamemode".** The package installs `/usr/share/applications/steamos-return-to-gamescope.desktop`, which appears in the application menu and KRunner. To put it on the Plasma desktop:

```bash
cp /usr/share/applications/steamos-return-to-gamescope.desktop ~/Desktop/
chmod +x ~/Desktop/steamos-return-to-gamescope.desktop
```

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
/usr/share/applications/steamos-return-to-gamescope.desktop  (menu/desktop entry: switch back to gamescope)
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

The package leaves two user-owned items in place by design:

- `~/.config/steamos-shim/` — the state file recording the next session to dispatch into. Remove manually if you want a fully clean slate.
- `~/Desktop/steamos-return-to-gamescope.desktop` — only present if you copied it there yourself.

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
