---
name: Problem report
about: Something does not work, or works badly, with dwc2usb.device
title: ''
labels: ''
assignees: ''
---

**What happened**
What you did, what you expected, and what happened instead.

**Driver build**
Output of `Dwc2Diag DEVS:USBHardware/dwc2usb.device PROBE`:

```
paste here
```

**Your setup**
- Pi model (e.g. Pi Zero 2 W, Pi 3A+):
- Emu68 version:
- AmigaOS version:
- Poseidon version:

**USB devices and how they are connected**
Directly on the Pi's port, or through which hub? The output of `PsdDevlister`
is ideal.

```
paste here
```

**Driver log**
Run this straight after the problem happens, and attach `RAM:dwc2log.txt`
(drag it into this box):

```
Dwc2Diag DEVS:USBHardware/dwc2usb.device LOG >RAM:dwc2log.txt
```

For audio problems, quit the player completely first - the audio figures are
only written when it closes.

**Anything else**
Does it happen every time? Did it work before? Anything unusual about your
setup?
