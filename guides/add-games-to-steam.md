# Add games to Steam

Use Steam integration for Gamepad UI, Steam Input, controller layouts, and a
console-style library. The game can still be launched by its existing UMU
wrapper.

## Recommended workflow

1. Create and test the normal launch script first.
2. In Steam, choose **Games → Add a Non-Steam Game**.
3. Browse to the launch script.
4. Rename the shortcut to the proper game name.
5. Leave Steam compatibility-tool forcing disabled because UMU owns Proton.
6. Set launch options only when the wrapper explicitly expects them.

For many games, use a shortcut importer rather than manually rewriting
`shortcuts.vdf`. Steam may overwrite direct metadata edits while running.

## Artwork

Steam uses several separate images:

- Library grid
- Hero
- Logo
- Wide grid
- Icon

SteamGridDB-compatible importers can populate these automatically. Verify that
artwork is associated with the non-Steam shortcut's generated app ID.

## Controller check

Open:

```text
Steam → Settings → Controller → Test Device Inputs
```

If Steam sees the controller but the game does not, enable Steam Input for that
shortcut and ensure Steam launches the wrapper rather than attaching after the
game has started.

