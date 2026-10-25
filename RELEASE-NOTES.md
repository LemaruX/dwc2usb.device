dwc2usb.device 0.9 (build 1404)

New in 0.9

- Mice, keyboards and other slower devices behind a hub use much less CPU, and are checked on time. With a wireless keyboard receiver, a mouse, a sound card and a USB stick behind one hub, about a quarter more CPU is left for other programs than with 0.8.
- USB drives behind a busy hub are a little faster for small transfers.
- More tolerant of hubs that garble a reply from a mouse or keyboard: it is asked for again instead of counting towards Poseidon giving up on the device.

Everything else behaves as in 0.8. Tested with Poseidon 6.2 and 4.5.
