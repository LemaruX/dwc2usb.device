# dwc2usb.device

A USB host controller driver for **Poseidon**, for Amigas accelerated by a
**PiStorm running Emu68** on a **Raspberry Pi 3-family** board (Pi Zero 2 W,
Pi 3A+). It drives the Pi's own USB port, the Synopsys DWC2 controller, so
Poseidon can use keyboards, mice, hubs, mass storage and USB audio through it.

**Version 0.10, build 1507. This is a test release.** It works well on the
machines it has been tested on, but it has not seen many setups yet. Reports
are welcome: see [Reporting a problem](#reporting-a-problem).

Pi 4 / CM4 users do not need this: use the Emu68 xHCI driver instead.


## What's new in 0.10

- **Fixed: ejecting a USB device plugged straight into the Pi made the Pi's
  port stop working.** After USB > Eject in the Workbench menu, or Eject in
  Trident, nothing plugged into the Pi's port afterwards was recognised - not
  the same device, not another one, not a hub - until Trident was restarted or
  the Amiga rebooted. The driver now notices the device being pulled out, as a
  USB hub does, and the next device is recognised normally. Devices behind a
  hub were never affected.

Everything else behaves as in 0.9. Tested with Poseidon 6.2.

The changes in earlier versions are listed in [CHANGES.md](CHANGES.md).

## What works

- **All four USB transfer types**: control, bulk, interrupt and isochronous.
- **Hubs**, including low- and full-speed devices behind a USB 2.0 hub
  (split transactions).
- **Several devices at once through a hub**: for example keyboard, mouse,
  USB stick and sound card all in use together.
- **Hot-plugging**: unplugging and replugging devices, and whole hubs.
- **USB audio** through `usbaudio.class` and AHI: directly on the Pi's port,
  behind a full-speed hub, or behind a high-speed hub.
- **Mass storage**: with large files, up to about 12 MB/s reading and
  11 MB/s writing with a USB 2.0 stick behind a hub (Poseidon 4.5, or 6.x
  once its stick-speed problem is fixed - see Known issues). A USB CD-ROM
  has also been used.

### Tested with

| | |
|---|---|
| Pi boards | Pi Zero 2 W, Pi 3A+ |
| Emu68 | 1.0.7 |
| AmigaOS | 3.2 |
| Poseidon | 4.5, 6.1 and 6.2 |
| Devices | SanDisk Cruzer Fit, Lexar and Kingston DataTraveler 3.0 USB sticks, Logitech USB mouse, Logitech Unifying receiver (K400 keyboard), IBM USB keyboard with TrackPoint, two different USB sound cards, a USB CD-ROM, USB 2.0 hubs with Terminus (1A40:0101), 214B:7260, WCH (1A86:8098) and Genesys (05E3:0608) chips |
| Hubs found faulty | Genesys 05E3:0610 ("USB2.1 Hub") - see Known issues |

The Pi 3B and 3B+ have not been tested. Their ports sit behind an onboard
high-speed hub, which this driver supports for audio as well.


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
2. Copy `C/Dwc2Diag` to `C:`, and `Dwc2Report` with its icon anywhere you
   like (the Workbench, for example). Both are only needed to report problems.
3. Open **Trident**, go to the **Hardware** page, add a new entry, pick
   `DEVS:USBHardware/dwc2usb.device`, unit **0**, and bring it online.
   Save the settings if you want it started at every boot.

Or, from a shell, without changing any saved settings:

```
AddUSBHardware DEVICE=DEVS:USBHardware/dwc2usb.device UNIT=0
AddUSBClasses
```

The root hub shows up in Poseidon as **DWC2 Hub**.

Upgrading from an earlier version: replace the file and reboot.

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

- **Some hubs cannot pass 48kHz audio cleanly.** If you hear constant clicks
  at 48kHz behind a hub, use 44.1kHz. Recording (a sound card's microphone)
  is not supported behind a high-speed hub.
- **Audio behind a high-speed hub uses more CPU** than with the sound card
  directly on the port.
- **Low-speed devices (many keyboards and mice) are refused if the hub
  plugged into the Pi runs at full speed.** They work directly on the port,
  or behind a high-speed hub.
- **Clicks in audio when the machine is very busy** can come from AHI holding
  back its mixing to protect the system. KNOWN-ISSUES.md explains how to tell,
  and what helps.
- **Devices behind a hub are checked at most every 8ms.** Fine for mice and
  keyboards; a device that needs faster polling works better plugged straight
  into the Pi.
- **The mouse can be slightly less smooth behind a hub during heavy file
  copying** to a USB drive on the same hub, or while audio plays, and **busy
  devices behind a hub slow a USB drive on the same hub** (much less since
  0.6).
- **Hubs with the Genesys 05E3:0610 chip ("USB2.1 Hub", often inside USB 3.0
  hubs) corrupt data from mice, keyboards and sound cards:** ghost mouse
  clicks, a jumpy pointer, rough scratchy audio. USB sticks are fine on them.
  A plain USB 2.0 hub, or the Pi's own port, is the safe choice for a sound
  card, mouse or keyboard.
- **USB sticks are much slower under Poseidon 6.x than under 4.5** (half the
  speed or worse). That is in Poseidon's `massstorage.class`, not this driver,
  and has been reported to Poseidon with a fix.
- **A device that misbehaves on its first connection** (not found, refused,
  or running slowly) usually just needs unplugging and plugging in again.
- **Duplicate drive numbers after eject or media change** (`UCD0`, `UCD1`...)
  come from `massstorage.class` and happen with other USB drivers too.
- **After USB > Eject, a stick plugged back in may not get an icon** until
  Trident is restarted, although it is mounted and usable. This happens with
  other USB drivers too. Pulling the stick out without ejecting avoids it.
- **If Workbench stops responding after you eject a USB stick,** update the
  fat95 filesystem: an old fat95 (3.18) caused exactly that, and 4.1 fixes it.


## Reporting a problem

Straight after the problem happens, **double-click `Dwc2Report`**, once. It
collects the driver build, the driver's log, the last mouse/keyboard reports,
Poseidon's device list and Poseidon's message log into one file, and shows a
window telling you its name, for example `RAM:dwc2report-11-Oct-26_10.42.13.txt`.

Then open an issue on this repository, attach that file, and say:

1. What is plugged in and how: directly, or through which hub, and whether the
   hub has its own power supply.
2. What you did, what you expected and what happened instead.

(The report already contains your Pi model, Emu68, Kickstart and Poseidon
versions.)

Good to know:

- **Run it once, right after the problem.** Reading Poseidon's message log
  empties it, so a second run has nothing to show there.
- For audio problems, **quit the player completely first**. Some players keep
  the audio open while stopped, and the audio figures are only written when it
  closes.
- From a shell, `Execute Dwc2Report` does the same. It uses the Dwc2Diag in
  the archive's `C` folder or next to itself first, then the one in `C:`.

If you prefer to do it by hand, these Dwc2Diag modes only read and are safe
while Poseidon is running:

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE
Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG >RAM:dwc2log.txt
Dwc2Diag DEVS:USBHardware/dwc2usb.device TRACE >RAM:dwc2trace.txt
Dwc2Diag DEVS:USBHardware/dwc2usb.device SYSTEM
```

**Only use `PROBE`, `LOG`, `TRACE` and `SYSTEM`.** Dwc2Diag's other modes are
development tests, and some of them reset the USB port or disturb Poseidon.


## Author

Copyright (c) 2026 Leigh "Lemaru" Russ.

Developed with the assistance of Claude AI (Anthropic).

Free to download and use. A licence will be added when the source code is
published.

If you find it useful and want to support its development:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M7CX5SO)
