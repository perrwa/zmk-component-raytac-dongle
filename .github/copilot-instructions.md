# Copilot Instructions

Zephyr module providing ZMK board definitions for two Raytac nRF52840 USB dongles (`raytac_mdbt50q_rx`, `raytac_mdbt50q_cx_40`) used as ZMK keyboard dongles. Consumed as a West module—no standalone build.

## Architecture

`zephyr/module.yml` sets `board_root: .` so Zephyr discovers boards under `boards/arm/<board_name>/`. Each board has five files:

- `Kconfig.board` — board symbol + SoC dependency
- `Kconfig.defconfig` — conditional defaults (USB stack, BT controller)
- `Kconfig` — board-level options (DC-DC converter)
- `<board_name>_defconfig` — SoC, build output format, NVS, BT power/PHY
- `<board_name>.dts` — flash partitions, GPIO (LED/button), USB CDC ACM console

## Board Differences

RX: UF2 bootloader, `BUILD_OUTPUT_UF2`, SoftDevice partition (152K) + 796K app, LED P1.13 active-high, button P0.15
CX-40: Nordic Open Bootloader, `BUILD_OUTPUT_HEX`, MBR (4K) + 944K app, LED P0.06 active-low, button P1.06

## Conventions

- Both boards: nRF52840 QIAA, identical Kconfig structure—mirror existing pattern for new boards
- Aliases: `sw0` (button), `led0` (LED), `zephyr_udc0` (USB), `zephyr,cdc-acm-uart` (console)
- NVS: 7 sectors (28K). BT: max TX power (`PLUS_8`), Coded PHY
- Version tags/branches match ZMK releases (`v0.3`, `v0.4`)
- Conventional commits scoped to board: `feat:`, `fix(cx40):`, `docs:`

## Workflow

- **PRs target origin** (`perrwa/zmk-component-raytac-dongle`) by default. Only open against upstream (`rschenk/zmk-component-raytac-dongle`) when explicitly asked.
- **No CI in this repo** — builds are tested via `perrwa/zmk-config` GitHub Actions. Don't look for local build/test commands.
- **Debugging**: State the error and goal before investigating. Don't exhaustively catalog upstream files — investigate incrementally.
