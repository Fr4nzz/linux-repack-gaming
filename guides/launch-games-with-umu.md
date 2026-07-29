# Launch games with UMU

UMU runs Proton outside Steam while supplying the runtime and environment Proton
expects. It is useful for independently installed games and works well from
scripts, desktop entries, Steam shortcuts, and Sunshine.

Alternatives include Bottles, Heroic, Lutris, plain Wine, or adding the
executable directly to Steam. UMU is used here because it is CLI-friendly and
the configuration remains easy to inspect and automate.

## 1. Install UMU

On CachyOS/Arch:

```bash
sudo pacman -S --needed umu-launcher
```

Obtain a Proton build separately and note its directory.

## 2. Create a per-game launcher

Copy the template:

```bash
install -Dm0755 scripts/launch-umu-game \
  "$HOME/.local/bin/launch-my-game"
```

Edit these values:

```bash
GAME_DIR="/path/to/game"
GAME_EXE="Game.exe"
WINEPREFIX="/path/to/prefix"
PROTONPATH="/path/to/Proton"
GAMEID="umu-default"
```

Then run:

```bash
"$HOME/.local/bin/launch-my-game"
```

## Proton version policy

1. Start a new game with a current stable Proton or GE-Proton build.
2. Keep it if the game works.
3. Test another build only for a known or reproduced compatibility issue.
4. Pin the verified build so an update cannot silently change behavior.
5. Re-test before upgrading an existing working launcher.

## Hybrid AMD/NVIDIA laptops

For NVIDIA render offload, add to the launcher's environment:

```bash
__NV_PRIME_RENDER_OFFLOAD=1
__VK_LAYER_NV_optimus=NVIDIA_only
__GLX_VENDOR_LIBRARY_NAME=nvidia
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/nvidia_icd.json
```

Confirm the game appears in:

```bash
nvidia-smi
```

Do not apply NVIDIA variables globally when the desktop and panel are driven by
the integrated AMD GPU.
