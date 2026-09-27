# Known issues

Things that are understood, reproducible, and either cannot be fixed in this
driver or have not been fixed yet. If you hit something that is not listed
here, it is worth reporting.


## Audio: clicks or brief repeats

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
| `iso sof ... missed N` | USB frame clock ticks that never arrived | this driver - should be 0 |
| `iso reload ... replays:N` | audio usbaudio.class sent twice | the class, not this driver |

### The known cause: usbaudio.class replaying audio

`usbaudio.class`, the Poseidon class that feeds the driver its audio, keeps two
buffers. When one runs out it asks AHI to mix the next one and then, in the same
breath, reads the variable that says which buffer to play from - a variable
only updated once the mixing has finished. If the mixing has not finished, the
class hands back the buffer it has just played and replays all of it, about 19
milliseconds you have already heard. You hear a click and a short repeat. The
class never reports this, which is why it is invisible to every "is the audio
starving?" check - hence the `replays` counter.

**How common it is depends on how busy the machine is.** On a tester's machine
it did not happen once, in any run of either player. It becomes more common
when other programs keep the CPU busy, because that delays AHI's mixing. It
is the same in every Poseidon version.

**What helps if you do see replays climbing:**

- **Match the AHI mode's sample rate to what you are playing.** Playing 44.1kHz
  material through a 48kHz mode makes AHI resample every buffer in software on
  the 68k, for nothing.
- **Close other programs that keep the CPU busy** while you listen.
- **Use a lighter player.**

**Why this driver does not paper over it.** It could detect a replay and send
silence instead, turning the click into a soft dropout. It deliberately does
not: a host controller driver's job is to carry the class's audio faithfully,
no other USB driver on this stack alters audio, and it would still leave 19
milliseconds of repeated or missing sound. The fix belongs in the class.


## Isochronous is refused behind a high-speed hub

Audio will not start if the sound card is behind a **high-speed** USB 2.0 hub.
It is accepted directly on the port, and also behind a hub that comes up at
**full speed** - Poseidon's device list shows which, and many cheap "USB 2.0"
hubs report Full.

A high-speed hub puts a transaction translator in the path, which requires
split isochronous transfers. Those are not implemented. On a one-port machine
this means audio and other devices cannot currently share a high-speed hub.


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
low-speed signalling.

Note that a full-speed hub is also the kind that lets USB audio work behind
it (see above), so a sound card and a low-speed mouse cannot currently share
one.


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
