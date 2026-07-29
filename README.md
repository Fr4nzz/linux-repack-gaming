# Linux Repack Gaming

Practical DODI, FitGirl, KaOs, ElAmigos, and similar repack installation
workflows for Wine—plus UMU, Proton, Gamescope, Steam, and Sunshine integration.

Task-focused guides for installing and launching Windows games on Linux, adding
desktop and Steam integration, and optionally using Gamescope, a dedicated
Gaming Mode, or Sunshine/Moonlight.

This repository is intentionally automation- and AI-agent-friendly. The tested
workflow uses inspectable command-line tools, per-game configuration files, and
small shell scripts. UMU was chosen because it provides a practical CLI for
running Proton outside Steam.

This is not the only good approach. Bottles, Heroic, Lutris, or Steam itself may
be more convenient for other users. These guides document the workflow that was
actually tested instead of attempting to cover every launcher.

## Start with the simplest path

Do not begin with every workaround enabled:

1. Verify the repack's files or checksums.
2. Try the installer with an ordinary current Wine/Lutris configuration.
3. Use the patched-Wine workflow only when extraction reproduces an
   ISDone/Unarc, WoW64, progress-rendering, or related compatibility failure.
4. After installation, use a fresh runtime prefix and test the game directly.
5. Add Gamescope only when it supplies a needed resolution, scaling, capture,
   fullscreen, or presentation fix.

The specialized fixes remain valuable because they are isolated and
reproducible; they are not intended as mandatory layers for every game.

## Choose what you need

| Goal | Guide |
|---|---|
| Install a compressed Windows game installer | [Install repacks with Wine](guides/install-repacks-with-wine.md) |
| Extract only an optional language or audio pack | [Extract one repack component](guides/extract-one-repack-component.md) |
| Launch an installed game with Proton | [Launch games with UMU](guides/launch-games-with-umu.md) |
| Control resolution, scaling, or fullscreen behavior | [Configure Gamescope](guides/configure-gamescope.md) |
| Add a proper desktop/menu shortcut and icon | [Create desktop shortcuts](guides/create-desktop-shortcuts.md) |
| Add a non-Steam game to Steam | [Add games to Steam](guides/add-games-to-steam.md) |
| Create a separate Steam-style CachyOS session | [Set up CachyOS Gaming Mode](guides/setup-cachyos-gaming-mode.md) |
| Launch games remotely with Moonlight | [Add games to Sunshine](guides/add-games-to-sunshine.md) |
| Back up, restore, or import saves | [Manage save games](guides/manage-save-games.md) |
| Fit games into an unusual usable screen region | [Damaged-display streaming](guides/damaged-display-streaming.md) |

## Tested design choices

| Choice | Reason |
|---|---|
| UMU | CLI-friendly Proton launching outside Steam |
| Isolated prefixes | Changes made for one game do not affect the others |
| Configurable Proton build | Start current; pin a different build only after a demonstrated compatibility difference |
| Shell launchers | Inspectable, reproducible, and easy for humans or agents to edit |
| Gamescope when useful | Standard wrapper for scaling, capture, fullscreen isolation, or known fixes; direct launch remains available |
| Optional Steam integration | Adds Gamepad UI and Steam Input without making Steam responsible for the installation |

## Scope

The repository contains scripts and documentation only. It does not include
games, cracks, repack archives, proprietary decompression helpers, Proton
builds, or other copyrighted binaries. Use it only with software you are
authorized to install.

## Status

The workflows are based on CachyOS/Arch Linux with KDE Wayland and a hybrid
AMD/NVIDIA laptop. Commands are written to be adaptable; paths and GPU IDs must
be changed for the target machine.

## Community references

This repository complements existing community workflows instead of replacing
them:

- [Lutris repack installation guide](https://www.reddit.com/r/LinuxCrackSupport/comments/yqfirv/)
- [Lutris installation and troubleshooting guide](https://www.reddit.com/r/LinuxCrackSupport/comments/1rlpyg4/)
- [fitgirl-wine development and testing thread](https://www.reddit.com/r/LinuxCrackSupport/comments/1v6so9f/)

Those guides are especially useful for people who prefer a graphical launcher.
This repository focuses on the command-line UMU workflow and on installer
failures the general Lutris guides explicitly do not solve.
