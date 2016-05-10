# Bionico Software system setup

## Requirements

- MacOSX
- Micro sd-card usb adapter
- Download and unzip
  [raspbian](https://downloads.raspberrypi.org/raspbian_lite_latest)
- Download and install [etcher](http://www.etcher.io/)

## Preliminary steps

- Insert sd-card usb adapter in you computer with a sd-card loaded in
  it.
- Launch Etcher, click on the `Select image` button, and select the
  raspbian image that you unzipped previously.
- Etcher should auto detect the sd-card and jump the the third step,
  click on then `Burn!` button.
- Insert the sd-card in the raspberryPI, connect it to the network, and
  power it on.

## Linux configuration

- Open the Terminal.app application on your Mac.
- Connect to the raspberryPI: `ssh pi@raspberrypi.local`
  password: `raspberry` (field on screen stays blank, it a feature, keep
typing)
- Type: `sudo raspi-config`
- Select `Expand filesystem`
- Select `Advanced Options` then `Hostname` then `Ok`, then enter the
  name `bionicodevkit`
- Now goto the `Finish` button by pressing Tab twice
- Select `Yes` to reboot
