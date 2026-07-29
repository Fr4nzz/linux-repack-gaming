# Damaged-display streaming

This optional guide confines a streamed game to the usable portion of a display
with a damaged edge.

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

Run the game windowed or borderless, discover its actual window class, and add a
dedicated KWin rule that forces:

```text
position: 320,0
size: 960,720
```

Do not match all Wine or Gamescope windows, and do not overwrite unrelated KWin
rules.

