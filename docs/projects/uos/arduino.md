---
layout: default
title: Arduino
nav_order: 2
parent: UOS

---

# Arduino UOS

![UOS Logo](/assets/images/uos/UOSBlackAndRedSmall.png) 

Arduino UOS allows dynamic run-time control over Arduino devices.
The suggested method for flashing this firmware is using the [arduino-cli](https://arduino.github.io/arduino-cli/latest/) tool.
This can be installed via the `brew` package manager (linux/macOS) or via the `chocolatey` package manager (window). 

With the arduino-cli you want to ensure you have the most up-to-date platform core installed for the microcontroller you plan to use.
Specifics regarding specific hardware are outlined in the `Supported Hardware` section below.

```shell
arduino-cli core update-index
```

You should download the latest stable firmware from the [repository tags](https://github.com/CreatingNull/UOS-Arduino/tags).
Once unpacked, navigate into the root dir and compile / upload to your device (arduino nano on COM3 used in the example).

```shell
cd UOS-Arduino
arduino-cli compile --fqbn='arduino:avr:nano'
arduino-cli upload --fqbn='arduino:avr:nano' --port='COM3'
```

---

## Supported Hardware

### Raspberry Pi Uno

Native support via the `arduino:avr` platform core.

* **FQBN**: `arduino:avr:uno`

### Raspberry Pi Nano

Native support via the `arduino:avr` platform core.

* **FQBN**: `arduino:avr:nano`

### Raspberry Pi Pico

The raspberry pi pico is supported in the arduino uos implementation via the helpful opensource platform core [arduino-pico](https://github.com/earlephilhower/arduino-pico).
This is an un-official 3rd party core, but is easy to use.

* **FQBN**: `rp2040:rp2040:rpipico`

Note: Officially only a vanilla rp2040s on the pico board are support in standard configurations. In theory variations are supported but this is beyond the scope of the project.

#### Getting Started

If you are using the `arduino-ide`, add the arduino-pico URL via the board manager.
If you are using the `arduino-cli`, we can run the following commands to achieve the same (See [installing support for 3rd party board cores](https://create.arduino.cc/projecthub/B45i/getting-started-with-arduino-cli-7652a5)).

Create a config file if one doesn't already exist.

```shell
arduino-cli config init
```

Modify the config to add the arduino-pico core url using your favourite text editor.
This is a simple yaml file so just make sure the `board_managher.additional_urls` list contains the correct URL.

```yaml
board_manager:
  additional_urls: ["https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json"]
```

---

**[Source Code](https://github.com/CreatingNull/UOS-Arduino)**
