# Extract one repack component

Optional languages, bonus content, and selective downloads are often stored in
separate archives. You may be able to extract one of these without reinstalling
the complete game.

This workflow was tested with a DODI `Spanish-Spain.doi` FreeArc archive. File
names and compression methods vary between repacks.

## 1. Find the optional archive

List likely language or audio archives:

```bash
find "/path/to/repack" -maxdepth 2 -type f \
  \( -iname '*spanish*' -o -iname '*audio*' -o -iname '*language*' \) \
  -printf '%f\n'
```

Do not assume the file extension identifies the archive format:

```bash
file "/path/to/repack/Spanish-Spain.doi"
```

In the tested repack, `file` identified the `.doi` file as a FreeArc archive.

## 2. Inspect it before extracting

Install a compatible FreeArc implementation that supplies `unarc`. On
Arch-based distributions this may come from the AUR, so inspect the current
package and its source before installing it. Then confirm the command exists:

```bash
command -v unarc
```

List the archive:

```bash
unarc l "/path/to/repack/Spanish-Spain.doi"
```

The listing shows exactly what the optional component would add. A voice pack
will commonly contain files resembling:

```text
DataPC_boot_sound_spa.forge
DataPC_boot_sound_spa_patch_01.forge
DataPC_Japan_sound_spa.forge
```

Record the names and uncompressed sizes. This distinguishes full dialogue
archives from small shared-language stubs.

## 3. Try native extraction

Always extract into an empty staging directory—not directly into the game:

```bash
mkdir -p "/path/to/staging"
unarc x -o+ -dp"/path/to/staging" \
  "/path/to/repack/Spanish-Spain.doi"
```

Test the result:

```bash
find "/path/to/staging" -type f -printf '%12s  %P\n' | sort
```

Some repacks use external LOLZ, SREP, or Oodle codecs. Native `unarc` may list
such an archive but stall or fail while extracting it. That does not
automatically mean the language archive is corrupt.

## 4. Use the installer's own codec chain when required

Run the installer in an isolated Wine prefix and allow it to reach its
installation screen. Inno-based installers normally unpack their matching
helpers under:

```text
$WINEPREFIX/drive_c/users/<Wine-user>/AppData/Local/Temp/is-*.tmp/
```

Discover the Wine username rather than assuming it matches the Linux username:

```bash
find "$WINEPREFIX/drive_c/users" -path '*/AppData/Local/Temp' -type d
```

Look for:

```text
unarc.dll
arc.ini
cls.ini
cls-lolz*.dll
cls-srep*.dll
oo2core_*.dll
oo2rec*.exe
```

Copy that complete helper set into a temporary working directory while the
installer is open. These files are version-matched to the repack; do not fetch
random replacements from DLL-download sites.

The advanced fallback is a small 32-bit Windows wrapper that calls the
installer's DLL directly. This repository intentionally does not automate it
because the `FreeArcExtract` arguments and companion codecs can vary between
repacks. Such a wrapper must:

1. Load the extracted `unarc.dll`.
2. Resolve its `FreeArcExtract` export.
3. Call it with `x`, `-o+`, a staging destination, a temporary directory, and
   the optional archive path.
4. Run under the same Wine prefix beside the copied codec helpers.

Use Windows-style paths visible inside that prefix, for example:

```text
C:\Repack\Spanish-Spain.doi
C:\LanguageStage
C:\LanguageTemp
```

Use this only after verifying the DLL export and argument layout for that
specific installer. An incompatible call can simply crash. For most users,
rerunning the installer with only the optional component selected is safer,
even if it is slower.

## 5. Compare with the installed game

Find which staged files are actually missing:

```bash
stage="/path/to/staging"
game="/path/to/installed/game"

while IFS= read -r relative; do
  if [[ -e "$game/$relative" ]]; then
    printf 'present  %s\n' "$relative"
  else
    printf 'missing  %s\n' "$relative"
  fi
done < <(cd "$stage" && find . -type f -printf '%P\n' | sort)
```

For files already present, compare hashes:

```bash
sha256sum \
  "$stage/DataPC_shared_00_sound_spa.forge" \
  "$game/DataPC_shared_00_sound_spa.forge"
```

Matching hashes mean there is no reason to overwrite the installed copy.
Different hashes may indicate a different game or patch version; do not mix
them blindly.

## 6. Install only the missing files

First back up the game's language configuration:

```bash
backup="/path/to/backups/before-language-pack-$(date +%F-%H%M%S)"
mkdir -p "$backup"
cp -a "/path/to/game-language-config" "$backup/"
```

Copy only verified missing files while the game is closed:

```bash
install -m 0644 \
  "$stage/DataPC_boot_sound_spa.forge" \
  "$game/DataPC_boot_sound_spa.forge"
```

Verify the source and installed copies:

```bash
sha256sum \
  "$stage/DataPC_boot_sound_spa.forge" \
  "$game/DataPC_boot_sound_spa.forge"
```

Repeat for each required missing archive. Preserve any relative subdirectories
such as `dlc_20/`.

## 7. Test before cleaning up

Start the game, select the newly installed voice language, and test both ordinary
dialogue and DLC content. Keep the staging directory until this succeeds.

After verification, remove the staging directory and disposable Wine prefix if
they are no longer needed. Keep the small configuration backup.

## Important limitations

- This extracts content already present in your local repack; it does not add a
  language the repack never included.
- Optional archives must match the installed game version.
- A late failure elsewhere in a full installation does not necessarily damage
  an independent language archive.
- Do not copy cracks, executables, or patch data from a partial installation
  into a working installation unless their versions have also been verified.
