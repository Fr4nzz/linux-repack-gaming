# Configure Gamescope

Gamescope is a compositor that can provide a controlled virtual display,
scaling, fullscreen isolation, capture-friendly output, and stable resolution
choices.

Use it when those features help. Direct presentation has fewer moving parts for
games that already work correctly.

## 1. Install and inspect

On Arch Linux or CachyOS:

```bash
sudo pacman -S --needed gamescope vulkan-tools
```

Record the version and available options:

```bash
gamescope --version
gamescope --help
vulkaninfo --summary
```

Do not copy a GPU PCI ID from another computer. Find the AMD/NVIDIA IDs in the
Vulkan summary or with:

```bash
lspci -nnk | grep -A3 -E 'VGA|3D|Display'
```

## 2. Basic nested wrapper

Render at 1920×1200 and output at 2560×1600:

```bash
gamescope \
  -w 1920 -h 1200 \
  -W 2560 -H 1600 \
  -r 60 \
  -S fit \
  -f --force-windows-fullscreen \
  -- /absolute/path/to/game-launcher
```

Meaning:

| Option | Purpose |
|---|---|
| `-w/-h` | Resolution exposed to the game |
| `-W/-H` | Size of the nested Gamescope output |
| `-r` | Virtual/nested refresh |
| `-S` | Scaling geometry |
| `-F fsr` | Optional Gamescope FSR1 spatial filter |
| `-f` | Fullscreen Gamescope's outer window |
| `--force-windows-fullscreen` | Size client windows to the nested display |

Scaling modes:

- `fit`: preserve the complete image and add bars when necessary.
- `fill`: preserve geometry while cropping excess edges.
- `stretch`: fill the output and distort when aspect ratios differ.

Prefer a game's modern temporal upscaler, such as DLSS or FSR 2/3, when it works
reliably. Avoid stacking it with Gamescope FSR1 without a measured reason.

## 3. Hybrid AMD-compositor/NVIDIA-renderer command

On a muxless laptop whose panel is connected to AMD, keep NVIDIA variables away
from Gamescope and apply them only to the game:

```bash
env \
  -u VK_DRIVER_FILES \
  -u __NV_PRIME_RENDER_OFFLOAD \
  -u __VK_LAYER_NV_optimus \
  -u __GLX_VENDOR_LIBRARY_NAME \
  gamescope \
    --prefer-vk-device 1002:DEVICE \
    --backend wayland \
    -w 1920 -h 1200 \
    -W 2560 -H 1600 \
    -r 60 \
    -S fit \
    -f --force-windows-fullscreen \
    -- env \
      __NV_PRIME_RENDER_OFFLOAD=1 \
      __VK_LAYER_NV_optimus=NVIDIA_only \
      __GLX_VENDOR_LIBRARY_NAME=nvidia \
      VK_DRIVER_FILES=/usr/share/vulkan/icd.d/nvidia_icd.json \
      /absolute/path/to/game-launcher
```

Replace `1002:DEVICE` with the actual AMD device. `--prefer-vk-device` chooses
Gamescope's compositor GPU; it does not choose the game's renderer.

If the launcher already exports NVIDIA variables internally, the child `env`
block can be omitted. The important boundary is that the outer Gamescope starts
without them.

## 4. Backend and WSI policy

Test one change at a time:

```text
--backend wayland
--backend sdl
```

Do not enable `ENABLE_GAMESCOPE_WSI=1` globally. It can fix presentation for one
title and break another. Scope it to the game child only when a controlled test
demonstrates a benefit:

```bash
gamescope ... -- env ENABLE_GAMESCOPE_WSI=1 /path/to/launcher
```

RDR2 required a particular AMD Gamescope/NVIDIA renderer/WSI arrangement on the
tested laptop. AC Shadows required its own separate boundary. Those results are
evidence for per-title configuration—not universal WSI defaults.

## 5. Frame limiting

`-r 60` exposes a 60 Hz virtual display. It is not guaranteed to be the best
render-work limiter for every presentation mode.

Prefer one deliberate authority:

```text
DX12/vkd3d: VKD3D_FRAME_RATE=60
DXVK:       DXVK_CONFIG="dxvk.maxFrameRate = 60"
or a proven in-game limiter
```

Do not stack in-game, VKD3D/DXVK, MangoHud, Steam, and Gamescope caps without a
specific experiment.

## 6. Dedicated Gaming Mode

Inside an embedded Gamescope session, launch the game directly unless it
demonstrably needs another virtual display. A second Gamescope adds another
swapchain, scaling stage, and synchronization boundary.

## 7. Regression test

For every Gamescope change:

1. Use the same save, scene, resolution, settings, and power profile.
2. Test launcher-to-game handoff.
3. Alt+Tab repeatedly and leave the game unfocused briefly.
4. Check fit/fill geometry and mouse/controller capture.
5. Quit normally and confirm no audio or process remains.

Reject the change for black output, screenshot-invisible flicker, frozen frames
with continuing audio, lost controller input, or failed fullscreen restoration.

Upstream option reference:
[ValveSoftware/gamescope](https://github.com/ValveSoftware/gamescope).
