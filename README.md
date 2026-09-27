# dwc2usb.device

A USB host controller driver for **Poseidon**, for Amigas accelerated by a
**PiStorm running Emu68** on a **Raspberry Pi 3-family** board (Pi Zero 2 W,
Pi 3A+). It drives the Pi's own USB port, the Synopsys DWC2 controller, so
Poseidon can use keyboards, mice, hubs, mass storage and USB audio through it.

**Version 0.1, build 1832. This is an early test release.** It works well on
the machines it has been tested on, but it has not seen many setups yet.
Reports are welcome: see [Reporting a problem](#reporting-a-problem).

Pi 4 / CM4 users do not need this: use the Emu68 xHCI driver instead.


## What works

- **All four USB transfer types**: control, bulk, interrupt and isochronous.
- **Hubs**, including low- and full-speed devices behind a USB 2.0 hub
  (split transactions).
- **Several devices at once through a hub**: for example keyboard, mouse and
  a USB stick all in use together.
- **Hot-plugging**: unplugging and replugging devices, and whole hubs.
- **USB audio** through `usbaudio.class` and AHI, with the sound card directly
  on the Pi's port or behind a full-speed hub (see Known issues).
- **Mass storage**: about 6.8 MB/s write and 5.5 MB/s read with a USB 2.0
  stick directly on the port. A USB CD-ROM has also been used.

### Tested with

| | |
|---|---|
| Pi boards | Pi Zero 2 W, Pi 3A+ |
| Emu68 | 1.0.7 |
| AmigaOS | 3.2 |
| Poseidon | 4.5 and 6.1 |
| Devices | SanDisk Cruzer Fit, Logitech USB mouse, IBM USB keyboard with TrackPoint, two different USB sound cards, a USB CD-ROM, a 4-port USB 2.0 hub |

The Pi 3B and 3B+ have not been tested. Their ports sit behind an onboard
high-speed hub, so everything should work except USB audio (see Known issues).


## Requirements

- A PiStorm with a Pi Zero 2 W or Pi 3-family board, running **Emu68**.
- **Poseidon** (4.5 or later) installed and working.
- For audio: **AHI**, and a USB sound card that Poseidon's `usbaudio.class`
  supports.
- On a Pi Zero 2 W, an **OTG adapter** for its single data port. A **powered
  hub** is recommended once you use more than one device.

**Use only one driver for the Pi's USB port.** If you have another driver for
it installed, remove or disable it in Trident first. Two drivers must never
run the same controller.


## Installing

1. Copy `Devs/USBHardware/dwc2usb.device` to `DEVS:USBHardware/`.
2. Copy `C/Dwc2Diag` to `C:` (optional, but needed to report problems).
3. Open **Trident**, go to the **Hardware** page, add a new entry, pick
   `DEVS:USBHardware/dwc2usb.device`, unit **0**, and bring it online.
   Save the settings if you want it started at every boot.

Or, from a shell, without changing any saved settings:

```
AddUSBHardware DEVICE=DEVS:USBHardware/dwc2usb.device UNIT=0
AddUSBClasses
```

The root hub shows up in Poseidon as **DWC2 Hub**.

**For audio**, open `SYS:Prefs/AHI` after the sound card has been detected,
choose the USB audio mode for the unit you want, and click **Use**. The mode
is registered anew each time the card is found, so you may need to click Use
again after a reboot.

To check which version is loaded:

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE
```


## Known issues

The full list, with explanations, is in [KNOWN-ISSUES.md](KNOWN-ISSUES.md).
In short:

- **USB audio is refused behind a high-speed hub.** It works directly on the
  port and behind a hub that runs at full speed. Poseidon's device list shows
  which one yours is.
- **Low-speed devices (many keyboards and mice) are refused if the hub
  plugged into the Pi runs at full speed.** The Pi's controller cannot reach
  them that way. They work directly on the port, or when the hub on the Pi is
  a high-speed one. A sound card and a low-speed mouse therefore cannot
  currently share a hub.
- **Occasional clicks in audio when the machine is very busy** can come from
  `usbaudio.class` itself, not this driver. KNOWN-ISSUES.md explains how to
  tell which, and what helps.
- **The mouse can be slightly less smooth behind a hub during heavy file
  copying** to a USB drive on the same hub.
- **Duplicate drive numbers after eject or media change** (`UCD0`, `UCD1`...)
  come from `massstorage.class` and happen with other USB drivers too.


## Reporting a problem

Please open an issue on this repository and include:

1. The build number, from `Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE`.
2. Your Pi model, Emu68 version, AmigaOS version and Poseidon version.
3. What is plugged in and how: directly, or through which hub. The output of
   `PsdDevlister` is ideal.
4. What you did, what you expected and what happened instead.
5. The driver's own log, captured straight after the problem:

   ```
   Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG >RAM:dwc2log.txt
   ```

   For audio problems, **quit the player completely first**. Some players
   keep the audio open while stopped, and the audio figures are only written
   when it closes.

**Only use `PROBE` and `LOG` with Dwc2Diag.** Its other modes are
development tests, and some of them reset the USB port or disturb Poseidon.
`LOG` only reads, and is safe while Poseidon is running.


## Author

Copyright (c) 2026 Leigh "Lemaru" Russ.

Developed with the assistance of Claude AI (Anthropic).

Free to download and use. A licence will be added when the source code is
published.
