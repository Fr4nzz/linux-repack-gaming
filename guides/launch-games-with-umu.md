# Launch games with UMU

UMU runs Proton outside Steam while supplying the runtime and environment Proton
expects. It is useful for independently installed games and works from shell
scripts, desktop entries, non-Steam shortcuts, and Sunshine.

Alternatives include Bottles, Heroic, Lutris, Wine and Steam. This workflow uses
UMU because its command-line configuration is easy to inspect and automate.

## 1. Install UMU and a Proton runner

On Arch Linux or CachyOS:

```bash
sudo pacman -S --needed umu-launcher
```

UMU can obtain its default compatible Proton automatically. For a first test,
use:

```bash
export WINEPREFIX="/path/to/game-prefix"
export GAMEID="umu-default"
export PROTONPATH="GE-Proton"

umu-run "/path/to/game/Game.exe"
```

`PROTONPATH=GE-Proton` asks UMU to obtain and use its supported current
GE-Proton choice. Pin an absolute runner directory after validation so an
automatic update cannot change a working game unexpectedly.

On CachyOS, a packaged runner can instead be installed and selected explicitly:

```bash
sudo pacman -S --needed proton-cachyos-slr

export PROTONPATH="/usr/share/steam/compatibilitytools.d/proton-cachyos-slr"
```

Confirm the selected path:

```bash
test -x "$PROTONPATH/proton"
```

UMU's upstream documentation defines `WINEPREFIX`, `GAMEID`, `STORE`, and
`PROTONPATH`: [umu-launcher](https://github.com/Open-Wine-Components/umu-launcher).

## 2. Create a dedicated runtime prefix

Create a runtime prefix separate from the installer prefix:

```bash
mkdir -p "/path/to/game-prefix"
```

UMU initializes it on first launch. Keep one prefix per game unless its
executables need to share one.

## 3. Create a per-game launcher

From this repository:

```bash
install -Dm0755 scripts/launch-umu-game \
  "$HOME/.local/bin/launch-my-game"
```

Edit these values in the copy:

```bash
GAME_DIR="/path/to/game"
GAME_EXE="Game.exe"
GAME_PREFIX="/path/to/game-prefix"
GAME_PROTON="/path/to/Proton"
GAME_ID="umu-default"
GAME_STORE=""
```

Then run it from a terminal once:

```bash
"$HOME/.local/bin/launch-my-game"
```

The template changes to `GAME_DIR` before executing the game because some
Windows titles resolve DLLs and data relative to their working directory.

## 4. Choose `GAMEID` and `STORE`

`GAMEID` is an UMU database identifier, commonly `umu-STEAM_APPID` when one
exists. It is not an arbitrary save-game ID.

For an unknown title, begin with:

```bash
GAMEID=umu-default
```

If the title has an UMU entry, use its documented ID and storefront:

```bash
GAMEID=umu-123456
STORE=steam
```

Do not copy another game's ID to obtain a Proton fix. Fixes are title-specific.

## 5. Hybrid AMD/NVIDIA laptops

First enumerate Vulkan devices and ICD manifests:

```bash
vulkaninfo --summary
ls -l /usr/share/vulkan/icd.d/
```

For NVIDIA render offload, add these exports to the game launcher:

```bash
export __NV_PRIME_RENDER_OFFLOAD=1
export __VK_LAYER_NV_optimus=NVIDIA_only
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export VK_DRIVER_FILES=/usr/share/vulkan/icd.d/nvidia_icd.json
```

Confirm the actual game appears while running:

```bash
nvidia-smi
```

Do not apply these variables globally when the panel and desktop compositor use
the integrated GPU. In a hybrid Gamescope command, clear them for Gamescope and
reintroduce them only for the game child after `--`.

## 6. First-launch logging

Use UMU logging for launcher/runtime diagnosis:

```bash
UMU_LOG=1 "$HOME/.local/bin/launch-my-game" \
  2>&1 | tee "/tmp/my-game-umu.log"
```

Use Proton logging only for a focused test because it can become very large:

```bash
PROTON_LOG=1 \
PROTON_LOG_DIR="/path/to/log-directory" \
  "$HOME/.local/bin/launch-my-game"
```

Turn it off for normal play.

## 7. Proton version policy

1. Start a new game with UMU's current supported runner or a current GE-Proton.
2. Test the menu, settings, a representative gameplay load, saving, and exit.
3. Pin the exact working runner directory.
4. Change runners only for a reproduced compatibility issue or a controlled
   upgrade test.
5. Keep the previous runner until the new one passes the same save and scene.

A higher version number does not guarantee better behavior for a specific game.
AC Shadows required a debug/breadcrumb build for a reproducible NVIDIA hang.
Replacing it with a newer GE release would remove the tested workaround.

## 8. Check for a clean exit

Quit from the game's own menu and verify its prefix has no remaining processes:

```bash
pgrep -af 'Game.exe|umu-run|wineserver' || true
```

If the wrapper changes the power profile or other system state, do not end the
script with `exec`; retain a shell `trap` that restores state after UMU returns.
If it has no restoration work, `exec umu-run ...` is preferable for Steam and
Sunshine process tracking.
