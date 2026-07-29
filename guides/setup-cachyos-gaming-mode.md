# Set up CachyOS Gaming Mode

A separate Gaming Mode logs out of Plasma and starts Steam Gamepad UI under one
Gamescope session. It can free memory used by browsers and desktop applications
and avoids KWin interactions while playing demanding games.

## Architecture

On a muxless AMD/NVIDIA laptop:

```text
SDDM
  → Gamescope DRM session on AMD
    → Steam Gamepad UI
      → per-game UMU wrapper
        → game rendered on NVIDIA
```

Keep Plasma as the normal/default session during testing.

## Practical requirements

- Add a separate SDDM Wayland session entry.
- Run outer Gamescope on the GPU connected to the internal panel.
- Do not export NVIDIA PRIME variables session-wide.
- Do not enable Gamescope WSI globally; scope it per game.
- Do not wrap every game in a second Gamescope instance.
- Keep zram enabled.
- Provide a reliable `loginctl terminate-session` exit shortcut.

## Session switching

A desktop “Enter Gaming Mode” shortcut can:

1. Save the Gaming Mode session as the next SDDM session.
2. Configure one-shot autologin for the current user.
3. Log out of Plasma.

Returning to desktop reverses the selection and terminates Gaming Mode. Test this
several times before relying on it. An incorrect SDDM setup can simply return to
the login screen or ask for a password instead of starting Steam.

## Recovery

If Gaming Mode is black or frozen:

```text
Ctrl+Alt+F3
```

Then:

```bash
loginctl list-sessions
loginctl terminate-session SESSION_ID
sudo systemctl restart sddm
```

Restarting SDDM terminates graphical sessions, so use it only after work is
saved.

