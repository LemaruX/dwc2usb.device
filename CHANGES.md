# dwc2usb.device - version history

What changed in each release, newest first. The README describes the current
version; known problems are in [KNOWN-ISSUES.md](KNOWN-ISSUES.md).


## 0.11

- **USB drives read about 70% faster.** Measured on the same stick and hub:
  large files read at about 23 MB/s instead of 13.4, and small reads at about
  15.5 MB/s instead of 9.2. Data read from a drive is now passed to programs
  much more efficiently, which also leaves a little more CPU for other
  programs while a drive is being read.

Writing is unchanged (it is limited by the USB stick itself), and everything
else behaves as in 0.10. Tested with Poseidon 6.2.


## 0.10

- **Fixed: ejecting a USB device plugged straight into the Pi made the Pi's
  port stop working.** After USB > Eject in the Workbench menu, or Eject in
  Trident, nothing plugged into the Pi's port afterwards was recognised - not
  the same device, not another one, not a hub - until Trident was restarted or
  the Amiga rebooted. The driver now notices the device being pulled out, as a
  USB hub does, and the next device is recognised normally. Devices behind a
  hub were never affected.

Everything else behaves as in 0.9. Tested with Poseidon 6.2.


## 0.9

- **Mice, keyboards and other slower devices behind a hub use much less
  CPU.** The driver now times its work for them to the right moment in each
  USB frame instead of waiting for it, and a scheduling fault that split
  their checks into two batches a few milliseconds apart is fixed, so they
  are also checked on time. With a wireless keyboard receiver, a mouse, a
  sound card and a USB stick behind one hub, about a quarter more CPU is left
  for other programs than with 0.8.
- **USB drives behind a busy hub are a little faster** for small transfers.
- **More tolerant of hubs that garble a reply:** a mouse or keyboard reply
  with a data toggle error is now simply asked for again, instead of counting
  towards Poseidon giving up on the device.

Everything else behaves as in 0.8. Tested with Poseidon 6.2 and 4.5.


## 0.8

- **Fixed: new USB devices were sometimes not detected after the Amiga's
  clock was set**, for example by a time sync when going online. The driver
  now keeps time with the Pi's own timer instead of the Amiga's clock.
- **USB drives behind a hub are a little faster** (up to about a fifth for
  small reads), a side effect of the same change.
- **Mice and keyboards behind a hub cope better with a briefly busy system:**
  a poll answered too late is now simply tried again instead of being counted
  as an error.

Everything else behaves as in 0.7. Tested with Poseidon 6.2 and 4.5.


## 0.7

- **Fixed: versions 0.4 to 0.6 stopped recognising devices from
  17 October 2026.** Poseidon waited for ever on any newly plugged-in
  device. Caused by a date-related timing fault - just install 0.7.
- **Mice and keyboards behind a hub** could have stopped being checked for
  the same reason once the first problem was out of the way; also fixed.
- **Fixed: unplugging a hub could leave USB unresponsive.** If a hub was
  pulled out while devices on it were being used, the next device plugged in
  was sometimes not recognised until a reboot, and once in testing the whole
  machine froze. The driver now notices what was left behind and resets the
  Pi's USB controller before the next device is set up.
- **Problem reports now record your setup automatically** - Emu68 version,
  Pi model, Kickstart and Poseidon version - so you do not need to look them
  up.

Everything else behaves as in 0.6. Tested with Poseidon 6.2.


## 0.6

- **Devices behind a hub cost about half the CPU.** Mice, keyboards,
  wireless receivers and a sound card's buttons behind a hub are now checked
  together instead of one after another, which roughly halves the time the
  driver spends on them when no audio is playing - leaving more of the machine
  for everything else.
- **USB drives faster when other devices share the hub.** With a wireless
  keyboard receiver on the same hub, a USB stick read about 40% faster than
  with 0.5.

Mice, keyboards and audio otherwise behave as in 0.5. Tested with Poseidon
4.5 and 6.2.


## 0.5

- **Faster USB drives.** Data now moves in larger pieces, and data being
  written goes to the drive straight from the program's memory instead of
  being copied first. Measured against 0.4 on the same machine on the same
  day: reading up to about 50% faster and writing up to about 75% faster,
  most for large files. (Under Poseidon 6.x, see the note about USB stick
  speed in [Known issues](KNOWN-ISSUES.md).)
- **Fixed a rare fault with full-speed devices that use very small data
  packets** plugged straight into the Pi (most USB drives are not affected).
- **Fixed occasional lost checks of devices behind a hub**, about once every
  two seconds, caused by the USB frame counter wrapping round.

Audio, mice, keyboards and hubs otherwise behave as in 0.4. Tested with
Poseidon 4.5 and 6.2.


## 0.4

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


## 0.3

- Cleaner audio through high-speed hubs, especially at 48kHz: audio is sent
  to a hub at the same point in each USB frame as Linux does.
- **Dwc2Report, a one-click problem report.** See
  [Reporting a problem](README.md#reporting-a-problem).


## 0.2

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


## 0.1

- First public release.
