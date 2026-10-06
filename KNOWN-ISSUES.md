# Known issues

Things that are understood, reproducible, and either cannot be fixed in this
driver or have not been fixed yet. If you hit something that is not listed
here, it is worth reporting.


## 48kHz audio behind a high-speed hub depends on the hub

At 48kHz each USB audio packet is too big to cross a high-speed hub's
transaction translator in one piece, so it is sent in two parts that the hub
has to join back together. Some hubs cannot do this and drop the packet - you
hear constant clicks or gaps. A hub with a Terminus 1A40:0101 chip handles it;
one with a 214B:7260 chip does not.

**44.1kHz is sent in one piece and works through every hub tested.** Most
music is 44.1kHz anyway, so setting the AHI mode to 44100 Hz is both the fix
and the better choice: it also spares AHI converting the rate in software.

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
when it closes), then run:

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG
```

The lines starting `iso` tell you where a fault is:

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
