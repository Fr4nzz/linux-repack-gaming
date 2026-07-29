# Manage save games

## Understand which component chooses the path

The repacker and the save emulator are separate:

- DODI, FitGirl, KaOs, and ElAmigos package the game.
- Goldberg, RUNE, CODEX, Razor1911, and similar components may emulate a store
  or account service.
- Many games ignore the emulator for progress and use their normal Windows save
  directory.

Therefore, a “DODI save location” does not exist universally. Inspect the game
files and the prefix instead of choosing a path from the repacker's name.

## Translate Windows paths into a Wine or Proton prefix

For a standalone UMU/Wine prefix:

```text
/path/to/prefix/drive_c/users/steamuser/
```

For a game managed by Steam:

```text
~/.local/share/Steam/steamapps/compatdata/APPID/pfx/drive_c/users/steamuser/
```

Replace `steamuser` if the prefix uses another Windows username. Windows
environment variables translate as follows:

| Windows variable | Location below `drive_c/users/USER` |
|---|---|
| `%USERPROFILE%` | `.` |
| `%APPDATA%` | `AppData/Roaming` |
| `%LOCALAPPDATA%` | `AppData/Local` |
| `%PUBLIC%` | usually `../Public` |
| `%USERPROFILE%/Documents` | `Documents` |

## Common game-controlled locations

These are worth checking before emulator-specific paths:

```text
AppData/Local/GAME/Saved/SaveGames
AppData/LocalLow/PUBLISHER/GAME
AppData/Roaming/PUBLISHER/GAME
Documents/GAME
Documents/My Games/GAME
Saved Games/GAME
```

Unreal Engine games frequently use:

```text
AppData/Local/GAME/Saved/SaveGames
```

Some games add a numeric Steam ID beneath the final directory:

```text
.../SaveGames/7656119XXXXXXXXXX/
```

That ID can be part of the save format or encryption context. Preserve it when
backing up, and do not assume a downloaded save belongs under the same ID.

## Common emulator-controlled locations

These are common patterns, not guarantees. Emulator builds and per-game
configuration can override them.

| Emulator or release | Common Windows path |
|---|---|
| Goldberg Steam Emulator | `%APPDATA%/Goldberg SteamEmu Saves/APPID/` |
| Goldberg Uplay emulator | `%APPDATA%/Goldberg UplayEmu Saves/GAMEID/` |
| RUNE Steam emulator | `%PUBLIC%/Documents/Steam/RUNE/APPID/remote/` |
| CODEX Steam emulator | `%PUBLIC%/Documents/Steam/CODEX/APPID/remote/` |
| Older CODEX layouts | `%APPDATA%/Steam/CODEX/APPID/remote/` |
| Razor1911 Social Club emulator | `%APPDATA%/.1911/GAME/profile/` |
| Steam itself | `Steam/userdata/STEAMID/APPID/remote/` |

