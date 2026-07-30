# Set up CachyOS Gaming Mode

A separate Gaming Mode logs out of Plasma and starts Steam Gamepad UI under an
embedded Gamescope session. It can free memory used by browsers and desktop
applications and removes KWin from the game's presentation path.

It is optional. It did not fix the AC Shadows NVIDIA hang in the tested setup;
that issue required a title-specific Proton/vkd3d workaround.

## 1. Understand the session boundary

Typical muxless-laptop architecture:

```text
SDDM
  → Gamescope DRM session on the display-connected iGPU
    → Steam Gamepad UI
      → Linux per-game wrapper
        → UMU/Proton game rendered on the dGPU
```

Entering Gaming Mode ends the Plasma login. Save desktop work first. Returning
creates a new Plasma session; it does not suspend and restore every process.

## 2. Start with CachyOS's packaged session

Install the supported package:

```bash
sudo pacman -S --needed \
  gamescope-session-cachyos \
  gamescope \
  steam \
  mangohud
```

Check that it installed an SDDM session:

```bash
test -f \
  /usr/share/wayland-sessions/gamescope-session.desktop
```

Save work, log out normally, select the Gamescope session in SDDM, and log in.
Do not configure autologin for the first test.

Test:

1. Steam Gamepad UI appears.
2. Display mode and orientation are correct.
3. Audio, network, and controller work.
4. A lightweight game launches and exits.
5. Exiting Steam returns to SDDM.
6. Plasma still starts normally afterward.

## 3. Hybrid-GPU rules

- Run the outer compositor on the GPU connected to the display.
- Do not export NVIDIA PRIME or NVIDIA ICD variables session-wide.
- Apply dGPU variables only inside each demanding game's launcher.
- Do not enable Gamescope WSI globally because one title needs it.
- Do not add a second nested Gamescope to every game.

Use:

```bash
vulkaninfo --summary
lspci -nnk | grep -A3 -E 'VGA|3D|Display'
```

to identify devices. Never copy another laptop's GPU ID.

The packaged session uses SteamOS-oriented defaults. Check its environment and
test each game.

## 4. Add games

Follow [Add games to Steam](add-games-to-steam.md). Each non-Steam shortcut
should target the same tested Linux wrapper used on the desktop:

- No forced Steam compatibility tool.
- No duplicated Proton command.
- NVIDIA variables scoped inside the game launcher.
- Nested Gamescope disabled in Gaming Mode unless explicitly required.

## 5. Memory and power policy

Keep zram enabled initially. Logging out of Plasma already removes the largest
desktop processes. Disabling zram trades compressed memory capacity for less
compression work and is not a universal gaming optimization.

Use a balanced platform profile unless measurements show a CPU-bound benefit
from performance mode. A GPU-bound game at a 60 FPS cap often gains fan noise
without useful performance.

## 6. Autologin and desktop switchers

One-shot session switching and autologin depend on SDDM. A wrong session name
can return to SDDM or loop back into Gaming Mode.

Only add automation after manual entry/exit works repeatedly. Before changing
SDDM:

```bash
sudo cp -a /etc/sddm.conf /etc/sddm.conf.backup 2>/dev/null || true
sudo cp -a /etc/sddm.conf.d /etc/sddm.conf.d.backup 2>/dev/null || true
```

Review every existing `[Autologin]` section:

```bash
sudo grep -R -n \
  -E '^\\[Autologin\\]|^User=|^Session=|^Relogin=' \
  /etc/sddm.conf /etc/sddm.conf.d 2>/dev/null
```

The repository does not include a universal autologin script because session
names and SDDM settings differ by machine. Use CachyOS's session integration or
a machine-specific script.

## 7. Recovery

If Gaming Mode is black or frozen:

```text
Ctrl+Alt+F3
```

Log in, then:

```bash
loginctl list-sessions
loginctl session-status SESSION_ID
loginctl terminate-session SESSION_ID
```

If SDDM does not recover:

```bash
sudo systemctl restart sddm
```

Restarting SDDM terminates graphical sessions. Use it only after work is saved.

## 8. Acceptance test

Before regular use, complete:

- Three manual logins and clean exits.
- Controller-only navigation and exit.
- One lightweight and one demanding game.
- Correct iGPU compositor/dGPU renderer boundary.
- No duplicate overlays.
- No hidden game process after exit.
- Plasma login and KDE Wallet behavior remain understood.

Keep Plasma as the default/recoverable session until these tests pass.
