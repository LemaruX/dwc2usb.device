# Known issues

Things that are understood, reproducible, and either cannot be fixed in this
driver or have not been fixed yet. If you hit something that is not listed
here, it is worth reporting.


## 48kHz audio behind a high-speed hub depends on the hub

At 48kHz each USB audio packet is too big to cross a high-speed hub's
transaction translator in one piece, so it is sent in two parts that the hub
has to join back together. Version 0.3 sends those parts at the moments Linux
and other USB controllers use, and both hubs tested here now play 48kHz
cleanly (Terminus 1A40:0101 and 214B:7260 chips; in 0.2 the 214B clicked).
A hub that still struggles is likely to exist, though.

**If you hear constant clicks or gaps at 48kHz behind a hub, set the AHI mode
to 44100 Hz.** 44.1kHz is sent in one piece, which every hub handles more
easily. Most music is 44.1kHz anyway, so it is usually the better choice: it
also spares AHI converting the rate in software.

Directly on the Pi's port, or behind a hub running at full speed, both rates
work.


## No recording behind a high-speed hub

A sound card's microphone (recording) cannot be used while the card is behind
a high-speed hub. Playback is not affected. Recording would need a second kind
of split transfer every millisecond, which this driver does not do. It works
with the card directly on the Pi's port or behind a full-speed hub.


## Audio behind a high-speed hub uses more CPU

Behind a high-speed hub the driver has to handle eight times as many USB timing
interrupts as with the sound card directly on the port. On a Pi Zero 2 that is
a noticeable share of the machine while audio plays. It has no effect on the
sound, but other programs get less CPU in the meantime.


## Audio: clicks when the machine is very busy

### How to tell what you are hearing

Play something for a minute or two, **quit the player completely** (some
players keep the audio open while stopped, and the figures are only written
when it closes), then double-click **Dwc2Report** (or run
`Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG` in a shell).

In the file it saves, the lines starting `iso` tell you where a fault is:

| line | what it counts | whose |
|---|---|---|
| `iso gaps ... 4+:N` | frames the driver failed to send | this driver - should be 0 |
| `iso sof ... missed N` | USB frame clock ticks that never arrived | this driver - should be close to 0 |
| `iso reload ... replays:N` | audio usbaudio.class sent twice | AHI and the class, not this driver |

A `replays` count of 1 is normal: it is the very start of playback.

### Replays: AHI protecting the machine

AHI has a **CPU usage limit** (in AHI prefs, Advanced settings). When the
machine is too busy, AHI skips mixing a buffer rather than let audio take over
the system, and `usbaudio.class` then sends the previous buffer again - about
20 milliseconds you have already heard. You hear a click and a short repeat.

**What helps:**

- **Close anything that keeps the CPU busy** while you listen.
- **Match the AHI mode's frequency to what you are playing.** Playing 44.1kHz
  material through a 48kHz mode makes AHI convert every buffer in software.
- **Use a lighter player.**


## Low-speed devices are refused behind a full-speed hub on the Pi

A low-speed device (many keyboards and mice) is refused if the hub plugged
into the Pi runs at **full speed**. That includes "USB 2.0" hubs that come up
at full speed - Poseidon's device list shows each hub's speed.

This is a limitation of the Pi's USB controller: with the PHY it is built with,
it cannot reach low-speed devices while its port runs at full speed. The Linux
driver refuses the same case on this hardware. Attempting it takes the whole
USB bus down until replug, so this driver refuses it instead.

Low-speed devices work directly on the Pi's port, and anywhere below a
**high-speed** hub plugged into the Pi - even behind a second, full-speed hub
further down - because the high-speed hub's transaction translator does the
low-speed signalling. Since audio now works behind a high-speed hub too, that
is the kind of hub to use for a sound card plus keyboard and mouse.


## Devices behind a hub are checked at most every 8ms

