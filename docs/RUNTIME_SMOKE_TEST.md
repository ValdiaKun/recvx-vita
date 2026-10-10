# Runtime smoke test: Resident Evil Code: Veronica X on Vita

This checklist distinguishes a successful cross-compile from a game that actually
boots and renders. A green CI build only proves that the source compiled and a
VPK was packaged; it does not prove runtime correctness.

## Before testing

- Use a PS Vita with homebrew enabled, or a compatible emulator for an initial
  smoke test. Real hardware is required for meaningful performance conclusions.
- Install the VPK produced by the `VitaSDK VPK Build` GitHub Actions workflow.
- Provide your own compatible US PS2 disc image (SLUS-20184) at
  `ux0:data/recvx/recvx.iso`. Do not commit the ISO or extracted copyrighted
  assets to Git.
- Ensure the shader compiler support file required by vitaGL is present at
  `ur0:data/libshacccg.suprx`, as described in the project README.

## Test A — Normal boot

1. Launch RECVX from the LiveArea.
2. Wait for the title screen.
3. Enter gameplay and verify that the scene is not entirely black or corrupted.
4. Check controller input, character movement, camera, audio, and save behavior.
5. Exit normally if possible.
6. Retrieve `ux0:data/recvx/log.txt` and inspect for fatal errors, repeated
   renderer warnings, and the periodic `frame=... fps=...` telemetry.

Record device type, firmware, VPK commit SHA, game region, scene, and observed
frame rate. A title screen alone is not a gameplay pass.

## Test B — Deterministic new-game smoke test

For unattended testing only, create
`ux0:data/recvx/boot.txt` containing the single word `newgame`. This bypasses
the memory-card check and title menu; it does not bypass the need for the game
disc image. Remove this file after the test.

## Test C — Frame capture for rendering verification

1. Create `ux0:data/recvx/capture.txt` with one or more frame numbers, one per
   line (for example, `120` and `300`).
2. Launch the game and allow it to reach the selected frames.
3. Retrieve `ux0:data/recvx/frame-120.ppm` and any other requested captures,
   along with `log.txt`.
4. Inspect captures for visible geometry, textures, text, missing objects,
   incorrect colors, and corruption. Confirm that the capture is not a blank
   frame or an uninitialized render target.
5. Remove `capture.txt` after the test. Frame capture calls `glReadPixels`
   and can be expensive, so do not leave it enabled during performance tests.

The current renderer supports at most 16 frame numbers per run. The capture
function writes PPM images at 960x544.

## Test D — Hang diagnostics

Only when investigating a reproducible hang, create
`ux0:data/recvx/watchdog.txt`. If no frame is presented for 15 seconds, the
watchdog logs thread state and intentionally crashes to produce a core dump.
This is a diagnostic mechanism and should not be enabled during ordinary play.

## Pass criteria

- **Build pass:** CI produces a non-empty `recvx.vpk`.
- **Boot pass:** the application starts and reaches the title screen.
- **Render pass:** a captured gameplay frame contains the expected scene.
- **Input/audio pass:** controls and sound function during gameplay.
- **Stability pass:** a sustained session has no repeatable crash or hang.
- **Performance pass:** record measured FPS and per-frame readback, texture
  decode, and VU1 timings from the log; do not claim a performance target
  without actual measurements.

Attach logs and captures to the relevant GitHub issue or CI artifact, excluding
the game ISO and all copyrighted game data.
