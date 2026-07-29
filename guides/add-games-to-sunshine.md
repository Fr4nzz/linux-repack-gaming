# Add games to Sunshine

Sunshine applications let Moonlight start a game remotely and stop its complete
process tree when the stream ends.

## Before adding the game

Verify locally that:

- The normal launcher works.
- The game uses the intended GPU.
- Controllers work.
- The launcher exits when the game exits.

## Application fields

Create a Sunshine application with:

- **Name:** the proper game name
- **Command:** absolute path to the tested wrapper
- **Working directory:** the game directory
- **Image:** optional cover art
- **Detached commands:** empty unless specifically required

Use a wrapper that keeps ownership of the process tree. Avoid launching the game
through a terminal emulator.

## Games with launchers

Some games open a launcher and then replace it with the game window. Sunshine
must keep tracking the wrapper until the final game process exits. Do not make
the launcher script return immediately after starting the first executable.

## Controllers

Enable controller input in Sunshine. If Moonlight detects the controller but the
game does not:

1. Confirm the virtual controller exists on the host.
2. Test the game from the same wrapper locally.
3. Route the Sunshine launch through Steam when Steam Input is required.
4. Avoid starting multiple headless desktop sessions solely to obtain input.

## Capture and scaling

Use Gamescope when the stream needs a fixed resolution, fit/fill behavior, or a
captureable window. If a game is black only in Moonlight, first determine whether
Sunshine is capturing the launcher, desktop, or final game surface.