Low- and full-speed devices behind a high-speed hub (mice, keyboards, game
pads, a sound card's buttons) need precisely timed transfers that the driver
has to wait for itself, using the CPU. Devices that asked to be checked every
1 or 2 milliseconds could take over the whole machine, so from 0.4 devices
behind a hub are checked at most every 8 milliseconds, and the time the driver
spends on them is capped.

That is the normal rate for a mouse or keyboard, and you will not notice it.
A device that genuinely needs faster polling (a 1000Hz gaming mouse, some
Bluetooth dongles or USB-serial adapters) still works, but more slowly behind a
hub. **Plug such a device straight into the Pi**, where it is not affected.


## The mouse can be less smooth behind a hub during heavy copying

With a mouse and a USB drive behind the same hub, a long file copy can make
the pointer slightly less smooth. Low- and full-speed devices behind a
high-speed hub need precisely timed transfers that the driver has to wait for,
and heavy disk traffic on the same hub makes those waits longer. Directly on
the port, or with no copy running, it is smooth.


## Audio gaps for the first window drag on some systems

On some setups (CaffeineOS, for example) dragging a window stops all programs
from running for up to about an eighth of a second. The driver notices the
first time this nearly starves the audio, and from then on keeps more audio
queued ahead (up to 128 milliseconds) for the rest of the session. So you may
hear one or two gaps on the first drags after starting the driver, and then no
more. Machines that never stall like this keep the normal, lower latency.


## Some hubs lose audio, or upset mice and keyboards

Sound cards, mice and keyboards are slower USB devices. Behind a high-speed hub
their traffic goes through a part of the hub called the transaction
translator, and some hubs' translators do not cope: the hub accepts the data
and then fails to pass it on. Audio crackles and stutters badly (sometimes
with crackling that carries on for a moment after you stop playback), or a
mouse pointer suddenly races off across the screen. The driver cannot see this
happening, so its log looks normal.

It depends on the exact hub chip, not just the maker. Two hubs with
**Genesys Logic** chips show both sides:

| hub chip (Poseidon's device list shows it) | result |
|---|---|
| Genesys 05E3:**0610**, named "USB2.1 Hub" | audio unusable, mouse goes wild; one user's machine froze when audio started with a sound card and a wireless mouse receiver on it |
| Genesys 05E3:**0608**, named "USB2.0 Hub" (e.g. a D-Link 7-port hub) | works well |

Poseidon 4.5 may warn that Genesys hubs are broken; as the table shows, that
is true of some and not of others.

Hubs that name themselves "USB2.1 Hub" are usually the USB 2 side of a **USB 3
hub**, and these seem to be the ones that struggle. **For a sound card, mouse
or keyboard, a plain USB 2.0 hub is the safest choice.** If a device misbehaves
behind a hub but works plugged straight into the Pi, try a different hub.

The freeze on the 0610 has not been reproduced on any other hub, so it is not
yet understood. If it happens to you, a different hub is the fix for now, and
a report is very welcome.

(A 7-port hub showing up in Trident as two hubs is normal: it is two 4-port
hub chips chained inside one box.)


## Devices resetting on a bus-powered hub

With a hub that takes its power from the Pi's USB port, plugging in or pulling
out a USB stick can briefly drop the power to everything else on the hub. A
device that browns out like this resets itself: a mouse stops responding until
Poseidon notices and restarts its port a few seconds later, and in a bad case a
USB stick drops out mid-transfer. The driver reports the errors correctly and
nothing is corrupted, but it is not pleasant.

**Use a powered hub (one with its own power supply) when you connect more than
one or two devices**, and especially for USB sticks.


## A device misbehaving on its first connection

Occasionally a device fails its first connection: it is not detected, it is
refused, or a fast USB stick comes up at the slow full speed (Poseidon's device
list shows the speed). Unplugging it and plugging it back in - or into another
port on the hub - fixes it. This is the device and its connection, not the
driver; the same plug-in a second time is handled normally.


## Not this driver: unplugging a USB sound card while it is playing

Pulling out a USB sound card mid-playback leaves the player unable to quit, and
the hub it was plugged into stops noticing new devices, until you reboot. This
is a lock-up between Poseidon's hub and audio classes while the card is being
removed; the USB driver keeps working (other devices on the same hub carry on).
**Quit the player before unplugging the sound card.**


## Not this driver: harsh, distorted sound with a high channel count in AHI

If AHI prefs give the USB audio mode many **Channels** (CaffeineOS defaults to
16) with Volume and Gain at 0 dB, single sounds can come out many times too
loud and clip into a harsh buzz. Set the USB mode to **1 channel** (or a few)
and lower Volume and Gain, for example to -20 dB. Music players like AmigaAMP
use their own channels and are not affected by this setting.


## Not this driver: USB sticks slower under Poseidon 6.x

Under Poseidon 6.x a FAT-formatted USB stick reads and writes at roughly half
the speed it does under Poseidon 4.5. The cause is two settings Poseidon 6.x's
`massstorage.class` gives the filesystem when it mounts a stick, and it happens
with the Emu68 xHCI driver too (there the gain from fixing it was 2.5x). It has
been reported to Poseidon's maintainer with a suggested fix and measurements.
Nothing in this driver can change it.


## Not this driver: duplicate drive numbers on eject or media change

Ejecting a drive, or swapping a card in a card reader, can bring it back with
an incremented unit number - `UCD0` and then `UCD1`. This is
`massstorage.class` failing to match the mount it already has when it
re-probes, and renaming to avoid the clash. It has been reproduced on three
different host controller drivers across two Poseidon versions and four
unrelated devices, including cases where no USB activity occurs at all, so no
host controller driver can affect it.

Turning off the **Simple SCSI** quirk in Trident's per-device settings is worth
trying for an optical drive, where eject genuinely needs a command that quirk
filters out - but it is not a reliable fix and does not address the renaming.
