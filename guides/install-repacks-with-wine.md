# Install repacks with Wine

This guide covers Inno Setup installers that use FreeArc, ISDone, Unarc, LOLZ,
SREP, or Oodle decompression.

## Scope and when to use this

Use an existing Wine or graphical-launcher setup if it works. Use this guide
for:

- FitGirl installers hanging near 0% under WoW64 Wine.
- ISDone/Unarc failures that persist with verified source archives.
- KaOs installers with missing progress rendering.
- DODI installers whose Oodle helper cannot be found by Wine.

The patched Wine runs the installer. Start the installed game with the latest
stable Proton supported by UMU, then pin the version that passes testing.

The complete example uses the same two fixes that allowed the tested Assassin's
Creed Shadows DODI installer to extract approximately 151 GB under Linux:

1. The `fitgirl-wine` 11.13 patched Wine build.
2. This repository's DODI Oodle helper-path wrapper.

The installer later failed inside a DLC archive with Unarc `-14`. The same
archive failed under native Windows, so that late failure was in the repack
rather than Wine. The Linux workflow had already extracted the base game and
proved that its earlier Wine-specific `-11` failure was solved.

## Wine versus Proton

These are related but not interchangeable roles in this workflow:

| Tool | Use here |
|---|---|
| `fitgirl-wine` 11.13 | Run the compressed repack installer |
| GE-Proton, Proton-CachyOS, or another Proton build | Launch the installed game through UMU or Steam |
| Gamescope | Optional presentation/scaling wrapper around the launched game |

`GE-Proton11-3` was useful for some installed games, but it is not the patched
installer runtime documented here.

## The exact tested Wine build

The tested build is:

```text
Project: BrandowLucas/fitgirl-wine
Release: 11.13
Asset:   wine-11.13-patched-fitgirl-amd64.tar.xz
Runtime: wine-11.13-42-gd30dcd75bc9
SHA256:  709c1a4f027507aeff8142c1074ca8060579e300a02273efb72868348ba902c8
```

Project and release:

- [fitgirl-wine repository](https://github.com/BrandowLucas/fitgirl-wine)
- [11.13 release](https://github.com/BrandowLucas/fitgirl-wine/releases/tag/11.13)

The build contains repack-installer compatibility patches for Wine's pipe,
virtual-memory, progress-rendering, and MSVCRT behavior. One MSVCRT patch was
upstreamed in Wine 11.14, but that alone does not make an ordinary Wine 11.14
build equivalent to this complete tested patch set.

## Complete example: Assassin's Creed Shadows DODI

Change the paths before running the commands. This layout keeps the disposable
installer prefix, the compressed source, and the final game directory separate.

### 1. Install basic host tools

On Arch Linux or CachyOS:

```bash
sudo pacman -S --needed curl git tar xz
```

Clone this repository if it is not already available:

```bash
git clone https://github.com/Fr4nzz/linux-repack-gaming.git
cd linux-repack-gaming
```

### 2. Define the paths

```bash
export REPACK_SOURCE="/mnt/Bolt/Descargas/Assassin Creed Shadows [DODI Repack]"
export GAME_DESTINATION="/mnt/Bolt/Games/ACShadows-DODI"
export INSTALL_ROOT="/mnt/Bolt/Games/.installers/ac-shadows-dodi"
export WINEPREFIX="$INSTALL_ROOT/prefix"
export WINEARCH=win64

export WINE_DOWNLOAD="$INSTALL_ROOT/wine-11.13-patched-fitgirl-amd64.tar.xz"
export WINE_ROOT="$INSTALL_ROOT/wine-11.13-patched-amd64"
export WINE="$WINE_ROOT/bin/wine"
export WINESERVER="$WINE_ROOT/bin/wineserver"
```

The Wine `C:` drive will be:

```text
/mnt/Bolt/Games/.installers/ac-shadows-dodi/prefix/drive_c
```

Nothing forces it into `~/.wine`; the `WINEPREFIX` variable explicitly chooses
its Linux location.

### 3. Check storage and source files

```bash
df -h /mnt/Bolt
test -f "$REPACK_SOURCE/Setup.exe"
find "$REPACK_SOURCE" -maxdepth 1 -type f -printf '%12s  %f\n' | sort
```

Leave enough space for the compressed repack, the installed game, temporary
decompression data, and the prefix. Do not assume the installer's advertised
final size is its peak temporary requirement.

### 4. Download and verify the patched Wine build

```bash
mkdir -p "$INSTALL_ROOT"

curl -fL --retry 3 \
  -o "$WINE_DOWNLOAD" \
  "https://github.com/BrandowLucas/fitgirl-wine/releases/download/11.13/wine-11.13-patched-fitgirl-amd64.tar.xz"

printf '%s  %s\n' \
  "709c1a4f027507aeff8142c1074ca8060579e300a02273efb72868348ba902c8" \
  "$WINE_DOWNLOAD" \
  | sha256sum -c -

tar -xJf "$WINE_DOWNLOAD" -C "$INSTALL_ROOT"

"$WINE" --version
```

The last command should report:

```text
wine-11.13-42-gd30dcd75bc9
```

Do not continue if the checksum fails.

### 5. Create the prefix and map source/destination drives

```bash
mkdir -p "$WINEPREFIX" "$GAME_DESTINATION"

"$WINE" wineboot -u

ln -sfn "$REPACK_SOURCE" \
  "$WINEPREFIX/dosdevices/r:"

ln -sfn "$GAME_DESTINATION" \
  "$WINEPREFIX/dosdevices/d:"
```

Inside the installer:

```text
R:\             compressed repack source
D:\             final Linux game directory
C:\             disposable Wine prefix
```

Select a destination such as:

```text
D:\Assassin's Creed Shadows
```

The resulting Linux directory will be:

```text
/mnt/Bolt/Games/ACShadows-DODI/Assassin's Creed Shadows
```

This mapping does not copy the repack or game. Wine drive letters are symbolic
links to the existing Linux directories.

If an installer behaves poorly through a mapped source drive, stage its files
inside the prefix instead:

```bash
mkdir -p "$WINEPREFIX/drive_c/Repack"
cp -a --reflink=auto "$REPACK_SOURCE/." \
  "$WINEPREFIX/drive_c/Repack/"
```

Then run `C:\Repack\Setup.exe`. This was the source layout used during the
diagnostic AC Shadows installation.

### 6. Start the installer with the Oodle fix

From the cloned repository:

```bash
scripts/install-dodi-oodle-fix \
  --wine "$WINE" \
  --prefix "$WINEPREFIX" \
  --setup "$REPACK_SOURCE/Setup.exe"
```

If the source was staged as `C:\Repack`, use:

```bash
scripts/install-dodi-oodle-fix \
  --wine "$WINE" \
  --prefix "$WINEPREFIX" \
  --setup "$WINEPREFIX/drive_c/Repack/Setup.exe"
```

The wrapper:

1. Starts the installer under the chosen Wine build.
2. Watches Inno Setup's temporary directories.
3. Finds the Oodle helpers already embedded in the installer.
4. Temporarily mirrors `oo2recm.exe`, `oo2reck.exe`, and
   `oo2core_7_win64.dll` into Wine's executable search directory.
5. Restores any previous files and removes its temporary copies at exit.

Do not download these proprietary helpers from DLL websites. The matching
versions are already inside the installer.

### 7. Choose components and destination

In the installer:

1. Select the languages and optional content you want.
2. Choose `D:\Assassin's Creed Shadows` as the destination.
3. Disable optional redistributable downloads unless required.
4. Start installation and leave the launching terminal open.

No `WINEDLLOVERRIDES` or Winetricks verbs were required for the successful
tested extraction. Do not add random `isdone.dll` or `unarc.dll` downloads to
the prefix.

### 8. Monitor decompression

In another terminal:

```bash
pgrep -af 'Setup.exe|Setup.tmp|cls-(lolz|srep)|oo2rec'
```

Watch destination growth:

```bash
watch -n 5 'du -sh "/mnt/Bolt/Games/ACShadows-DODI"'
```

During the successful AC Shadows extraction, LOLZ/SREP and Oodle helpers ran and
the destination continued growing beyond the first archive. The earlier broken
run failed immediately near 0%; sustained growth distinguished the fixed
pipeline from that failure.

### 9. Interpret Unarc errors correctly

`ISDone.dll` and `Unarc.dll` codes describe where a decompression pipeline
failed; they do not uniquely identify the cause.

| Behavior | Likely next action |
|---|---|
| Immediate `-11` on the first Oodle-compressed file | Confirm patched Wine and the Oodle wrapper are both in use |
| Installer rejects memory or hangs before useful extraction | Confirm the exact `fitgirl-wine` build and a fresh prefix |
| Failure consistently occurs late at the same file | Verify that archive under native Windows or against published checksums |
| Different files fail on different runs | Check storage, RAM stability, source integrity, and filesystem/kernel logs |
| Destination is not writable | Verify the mapped drive, ownership, mount state, and free space |

The tested AC Shadows repack reached approximately 151 GB before failing at a
specific `dlc_20` archive with `-14`. It failed at the same content under native
Windows. That means:

- The patched Wine and Oodle-path fixes worked.
- The extracted base files were not proof that the entire repack was healthy.
- The late error did not invalidate the Linux installer fix.

### 10. Stop Wine cleanly after an error

Close the installer window first. Then stop only this prefix:

```bash
"$WINESERVER" -k
```

Verify that no process from the prefix remains:

```bash
pgrep -af "$WINEPREFIX|Setup.exe|Setup.tmp|cls-(lolz|srep)|oo2rec" || true
```

Do not use broad commands such as `pkill wine` if another Wine game or
application may be running.

### 11. Verify a completed installation

Inspect the output:

```bash
du -sh "$GAME_DESTINATION"
find "$GAME_DESTINATION" -maxdepth 3 -type f \
  -printf '%12s  %p\n' | sort -nr | head -30
```

Use any file-verification utility included with the repack. A successful
installer exit alone does not guarantee every optional archive was selected or
every game file is intact.

Do not launch the game with this installer Wine prefix by default. Create a
separate runtime prefix and follow
[Launch games with UMU](launch-games-with-umu.md).

### 12. Clean up after the game is verified

The following are disposable after the installed game launches and its files
have been verified:

```text
$WINEPREFIX
$WINE_DOWNLOAD
$WINE_ROOT
```

Keep the patched Wine build in a shared tools directory if more repacks will be
installed. Keep the compressed source until satisfied that no optional language
or DLC archive will be needed later.

Before deleting a partial installation or prefix, inspect it for:

- Save files.
- Optional language packs successfully extracted before a late failure.
- Installer logs needed to diagnose the source archive.

See [Extract one repack component](extract-one-repack-component.md) when only a
language or audio archive is needed.

## Generic template for another repack

After installing and verifying the same patched Wine build:

```bash
export REPACK_SOURCE="/path/to/repack"
export GAME_DESTINATION="/path/to/final/game-parent"
export WINEPREFIX="/path/to/disposable-installer-prefix"
export WINEARCH=win64
export WINE="/path/to/wine-11.13-patched-amd64/bin/wine"

mkdir -p "$WINEPREFIX" "$GAME_DESTINATION"
"$WINE" wineboot -u

ln -sfn "$REPACK_SOURCE" "$WINEPREFIX/dosdevices/r:"
ln -sfn "$GAME_DESTINATION" "$WINEPREFIX/dosdevices/d:"

scripts/install-dodi-oodle-fix \
  --wine "$WINE" \
  --prefix "$WINEPREFIX" \
  --setup "$REPACK_SOURCE/Setup.exe"
```

Choose a `D:\GameName` destination in the installer.

For installers that do not contain Oodle helpers, the wrapper remains harmless
but may not address their failure. Start current ordinary Wine for conventional
installers; use this patched build when the installer exhibits the documented
FreeArc/ISDone compatibility behavior.