Goldberg documents its default as
`%APPDATA%/Goldberg SteamEmu Saves/`. A `local_save.txt` beside its emulator
library can redirect storage into a directory beside the game instead. See the
[Goldberg emulator README](https://gitlab.com/Mr_Goldberg/goldberg_emulator/-/blob/master/Readme_release.txt).

RUNE, CODEX, and other release emulators may customize their layouts. Some only
store Steam Remote Storage data centrally while the game keeps its actual
progress in `Documents` or `AppData`.

## Verified examples from the tested setup

All paths below are relative to each game's active prefix unless an absolute
prefix is shown.

### Red Dead Redemption 2 — Razor1911

Prefix:

```text
/mnt/Bolt/Games/RDR2/.prefix
```

Save directory:

```text
drive_c/users/steamuser/AppData/Roaming/.1911/
Red Dead Redemption 2/profile/
```

Progress files include:

```text
SRDR30000
SRDR30015
SRDR30000.bak
SRDR30015.bak
cfg.dat
```

Back up the complete `profile` directory, not only the newest `SRDR` file.

### GTA V Enhanced — RUNE/Social Club layout

Prefix:

```text
/mnt/Bolt/Games/GTAV/.prefix
```

Save directory:

```text
drive_c/users/Public/Documents/Socialclub/RUNE/
GTAV Enhanced/101001101010/
```

Progress files include:

```text
SGTA50000
SGTA50001
SGTA50015
SGTA50000.bak
cfg.dat
```

This is not RUNE's usual `Steam/RUNE/APPID/remote` pattern; the included Social
Club emulator selected a title- and account-specific directory.

### Marvel's Spider-Man 2 — native Documents path

Prefix:

```text
/mnt/Bolt/Games/SpiderMan2/.prefix
```

Save directory:

```text
drive_c/users/steamuser/Documents/
Marvel's Spider-Man 2/76561197960271872/
```

Important files include:

```text
slot1-autosave.save
slot1-manual-0.save
prefs-autosave.save
```

When importing progress, preserve `prefs-autosave.save` unless the downloaded
save explicitly requires replacing preferences too.

### Assassin's Creed Shadows — Goldberg Uplay emulator

Active tested prefix:

```text
/mnt/Bolt/Games/ACShadows/.prefix-breadcrumb-test
```

Save directory:

```text
drive_c/users/steamuser/AppData/Roaming/
Goldberg UplayEmu Saves/8006/
```

Files include:

```text
ACShadows[AutoSave01].save
ACShadows[AutoSave02].save
ACShadows[Options].save
```

The options file is separate from gameplay progress. Back it up, but do not
delete all autosaves merely to reset a broken graphics or language setting.

### Clair Obscur: Expedition 33 — native Unreal path

Prefix:

```text
/mnt/Bolt/Games/ClairObscurExpedition33/.prefix
```

Save directory:

```text
drive_c/users/steamuser/AppData/Local/Sandfall/Saved/
SaveGames/76561197960271872/
```

Important files include:

```text
EXPEDITION_0.sav
SavesContainer.sav
PlatformSaveData.sav
SharedGameUserSettings.sav
EnhancedInputUserSettings.sav
```

`EXPEDITION_0.sav` contains a slot, while `SavesContainer.sav` can be needed for
the game to enumerate it. Back up the entire numeric directory before importing
a slot.

### Escape From Ever After — native Unreal path

Prefix:

```text
/mnt/Bolt/Games/EscapeFromEverAfter/.prefix
```

Save directory:

```text
drive_c/users/steamuser/AppData/Local/efea/Saved/SaveGames/
```

The tested progress file is:

```text
1124073472/file1.sav
```

Preserve the numeric parent directory.

## Find an unknown save reliably

Start by searching likely names and extensions:

```bash
prefix="/path/to/prefix"

find "$prefix/drive_c/users" -type f \
  \( -iname '*.sav' -o -iname '*.save' -o -iname 'SGTA*' \
     -o -iname 'SRDR*' -o -iname 'slot*' \) \
  -printf '%T@ %s %p\n' | sort -nr | head -100
```

If that is inconclusive:

1. Close the game.
2. Record recent files.
3. Launch the game, create a new manual save, then exit normally.
4. Search by modification time again:

```bash
find "$prefix/drive_c/users" -type f -mmin -10 \
  -printf '%TY-%Tm-%Td %TH:%TM:%TS %10s %p\n' | sort
```

Also inspect emulator configuration beside the game:

```bash
find "/path/to/game" -maxdepth 4 -type f \
  \( -iname 'steam_emu.ini' -o -iname 'steam_api.ini' \
     -o -iname 'local_save.txt' -o -iname 'account_name.txt' \
     -o -iname 'user_steam_id.txt' \) -print
```

Do not confuse shader caches, crash reports, settings, or launcher metadata
with progress. Confirm by creating a known save and observing which files
change.

## Back up before changing anything

Stop the game and its launcher, then back up the whole logical save directory:

```bash
scripts/backup-game-save \
  "/path/to/save-directory" \
  "/path/to/save-backups"
```

Keep account-ID directories, container files, configuration files, and the
emulator's own `.bak` files. A complete directory backup is cheap and avoids
guessing which files are linked.

## Import a downloaded save

1. Extract the archive into a temporary directory.
2. Inspect its directory structure, filenames, and Steam/account ID.
3. Back up the current save directory.
4. Let the game create a save slot first when possible.
5. Replace only the corresponding progress file initially.
6. Preserve capitalization and numeric parent directories.
7. Launch and verify before deleting the backup.

Some games bind saves to an account ID, encrypt them, or maintain a
container/index file. If the imported save is not recognized, compare the
source and active account directories and restore the backup before trying a
conversion.

## Migrating between cracks or to an official copy

Changing the emulator can change both the storage path and reported Steam ID.
Copying files to the new directory may be sufficient, but it is not universally
safe.

Before migration, record:

- Old and new emulator names.
- Old and new app IDs.
- Old and new reported Steam/account IDs.
- Every file in the original save directory.
- Whether the game creates a new empty save in the destination.

Prefer this order:

1. Launch the destination build once and create a new save.
2. Exit fully.
3. Back up both source and destination directories.
4. Replace only the equivalent progress files.
5. Preserve destination-side settings, indexes, and containers initially.
6. Restore the destination backup if the save is rejected.
