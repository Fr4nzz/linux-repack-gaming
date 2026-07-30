# Add games to Steam

Steam integration provides Gamepad UI, Steam Input, controller layouts, and a
console-style library. Keep UMU responsible for Proton by adding the Linux
launcher—not the Windows executable.

## 1. Prepare the launcher

The target must be executable and work without a terminal:

```bash
test -x "$HOME/.local/bin/launch-my-game"
"$HOME/.local/bin/launch-my-game"
```

It should remain alive until the game exits. If it spawns the game in the
background and returns immediately, Steam cannot track it correctly.

## 2. Reliable graphical method

In Steam:

1. Choose **Games → Add a Non-Steam Game to My Library**.
2. Browse to the absolute Linux launcher.
3. Open the new shortcut's **Properties**.
4. Set its name and **Start In** game directory.
5. Leave **Launch Options** empty initially.
6. Leave **Force the use of a specific Steam Play compatibility tool** off.

The wrapper already owns UMU, Proton, prefix, GPU, and compatibility variables.
Forcing Steam Proton would create a second compatibility layer.

## 3. Command-line bootstrap

Steam supports an `addnonsteamgame` URI. Percent-encode the absolute target:

```bash
target="$HOME/.local/bin/launch-my-game"

encoded="$(
  python -c 'import sys, urllib.parse; print(urllib.parse.quote(sys.argv[1], safe=""))' \
    "$target"
)"

steam "steam://addnonsteamgame/$encoded"
```

Add one shortcut at a time. This creates a structurally valid entry but normally
uses a filename-derived name; curate it afterward in Steam.

## 4. Why direct `shortcuts.vdf` editing is fragile

Steam stores non-Steam metadata in a binary `shortcuts.vdf` below the active
user's `userdata/STEAMID/config/` directory. Steam can overwrite it while
running.

For one game, prefer Steam's UI or URI. For a managed catalog, use a maintained
shortcut manager such as Steam ROM Manager or another tool that understands the
binary format and generated non-Steam AppIDs.

Before any external manager changes metadata:

```bash
steam -shutdown

while pgrep -x steam >/dev/null; do
  sleep 1
done
```

Back up the exact active user's file:

```bash
cp -a \
  "$HOME/.local/share/Steam/userdata/STEAMID/config/shortcuts.vdf" \
  "$HOME/.local/share/Steam/userdata/STEAMID/config/shortcuts.vdf.backup"
```

Do not assume the first numeric `userdata` directory is the active account.

## 5. Artwork

A non-Steam shortcut has a generated AppID distinct from the official game's
Steam AppID. Artwork tools such as SteamGridDB/SGDBoop or Steam ROM Manager can
associate grid, hero, logo, and icon assets with that generated ID.

When installing manually, the active account's `config/grid/` directory uses
the generated non-Steam ID in filenames commonly shaped as:

```text
NONSTEAMID.jpg
NONSTEAMIDp.jpg
NONSTEAMID_hero.jpg
NONSTEAMID_logo.png
```

Do not guess the ID. Read it from a shortcut-aware manager or the parsed VDF.

## 6. Controller check

Open:

```text
Steam → Settings → Controller → Test Device Inputs
```

Then open the shortcut's controller settings. If Steam sees the controller but
the game does not:

1. Enable Steam Input for that shortcut.
2. Check that Steam launched the wrapper instead of attaching to an existing game.
3. Restart the game after changing controller enumeration.
4. Avoid stacking Steam Input with a second virtual-controller mapper unless
   the title requires it.

## 7. Verification

Confirm in both desktop Steam and Gamepad UI:

- Correct name and artwork.
- Correct Linux target and working directory.
- Compatibility-tool forcing disabled.
- Controller works.
- Steam shows the game running until its final process exits.
- Exiting leaves no hidden audio or Wine process.
