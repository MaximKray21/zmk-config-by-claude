# zmk-keyboard-tbk-mini

ZMK firmware module for the [TBK Mini](https://bastardkb.com/tbk-mini/) split keyboard by BastardKB.

## Hardware

| Property      | Value                                      |
|---------------|--------------------------------------------|
| Layout        | 3×6 + 3 thumb keys per side (42 keys total)|
| Controller    | Pro Micro compatible (e.g. nice!nano v2)   |
| Connection    | Wireless BLE (ZMK split)                   |
| Matrix        | col2row, diodes from column to row         |

## Directory structure

```
zmk-keyboard-tbk-mini/
├── .github/
│   └── workflows/
│       └── build.yml              # GitHub Actions – builds firmware on push
├── boards/
│   └── shields/
│       └── tbk_mini/
│           ├── Kconfig.defconfig  # Default Kconfig values for the shield
│           ├── Kconfig.shield     # Shield name definitions
│           ├── tbk_mini.conf      # User-facing config suggestions
│           ├── tbk_mini.dtsi      # Shared devicetree (matrix transform, kscan rows)
│           ├── tbk_mini.keymap    # Default keymap (4 layers)
│           ├── tbk_mini.zmk.yml   # ZMK hardware metadata
│           ├── tbk_mini_left.overlay   # Left half – column GPIO assignments
│           └── tbk_mini_right.overlay  # Right half – column GPIO assignments + col-offset
├── zephyr/
│   └── module.yml                 # Zephyr module descriptor
├── build.yaml                     # Boards/shields to build via GitHub Actions
├── west.yml                       # West manifest (fetches ZMK)
└── README.md
```

## Keymap layers

| Layer | Name | Activated by          |
|-------|------|-----------------------|
| 0     | Base | default               |
| 1     | Nav  | hold left thumb (NAV) |
| 2     | Sym  | hold right thumb (SYM)|
| 3     | Sys  | Nav + Sym simultaneously|

## Pin mapping

Adjust the GPIO pin numbers in [`tbk_mini.dtsi`](boards/shields/tbk_mini/tbk_mini.dtsi),
[`tbk_mini_left.overlay`](boards/shields/tbk_mini/tbk_mini_left.overlay) and
[`tbk_mini_right.overlay`](boards/shields/tbk_mini/tbk_mini_right.overlay) to match
your actual wiring.

Default mapping (Pro Micro labels):

| Signal | Pro Micro pin |
|--------|---------------|
| Row 0  | 4             |
| Row 1  | 5             |
| Row 2  | 6             |
| Row 3  | 7             |
| Col 0  | 8             |
| Col 1  | 9             |
| Col 2  | 10            |
| Col 3  | 11            |
| Col 4  | 14            |
| Col 5  | 15            |

## Building

### GitHub Actions (recommended)

1. Fork / clone this repository to your GitHub account.
2. Edit `build.yaml` to select your controller board (default: `nice_nano_v2`).
3. Push a commit – GitHub Actions will build the firmware automatically.
4. Download the `.uf2` files from the Actions artifacts.

### Local toolchain

```bash
# Initialise west workspace
west init -l .
west update

# Build left half
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=tbk_mini_left -DZMK_CONFIG="$(pwd)"

# Build right half
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=tbk_mini_right -DZMK_CONFIG="$(pwd)"
```

## Using as an external module

If you keep the shield in a separate repository and reference it from a `zmk-config`,
add the following to your config's `west.yml`:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: your-github-user
      url-base: https://github.com/your-github-user
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-keyboard-tbk-mini
      remote: your-github-user
      revision: main
  self:
    path: config
```

Then in your `build.yaml`:

```yaml
---
include:
  - board: nice_nano_v2
    shield: tbk_mini_left
  - board: nice_nano_v2
    shield: tbk_mini_right
```

## License

MIT
