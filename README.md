# Wireless Corne firmware

This configuration targets nice!nano v2 controllers and nice!view displays on both Corne halves. ZMK and its reusable workflow are pinned to the same v0.3 commit in `config/west.yml` and `.github/workflows/build.yml`. This is a recovery candidate, not a known-good flashed release until both builds and the hardware checks below pass. The upstream workflow uses a mutable build-container tag, so the full toolchain is not yet immutable.

## Build and retrieve firmware

Push a branch or open a pull request; the GitHub Actions build runs for both halves. To rebuild a committed revision, open Actions, select the firmware build workflow, choose **Run workflow**, and select the branch. Open the completed run and download its `firmware` artifact, containing the left and right UF2 files. A successful build validates compilation, not the keyboard's behavior. Do not use a settings-reset firmware for routine upgrades.

Before flashing, retain a copy of firmware known to work on this keyboard, record its source commit and build artifacts, and follow the official ZMK split-keyboard flashing instructions. This repository does not establish which firmware is currently installed. Flashing and any Bluetooth pairing changes require deliberate user action. To roll back, reflash the saved known-working UF2s for the corresponding halves. GitHub Actions artifacts expire; archive verified firmware outside Actions and record the source SHA. Treat a build as known-good only after the checks below.

## Home-row modifiers and verification

`hml`, `hmr`, and `hrm` use `quick-tap-ms = <275>`: if a second press of the same hold-tap follows the first press within 275 ms, ZMK resolves it to the tap behavior so holding that press can repeat the letter. Adjust the three values together if necessary. The previous values were 200 ms for `hml`/`hmr` and no explicit quick-tap for `hrm`; increasing the window can favor a letter over a modifier when repeating a key quickly. `hml` and `hmr` retain their 140 ms prior-idle requirement and positional hold triggers; `hrm` retains its original 150 ms tapping term and no positional rule. Balanced flavor resolves interrupted presses according to the hold-tap timing and key release, while prior idle can resolve a key immediately as a tap during fast typing. Positional rules influence whether an interruption can become a hold. No modifier bindings were changed.

After a successful build and a user-directed flash, check on the actual keyboard:

- Tap A/S/D/F and J/K/L/semicolon individually; hold each modifier and type a key with the opposite hand.
- Tap, release, then hold each of those keys within 275 ms (measured press to press); confirm the OS repeats its letter. Check rapid typing and repeated letters for accidental modifier activation or missed holds.
- Hold Lower, press W+E together, release, then test the sticky function layer with a harmless F key. Check that it exits as before.
- Verify both halves, normal Bluetooth reconnection and profile switching, and both nice!view displays. Avoid reset and clear-pairing combos during this check.

See [ZMK hold-tap documentation](https://zmk.dev/docs/keymaps/behaviors/hold-tap) for timing details. The separate `&key_repeat` behavior is not used: this configuration holds the physical key's tap output for host repeat.

## Experimental mouse layer

This branch adds an experimental toggle-only mouse layer. Hold the outer right thumb (Right Alt), then press Y+P within 50 ms to enter. Release Right Alt before using the layer; otherwise Alt stays held until released. Press Y+P together again to exit. The modifier also temporarily enables a transparent helper layer, so ordinary Y/P typing without that modifier is unchanged. When the mouse layer is active, its Y+P exit combo can delay or consume those keys if pressed together.

- Movement: S/E/D/F and J/I/K/L are left/up/down/right.
- Scroll: W/U up and R/O down.
- Both inner thumbs: left-click; both middle thumbs: right-click; both outer thumbs: middle-click. Mouse buttons are held while the key is held for dragging.
- Movement uses ZMK v0.3 defaults: maximum speed 600, linear acceleration to maximum in 300 ms. These are tunable in `pointing.h` settings and `&mmv`.

The existing default Space/Shift, Enter, home-row mods, Lower, Raise, and function-layer combo remain unchanged. Check Alt shortcuts and timing carefully: the helper uses a zero-wait macro to hold Right Alt and a transparent layer simultaneously. It has not been tested on hardware.

Enabling `CONFIG_ZMK_POINTING` changes the Bluetooth HID descriptor. Test via USB first; hosts may require removing and re-pairing the keyboard before mouse input works over Bluetooth. Do not clear keyboard Bluetooth settings unless needed and explicitly chosen. Reflash the retained prior candidate to roll back; switching HID descriptors can require host re-pairing again. Verify wired and wireless movement, diagonal combinations, acceleration, both scroll directions, each thumb button, two-hand drag, Y+P toggling, ordinary Y/P typing, Right Alt shortcuts, Space/Shift, Enter, home-row mods, and W+E function-layer access. Successful compilation does not verify these physical behaviors.
