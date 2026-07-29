# Configure Gamescope

Gamescope is a small compositor that can provide a controlled virtual display,
scaling, fullscreen isolation, capture-friendly output, and reliable resolution
choices.

Use it when those features help. A direct launch remains simpler for games that
already present correctly.

## Basic wrapper

Render at 1920×1200 and output at 2560×1600:

```bash
gamescope \
  -w 1920 -h 1200 \
  -W 2560 -H 1600 \
  -r 60 \
  -S fit \
  -f --force-windows-fullscreen \
  -- /path/to/game-launcher
```

Common scaling modes:

- `-S fit`: preserve the complete image and add bars when needed.
- `-S stretch`: fill the output but distort mismatched aspect ratios.
- `-S fill`: preserve aspect ratio while cropping excess edges.
- `-F fsr`: apply Gamescope's spatial FSR scaling filter.

Prefer the game's own modern temporal upscaler, such as DLSS or FSR 2/3, when it
works reliably. Avoid stacking unnecessary upscalers.

## Hybrid GPU selection

To run Gamescope on an AMD display GPU:

```bash
gamescope --prefer-vk-device 1002:DEVICE ...
```

Apply NVIDIA PRIME variables only after `--` so the game, rather than the outer
compositor, uses NVIDIA.

## Avoid nested Gamescope

Inside a dedicated Gamescope Gaming Mode, launch the game directly unless a
title demonstrably requires a second virtual display. Two Gamescope layers add
another scaling and synchronization boundary.

## Troubleshooting

If Gamescope causes black output, flicker, frozen frames with continuing audio,
or Alt+Tab problems:

1. Revert to the last working backend and GPU assignment.
2. Test SDL and Wayland backends separately.
3. Change Gamescope WSI only as a separate variable.
4. Test direct launch to determine whether Gamescope is involved.
