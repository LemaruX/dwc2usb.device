# dwc2usb.device

A USB host controller driver for **Poseidon**, for Amigas accelerated by a
**PiStorm running Emu68** on a **Raspberry Pi 3-family** board (Pi Zero 2 W,
Pi 3A+). It drives the Pi's own USB port, the Synopsys DWC2 controller, so
Poseidon can use keyboards, mice, hubs, mass storage and USB audio through it.

**Version 0.4, build 1605. This is a test release.** It works well on the
machines it has been tested on, but it has not seen many setups yet. Reports
are welcome: see [Reporting a problem](#reporting-a-problem).

Pi 4 / CM4 users do not need this: use the Emu68 xHCI driver instead.


## What's new in 0.4

- **Smoother mice and keyboards behind a hub while audio plays.** Each device
  behind a hub now gets its own moment in the USB frame instead of competing
  for the same one, so far fewer of their reports are lost.
- **No more freezes from busy hub devices.** Devices behind a hub that asked to
  be checked every 1-2ms could take over the whole machine. Devices behind a
  hub are now checked at most every 8ms, and the time the driver spends on
  them is capped. Devices plugged straight into the Pi are not affected.
- **One slow device no longer holds up the others.** Setup requests to a
  device now run in the background, so a device that is slow to answer no
  longer stalls every other USB device meanwhile.
- **Unplugging a hub is handled much faster.** A device plugged in straight
  afterwards appears in well under a second instead of 30 seconds or more.
- **Poseidon 6.2 power saving works on the Pi's port**: suspend and resume,
  and no more "Remote wakeup feature for 'DWC2 Hub' could not be enabled".
- More robust error handling, following the USB specification and Linux.

Audio, USB drives and devices plugged straight into the Pi otherwise behave as
in 0.3.


## What was new in 0.3

- Cleaner audio through high-speed hubs, especially at 48kHz: audio is sent
  to a hub at the same point in each USB frame as Linux does.
- **Dwc2Report, a one-click problem report.** See
  [Reporting a problem](#reporting-a-problem).


## What was new in 0.2

- USB audio behind a high-speed USB 2.0 hub, so a sound card can share the
  Pi's single port with a keyboard, a mouse and a USB stick.
- Cleaner audio: the clicks and short repeats some people heard in 0.1 are
  fixed, as are dropouts with players that run at a high priority.
- Audio survives system stalls (CaffeineOS window drags, for example) by
  keeping more audio queued when it is needed.
- Sturdier with USB drives, tested with several gigabytes written and read
  back, every byte checked; a stick dropping out at the wrong moment no longer
  freezes USB.
- The USB controller is stopped cleanly on a reset.


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
| Poseidon | 4.5, 6.1 and 6.2 |
| Devices | SanDisk Cruzer Fit, Lexar and Kingston DataTraveler 3.0 USB sticks, Logitech USB mouse, Logitech Unifying receiver (K400 keyboard), IBM USB keyboard with TrackPoint, two different USB sound cards, a USB CD-ROM, USB 2.0 hubs with Terminus (1A40:0101), 214B:7260 and Genesys (05E3:0608) chips |

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
  copying** to a USB drive on the same hub.
- **Some hubs lose audio or upset mice and keyboards**, while others with the
  same maker's chips work fine (for example, a Genesys "USB2.1 Hub" fails
  where a Genesys "USB2.0 Hub" works), and one user's machine froze with a
  sound card and a mouse on a Genesys "USB2.1 Hub". A plain USB 2.0 hub is
  the safest choice for a sound card, mouse or keyboard.
- **FAT-formatted USB sticks are slower under Poseidon 6.x than under 4.5.** That is in
  Poseidon's mounter, not this driver, and has been reported to Poseidon.
- **A device that misbehaves on its first connection** (not found, refused,
  or running slowly) usually just needs unplugging and plugging in again.
- **Duplicate drive numbers after eject or media change** (`UCD0`, `UCD1`...)
  come from `massstorage.class` and happen with other USB drivers too.


## Reporting a problem

Straight after the problem happens, **double-click `Dwc2Report`**, once. It
collects the driver build, the driver's log, the last mouse/keyboard reports,
Poseidon's device list and Poseidon's message log into one file, and shows a
window telling you its name, for example `RAM:dwc2report-11-Oct-26_10.42.13.txt`.

Then open an issue on this repository, attach that file, and say:

1. Your Pi model, Emu68 version, AmigaOS version and Poseidon version.
2. What is plugged in and how: directly, or through which hub, and whether the
   hub has its own power supply.
3. What you did, what you expected and what happened instead.

Good to know:

- **Run it once, right after the problem.** Reading Poseidon's message log
  empties it, so a second run has nothing to show there.
- For audio problems, **quit the player completely first**. Some players keep
  the audio open while stopped, and the audio figures are only written when it
  closes.
- From a shell, `Execute Dwc2Report` does the same. It finds Dwc2Diag in `C:`,
  next to itself, or in the archive's `C` folder.

If you prefer to do it by hand, these Dwc2Diag modes only read and are safe
while Poseidon is running:

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE
Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG >RAM:dwc2log.txt
Dwc2Diag DEVS:USBHardware/dwc2usb.device TRACE >RAM:dwc2trace.txt
```

**Only use `PROBE`, `LOG` and `TRACE`.** Dwc2Diag's other modes are
development tests, and some of them reset the USB port or disturb Poseidon.


## Author

Copyright (c) 2026 Leigh "Lemaru" Russ.

Developed with the assistance of Claude AI (Anthropic).

Free to download and use. A licence will be added when the source code is
published.
