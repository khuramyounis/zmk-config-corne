# Corne ZMK config — agent guardrails

This repo is **ZMK**, not QMK. Toolchain is Zephyr + west, not avr-gcc.
Builds run on **GitHub Actions** via `zmkfirmware/zmk/.github/workflows/build-user-config.yml`. There is no local build path — every change is verified by pushing to `main` and watching the workflow.

## Hard rules

- **Never add `cmake-args` flags** without citing a real symbol in upstream ZMK's `Kconfig` tree. Invented flags (notably `CONFIG_ZMK_BOARD_COMPAT` — does not exist) waste build cycles and lie in commit history. The build was broken from 2026-04-29 to 2026-05-01 by exactly this mistake.
- **On red builds, read the actual error first**: `gh run view --log-failed`. Do not stack workarounds. The error is in the log; the fix is in the log.
- **Board slug is `nice_nano/nrf52840/zmk`** (Zephyr hwmv2 syntax). The legacy `nice_nano_v2` board name does not exist on current ZMK main — the v2 hardware is the default revision of the `nice_nano` board.
- **`config/west.yml` pins ZMK to a specific commit SHA**, not `main`. Reason: drift. If a bump is needed, copy a known-good SHA from a maintained config like `urob/zmk-config` rather than tracking `main`.
- **ZMK Studio is enabled** (`config/corne.conf:12`). Bindings (key positions on existing layers) can be edited live via Studio with no rebuild. Behaviors (hold-taps, combos, macros, custom hold-tap definitions) and layer-count changes require a rebuild + reflash.
- **Comments stay verbose.** Each behavior, layer, and timing parameter has a why-comment explaining the choice; do not strip them as "noise" — future-you needs them.

## Files an agent may edit without asking

- `config/corne.keymap`
- `config/corne.conf`
- `build.yaml`
- `CLAUDE.md` (this file)

## Files that require explicit permission

Anything else — `.github/workflows/`, `config/west.yml` (manifest changes have repo-wide blast radius), `boards/`, `zephyr/`, `zmk/`. Ask first.

## Vim-canon scope (active 2026-05-01 → ~2026-07-01)

Strict canon. **No** home-row mods, **no** combos beyond Esc, **no** custom enhancements, **no** "while we're here" tweaks. Goal is muscle memory, not optimization. Re-evaluate at week 8 (~2026-07-01).

The vim Nav semantics live on the **existing `lower_layer`** (held by the left-middle thumb, `&mo 1`) — not a separate dedicated layer. This follows the dominant Corne community pattern (the ZMK Corne template ships with arrows in this exact spot) and the canonical opposite-hand split-keyboard rule: left thumb activates, right hand types.

Cross-keyboard symmetry is **semantic, not letter-positional**: ProtoArc/MacBook use Karabiner's Caps layer with the canonical letter mapping (`Caps+w/b` = word, `Caps+u/d` = page, `Caps+0/$` = line edges). Corne's lower layer is shared with numbers + Bluetooth, so the Nav semantics on the right hand are placed by ergonomics (END right of RIGHT, HOME under LEFT, PgDn/PgUp stacked under DOWN/UP, ⌘← / ⌘→ on the outside). The trigger SIDE matches across keyboards (left), but the destination KEYS differ between Corne and Karabiner.

## How a build is verified

```
gh run watch --exit-status -R khuramyounis/zmk-config-corne
```
Exit 0 = green. Both `corne_left*.uf2` and `corne_right*.uf2` artifacts download from the run page or via `gh run download <id>`.

## How to flash a new build

1. Plug one half via USB, double-tap the reset button. The half mounts as a USB mass-storage device named `NICENANO`.
2. Drag the matching `.uf2` (left or right) onto the volume.
3. The half reboots and rejoins the BLE pair automatically.
4. Repeat for the other half.

A reflash is needed for any change to behaviors, layer count, or hold-tap parameters. Binding-only changes (different key on an existing layer) can ride live via ZMK Studio without flashing.
