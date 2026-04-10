# ZMK Raytac USB Dongle Module

This repository adds ZMK board definitions named `raytac_mdbt50q_rx` for the [Raytac MDBT50Q-RX](https://www.raytac.com/product/ins.php?index_id=89) USB stick and `raytac_mdbt50q_cx_40` for the [Raytac MDBT50Q-CX-40](https://www.raytac.com/product/ins.php?index_id=156) USB stick, with the intention of using them as [ZMK keyboard dongles](https://zmk.dev/docs/development/hardware-integration/dongle).

![IMG_1775](./docs/images/IMG_1775.jpeg)

## Caveat: Buy The RX With A UF2 Bootloader Installed

The stock MDBT50Q-RX does not come with the UF2 bootloader installed—that's the bit of code that allows it to mount itself as a USB drive for flashing.

But! Some versions of the RX dongle do exist with the UF2 bootloader pre-installed, and that's what you want to buy: [Adafruit (USA)](https://www.adafruit.com/product/5199) and [Pi Hut (UK)](https://thepihut.com/products/nrf52840-usb-key-with-tinyuf2-bootloader-bluetooth-low-energy-mdbt50q-rx) sell them, and probably others if you search around. I used the one from Adafruit.

> **Note:** The MDBT50Q-CX-40 does not have a UF2 bootloader option. It ships with Nordic's built-in Open bootloader and is flashed via `nrfutil` over USB serial instead. See [Flashing The CX-40](#flashing-the-cx-40-built-in-bootloader) below.

### Update The Bootloader

I upgraded the version of the bootloader on my Raytac dongle. I don't know if this is absolutely necessary because I upgraded before flashing ZMK onto it, but [someone on Reddit](https://www.reddit.com/r/ErgoMechKeyboards/comments/1k4ejtx/raytac_dongle/norwr6p/) confirmed that they needed to update to get it working. So go ahead and update, it's easy.

I used these [instructions](https://learn.adafruit.com/introducing-the-adafruit-nrf52840-feather/update-bootloader-use-uf2) to update the bootloader to version [0.9.2](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases/tag/0.9.2). Specifically I used [this exact one](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases/download/0.9.2/update-raytac_mdbt50q_rx_bootloader-0.9.2_nosd.uf2).

## ZMK Versioning

ZMK v0.4 will introduce breaking changes which _affect this module and you as a user._

You must [pin your ZMK version](https://zmk.dev/blog/2025/06/20/pinned-zmk) and pin this module to match.

The remainder of this readme will detail how to use the module **prior to ZMK v0.4.**

If you are using ZMK 0.4, please refer to an [later version of this readme](https://github.com/rschenk/zmk-component-raytac-dongle/tree/v0.4).

## Usage for ZMK 0.3 or earlier

Add the following entries to `remotes` and `projects` in `config/west.yml`

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: rschenk
      url-base: https://github.com/rschenk
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3 # <-- ZMK pinned to v0.3
      import: app/west.yml
    - name: zmk-component-raytac-dongle
      remote: rschenk
      revision: v0.3 # <-- This module pinned to the same version as ZMK
  self:
    path: config
```

## Configuring Your Dongle

Follow the setup steps in the [ZMK Keyboard Dongle docs](https://zmk.dev/docs/development/hardware-integration/dongle) to configure your keyboard and dongle. When you get to the [Building the Firmware](https://zmk.dev/docs/development/hardware-integration/dongle#building-the-firmware) step, you will use `board: raytac_mdbt50q_rx` (or `board: raytac_mdbt50q_cx_40` for the CX-40 dongle) in your `build.yml` file like so:

```yaml
include:
  # Config settings for the dongle (use raytac_mdbt50q_rx or raytac_mdbt50q_cx_40)
  - board: raytac_mdbt50q_rx
    shield: my_keyboard_dongle
    
  - board: raytac_mdbt50q_rx
    shield: settings_reset

  # Whatever your keyboard uses...
  - board: nice_nano_v2
    shield: my_keyboard
```

## Flashing The Raytac

### Flashing The RX (UF2 Bootloader)

Entering the bootloader mode is a bit annoying. Unplug the dongle, then hold the button down while plugging it back in.

For some reason you *can't* double-click the reset button like on many boards.

### Flashing The CX-40 (Built-In Bootloader)

The CX-40 is factory-programmed with the Open bootloader from Nordic's nRF5 SDK. You'll use Nordic's `nrfutil` to create firmware packages and flash them to the device. Make sure `nrfutil` is installed before proceeding.

#### Entering Bootloader Mode

Unplug the dongle, then hold the push button while plugging it into USB. The push button is on the far side of the board from the USB connector. Note that the button does not face up—you'll push it from the outside in, towards the USB connector. The red LED should start a fade pattern, signalling the bootloader is running.

#### Packaging and Flashing

After building your ZMK firmware (which produces a `.hex` file in `build/zephyr/zephyr.hex`), package it for the bootloader:

```sh
nrfutil nrf5sdk-tools pkg generate \
        --hw-version 52 \
        --sd-req=0x00 \
        --application build/zephyr/zephyr.hex \
        --application-version 1 \
        firmware.zip
```

Then flash it onto the board over USB serial:

```sh
nrfutil nrf5sdk-tools dfu usb-serial -pkg firmware.zip -p /dev/ttyACM0
```

> **Note:** The serial port will vary by OS. It's `/dev/ttyACM0` on Linux, something like `COMx` on Windows, and something like `/dev/cu.usbmodemXXXX` on macOS.

For more information, see [Nordic Semiconductor USB DFU](https://docs.zephyrproject.org/latest/boards/raytac/mdbt50q_cx_40_dongle/doc/index.html#option-1-using-the-built-in-bootloader-only) in the Zephyr docs.
