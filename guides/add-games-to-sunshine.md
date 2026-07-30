# Add games to Sunshine

Sunshine applications let Moonlight start a game remotely. The wrapper must
remain alive until the game exits so Sunshine can track it.

## 1. Install and start Sunshine

On Arch Linux or CachyOS:

```bash
sudo pacman -S --needed sunshine
systemctl --user enable --now sunshine
```

Check status and logs:

```bash
systemctl --user status sunshine
journalctl --user -u sunshine -n 100 --no-pager
```

Open the local web interface:

```text
https://localhost:47990
```

Accept the local certificate warning, create the Sunshine credentials, and pair
Moonlight before adding several games.

If using a custom configuration directory or a fork such as Apollo, identify
the actual service and config first. Do not edit Sunshine's default files while
another service is the active host.

## 2. Test the launcher locally

Before adding it:

```bash
"/absolute/path/to/launch-my-game"
```

Check:

- The game starts, not just its launcher.
- Correct GPU and resolution are used.
- Local controller input works.
- The wrapper remains alive through launcher-to-game handoff.
- Exiting the game ends the wrapper and all audio.

## 3. Add the application through the web UI

Sunshine's upstream documentation recommends configuring applications through
the web UI.

Open **Applications → Add New** and set:

```text
Application Name: My Game
Command:          /absolute/path/to/launch-my-game
Working Directory:/absolute/path/to/game
```

Leave detached commands empty unless the launcher must outlive the stream. Add
an image if wanted, save, and restart Sunshine only when the UI
requests it.

Use an absolute command and working directory. Avoid:

- A terminal emulator wrapping the launcher.
- `sh -c` with a long quoted Proton command.
- A Windows `.exe` when the tested entry point is a Linux UMU wrapper.
- A command ending in `&`.

Reference:
[Sunshine application examples](https://docs.lizardbyte.dev/projects/sunshine/latest/md_docs_2configuration.html).

## 4. Games with a separate launcher

Some games start a launcher which later spawns or replaces itself with the game.
The Linux wrapper must wait for the final process.

If Moonlight becomes black while the GPU/fans remain active:

1. Check whether the launcher closed but the game is still running.
2. Determine which surface Sunshine is capturing.
3. Check Gamescope and Sunshine logs.
4. Close the complete old process tree before retrying.

Do not solve a process-tracking problem by repeatedly launching additional game
instances.

## 5. Controllers

Enable gamepad input in Sunshine. If Moonlight detects buttons but the game does
not:

```bash
ls -l /dev/input/by-id/
cat /proc/bus/input/devices
```

Then:

1. Confirm Sunshine created the expected virtual controller.
2. Test the same launcher locally.
3. Restart the game after the controller connects so enumeration happens again.
4. Route the launch through Steam only when Steam Input is required.
5. Avoid multiple controller-merging or translation layers.

## 6. Capture and Gamescope

Use Gamescope when the stream needs a fixed resolution, fit/fill behavior, or a
single captureable surface. If direct capture works correctly, it is simpler.

When using Gamescope, Sunshine should invoke the outer wrapper which owns both
Gamescope and the game. Do not background Gamescope.

If two performance overlays appear, disable either in-game MangoHud or outer
Gamescope MangoApp. Normally keep only one.

## 7. Test from Moonlight

Verify:

- Application appears with the correct name.
- One selection starts one game instance.
- Video follows launcher-to-game transition.
- Audio and controller work.
- Resolution and aspect ratio are correct.
- Quitting returns to Moonlight.
- No stale Wine, Gamescope, or audio process remains.

Upstream Sunshine documents the application fields `name`, `cmd`, and
`working-dir`; prefer the UI over direct JSON editing because schemas can change.
