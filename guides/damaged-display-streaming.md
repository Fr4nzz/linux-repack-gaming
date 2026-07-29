# Damaged-display streaming

This optional guide confines a streamed game to the usable portion of a display
with a damaged edge.

First make ordinary streaming work by following
[Add games to Sunshine](add-games-to-sunshine.md).

## Coordinate conversion

KDE Wayland rules use logical coordinates:

```text
logical size = physical size / display scale
```

For a 1920×1080 stream at 150% scale:

```text
logical desktop: 1280×720
physical damaged left 480 px → logical damaged left 320 px
usable logical rectangle: x=320, y=0, width=960, height=720
```

## Presentation choices

- Native 4:3: best when the game supports 960×720 or another 4:3 mode.
- Fit: preserves all 16:9 content and adds bars.
- Fill: preserves shape but crops the left and right edges.
- Stretch: fills the region but distorts the image; generally avoid it.

Gamescope can expose a controlled output rectangle and switch between fit and
fill. Keep a fit mode available when cropped HUD elements must be read.

## KWin confinement

Run the game windowed or borderless. On Plasma Wayland, discover its real
window class using KWin's rule picker:

1. Open **System Settings → Window Management → Window Rules**.
2. Choose **Add New**, then **Detect Window Properties**.
3. Click the game or its Gamescope window.
4. Match the narrowest stable class or title.

Add a dedicated rule that forces:

```text
position: 320,0
size: 960,720
```

Do not match all Wine or Gamescope windows, and do not overwrite unrelated KWin
rules.

## Gamescope rectangle

For a nested Gamescope window, make its physical output match the usable
region:

```bash
gamescope \
  -W 1440 -H 1080 \
  -w 1440 -h 1080 \
  -r 60 \
  -S fit \
  --backend wayland \
  -- your-game-command
```

The KWin rule then sizes that Gamescope window to the corresponding logical
`960×720` rectangle. Start with `fit`; use `fill` only when accepting cropped
side content. Avoid `stretch` unless distortion is intentional.

## Validate

- The entire launcher remains reachable before starting the game.
- Fit mode shows every HUD edge.
- Fill mode crops symmetrically rather than shifting the image.
- Controller input reaches the game through Moonlight.
- Closing the window also terminates the underlying Wine/game process.
