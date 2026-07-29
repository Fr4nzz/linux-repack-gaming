# Install repacks with Wine

This guide covers Inno Setup installers that use FreeArc, ISDone, Unarc, LOLZ,
SREP, or Oodle decompression.

## 1. Prepare storage

Keep the installer, Wine prefix, temporary directory, and destination on a
Linux-native filesystem such as Btrfs or ext4. Confirm free space:

```bash
df -h /path/to/source /path/to/destination
```

Create an isolated prefix:

```bash
export WINEPREFIX="/path/to/prefix"
export WINEARCH=win64
wineboot -u
```

Map a large destination as a drive letter:

```bash
ln -sfn "/path/to/game-storage" "$WINEPREFIX/dosdevices/d:"
```

The installer can now use `D:\GameName`.

## 2. Start with an appropriate Wine build

Try a current Wine build first:

```bash
WINEPREFIX="/path/to/prefix" wine "/path/to/Setup.exe"
```

Some FreeArc installers incorrectly reject Wine's memory report. If installation
fails before decompression begins, try a Wine build carrying the FreeArc/ISDone
large-memory-probe compatibility patch.

## 3. Fix immediate DODI Oodle `-11` failures

A misleading error may appear immediately:

```text
ISDone.dll
Unarc.dll returned an error code: -11
archive data corrupted (decompression fails)
```

In the tested failure, DODI extracted `oo2recm.exe`, `oo2reck.exe`, and
`oo2core_7_win64.dll` into one Inno temporary directory while Unarc ran from a
second directory. Wine did not include the helper directory in the child
process search path. The archive itself was readable.

Use the included wrapper:

```bash
scripts/install-dodi-oodle-fix \
  --wine "/path/to/wine" \
  --prefix "/path/to/prefix" \
  --setup "/path/to/Setup.exe"
```

The wrapper:

1. Watches Inno's temporary directories.
2. Copies only the three required transient helpers into Wine's executable
   search directory.
3. Runs the installer.
4. Restores any pre-existing files and removes its temporary copies.

Do not download these proprietary helper files separately. The wrapper uses
only the copies already embedded in the installer.

## 4. Verify

During a working Oodle extraction:

```bash
pgrep -af 'cls-(lolz|srep)|oo2rec'
du -sh "/path/to/destination"
```

You should see LOLZ/SREP and Oodle processes together, and the destination
should continue growing beyond the first output file.

## Troubleshooting

- `-11` is a generic pipeline failure, not proof of corrupt archives.
- Verify source checksums when the distributor provides them.
- Check `dmesg` and the filesystem for I/O errors.
- Avoid changing Wine, CPU topology, RAM limits, and storage paths together.
- If the Oodle wrapper changes nothing, collect process and file traces before
  trying more compatibility variables.
