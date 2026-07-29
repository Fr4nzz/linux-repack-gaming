# Create desktop shortcuts

## 1. Create the desktop entry

Save this as:

```text
~/.local/share/applications/my-game.desktop
```

```ini
[Desktop Entry]
Type=Application
Name=My Game
Comment=Launch My Game
Exec=/home/USER/.local/bin/launch-my-game
Icon=my-game
Terminal=false
Categories=Game;
StartupNotify=true
```

Use absolute paths in `Exec`.

## 2. Install an icon

Prefer an icon embedded in the game's executable:

```bash
wrestool -x -t 14 "/path/to/Game.exe" > game.ico
convert game.ico[0] my-game.png
install -Dm0644 my-game.png \
  "$HOME/.local/share/icons/hicolor/256x256/apps/my-game.png"
```

Then refresh:

```bash
update-desktop-database "$HOME/.local/share/applications"
gtk-update-icon-cache -f -t "$HOME/.local/share/icons/hicolor"
```

Copy the entry to the desktop if wanted:

```bash
cp "$HOME/.local/share/applications/my-game.desktop" "$HOME/Desktop/"
chmod +x "$HOME/Desktop/my-game.desktop"
```

Keep `Terminal=false` so launching the game does not open a terminal window.
