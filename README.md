# dwc2usb.device

A USB host controller driver for **Poseidon**, for Amigas accelerated by a
**PiStorm running Emu68** on a **Raspberry Pi 3-family** board (Pi Zero 2 W,
Pi 3A+). It drives the Pi's own USB port, the Synopsys DWC2 controller, so
Poseidon can use keyboards, mice, hubs, mass storage and USB audio through it.

**Version 0.2, build 1435. This is a test release.** It works well on the
machines it has been tested on, but it has not seen many setups yet. Reports
are welcome: see [Reporting a problem](#reporting-a-problem).

Pi 4 / CM4 users do not need this: use the Emu68 xHCI driver instead.


## What's new in 0.2

- **USB audio now works behind a high-speed USB 2.0 hub**, so a sound card can
  share the Pi's single port with a keyboard, a mouse and a USB stick. 44.1kHz
  works through any hub; 48kHz needs a hub that handles it (see Known issues).
- **Cleaner audio**: the clicks and short repeats some people heard in 0.1 are
  fixed, as are dropouts with players that run at a high priority.
- **Audio survives system stalls**: on setups where dragging a window briefly
  freezes everything (CaffeineOS, for example), the driver now notices and
  keeps more audio queued, by itself, so the sound no longer breaks up.
- **Sturdier with USB drives**: tested with three different USB sticks -
  several gigabytes written and read back, every byte checked, including two
  and three sticks in use at once and a stick pulled out in the middle of
  writing. A fault where a stick dropping out at the wrong moment could freeze
  USB until a reboot is fixed.
- **Safe across a reset**: the USB controller is now stopped cleanly on
  Ctrl-Amiga-Amiga or a software reboot, so it cannot keep running into the
  restarting system.
- Better recovery when a full- or low-speed device behind a hub hits an error.
- `Dwc2Diag` run without a mode now prints its usage instead of running a
  development test.


## What works

- **All four USB transfer types**: control, bulk, interrupt and isochronous.
- **Hubs**, including low- and full-speed devices behind a USB 2.0 hub
  (split transactions).
- **Several devices at once through a hub**: for example keyboard, mouse,
  USB stick and sound card all in use together.
- **Hot-plugging**: unplugging and replugging devices, and whole hubs.
- **USB audio** through `usbaudio.class` and AHI: directly on the Pi's port,
  behind a full-speed hub, or behind a high-speed hub.
- **Mass storage**: about 7 MB/s write and 5.8 MB/s read with a USB 2.0
  stick directly on the port. A USB CD-ROM has also been used.

### Tested with

| | |
|---|---|
| Pi boards | Pi Zero 2 W, Pi 3A+ |
| Emu68 | 1.0.7 |
| AmigaOS | 3.2 |
| Poseidon | 4.5 and 6.1 |
| Devices | SanDisk Cruzer Fit, Lexar and Kingston DataTraveler 3.0 USB sticks, Logitech USB mouse, IBM USB keyboard with TrackPoint, two different USB sound cards, a USB CD-ROM, USB 2.0 hubs with Terminus (1A40:0101) and 214B:7260 chips |

The Pi 3B and 3B+ have not been tested. Their ports sit behind an onboard
high-speed hub, which this release now supports for audio as well.


## Requirements

- A PiStorm with a Pi Zero 2 W or Pi 3-family board, running **Emu68**.
- **Poseidon** (4.5 or later) installed and working.
- For audio: **AHI**, and a USB sound card that Poseidon's `usbaudio.class`
  supports.
- On a Pi Zero 2 W, an **OTG adapter** for its single data port.
- **A powered hub when you use more than one device.** The Pi's port cannot
  power a hub full of devices reliably: with a bus-powered hub, plugging in or
  pulling out a USB stick can make other devices on the hub briefly lose power
  and reset (a mouse stops responding until Poseidon restarts it, for
  example).

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

Upgrading from 0.1: replace the file and reboot.

**For audio**, open `SYS:Prefs/AHI` after the sound card has been detected,
choose the USB audio mode for the unit you want, and click **Use**. The mode
is registered anew each time the card is found, so you may need to click Use
again after a reboot. Set the frequency to match what you play - usually
**44100 Hz** for music.

To check which version is loaded:

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE
```


## Known issues

The full list, with explanations, is in [KNOWN-ISSUES.md](KNOWN-ISSUES.md).
In short:

- **48kHz audio behind a high-speed hub depends on the hub.** Some hubs cannot
  pass it and you will hear constant clicks; 44.1kHz works through all of
  them. Recording (a sound card's microphone) is not supported behind a
  high-speed hub.
- **Audio behind a high-speed hub uses more CPU** than with the sound card
  directly on the port.
- **Low-speed devices (many keyboards and mice) are refused if the hub
  plugged into the Pi runs at full speed.** They work directly on the port,
  or behind a high-speed hub.
- **Clicks in audio when the machine is very busy** can come from AHI holding
  back its mixing to protect the system. KNOWN-ISSUES.md explains how to tell,
  and what helps.
- **The mouse can be slightly less smooth behind a hub during heavy file
  copying** to a USB drive on the same hub.
- **A device that misbehaves on its first connection** (not found, refused,
  or running slowly) usually just needs unplugging and plugging in again.
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

**Only use `PROBE`, `LOG` and `TRACE` with Dwc2Diag.** Its other modes are
development tests, and some of them reset the USB port or disturb Poseidon.
These three only read, and are safe while Poseidon is running.


## Author

Copyright (c) 2026 Leigh "Lemaru" Russ.

Developed with the assistance of Claude AI (Anthropic).

Free to download and use. A licence will be added when the source code is
published.
