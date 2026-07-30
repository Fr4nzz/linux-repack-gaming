# Create desktop shortcuts

Create and test the shell launcher before adding graphical shortcuts. Use the
same launcher from the desktop, application menu, Steam and Sunshine.

## 1. Install desktop and icon tools

On KDE Plasma under Arch Linux or CachyOS:

```bash
sudo pacman -S --needed \
  desktop-file-utils \
  icoutils \
  imagemagick
```

`wrestool` and `icotool` come from `icoutils`.

## 2. Create the application entry

Create `~/.local/share/applications/my-game.desktop`:

```ini
[Desktop Entry]
Type=Application
Name=My Game
Comment=Launch My Game
Exec=/home/USER/.local/bin/launch-my-game
Path=/path/to/game
Icon=my-game
Terminal=false
Categories=Game;
StartupNotify=true
```

Use absolute paths in `Exec` and `Path`. Do not put a long shell command or
environment-variable expression directly in `Exec`; keep that logic in the
launcher script.

Install and validate:

```bash
chmod 0755 "$HOME/.local/share/applications/my-game.desktop"
desktop-file-validate \
  "$HOME/.local/share/applications/my-game.desktop"
```

## 3. Extract the official executable icon

List icon groups:

```bash
wrestool -l -t 14 "/path/to/Game.exe"
```

Extract one valid group by its numeric or named resource ID:

```bash
wrestool -x -t 14 -n RESOURCE_ID \
  "/path/to/Game.exe" > "/tmp/my-game.ico"

icotool -l "/tmp/my-game.ico"
mkdir -p "/tmp/my-game-frames"
icotool -x -o "/tmp/my-game-frames" "/tmp/my-game.ico"
```

Executables may contain several icon groups. Inspect them separately instead of
combining them into one `.ico`.

Choose the best extracted frame for each installed size. If a high-resolution
frame is available, ImageMagick can create exact PNG dimensions:

```bash
magick "/tmp/my-game-frames/BEST-FRAME.png" \
  -resize 256x256 \
  "/tmp/my-game-256.png"

install -Dm0644 "/tmp/my-game-256.png" \
  "$HOME/.local/share/icons/hicolor/256x256/apps/my-game.png"
```

Repeat for useful sizes such as 16, 32, 48, 64, 128 and 256. Prefer native
embedded frames over enlarged small icons.

## 4. Refresh KDE caches

```bash
update-desktop-database \
  "$HOME/.local/share/applications"

gtk-update-icon-cache -f -t \
  "$HOME/.local/share/icons/hicolor"

kbuildsycoca6 --noincremental
```

Confirm KDE resolves the icon:

```bash
kiconfinder6 my-game
```

If Plasma still shows the old icon, log out and back in. Restarting
`plasma-plasmashell` is a more disruptive fallback.

## 5. Add an optional desktop copy

Discover the configured desktop directory:

```bash
desktop_dir="$(xdg-user-dir DESKTOP)"
```

Then copy:

```bash
cp "$HOME/.local/share/applications/my-game.desktop" \
  "$desktop_dir/my-game.desktop"
chmod 0755 "$desktop_dir/my-game.desktop"
```

Keep `Terminal=false` so the shortcut does not open a console.

## 6. Verify all surfaces

Check:

- Desktop icon.
- Plasma application search/menu.
- Task Manager icon while the game runs.

If only the running-task icon is wrong, inspect the window class:

```bash
kdotool search --name "My Game" getwindowclassname
```

Add `StartupWMClass=` only when the observed stable class requires it. Do not
change the executable merely to influence task grouping.
