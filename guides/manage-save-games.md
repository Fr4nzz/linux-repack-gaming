# Manage save games

## Find the prefix save location

Windows paths live below:

```text
/path/to/prefix/drive_c/users/USER/
```

Common locations:

```text
AppData/Local/GAME/Saved/SaveGames
AppData/Roaming/GAME
Documents/My Games/GAME
```

Search recently changed files:

```bash
find "/path/to/prefix/drive_c/users" -type f -printf '%T@ %p\n' \
  | sort -nr | head -50
```

## Back up before changing anything

```bash
scripts/backup-game-save \
  "/path/to/save-directory" \
  "/path/to/save-backups"
```

Stop the game before copying saves.

## Import a downloaded save

1. Extract the archive into a temporary directory.
2. Inspect its directory structure and extensions.
3. Back up the current save directory.
4. Copy only the required save files.
5. Preserve filenames and capitalization.
6. Launch and verify before deleting the backup.

Some games bind saves to an account or require a container/index file to be
updated. Restore the backup if the imported save is not recognized.

