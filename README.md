# Hubs Cloud

A fork of the [Hubs Foundation](https://hubsfoundation.org/) Hubs Cloud Community Edition with changes to support installation onto a Raspberry Pi.

The intent is that this repository, and associated repositories, are temporary forks for testing. Ideally when properly validated these repositories will be presented to Hubs Foundation upstream for inclusion in their codebase as a set of pull requests. 

It is understood that these changes could potentially break a lot of behaviours so we're very much looking for people who are interested in testing this out on Raspperry Pis, arm64, and amd64 platform architectures.

# Installation (Raspberry Pi 5)

We're testing on a Raspberry Pi 5 with 8GB of RAM but this *should* work on a 4GB board and possibly even on a 2GB board variant.

## Setup a 'vanilla' Raspberry Pi OS image

Go through a standard imaging procedure such as using `rpi-imager` (link here)[https://www.raspberrypi.com/software] to install a standard **64-bit** Raspberry Pi desktop [image](https://www.raspberrypi.com/software/raspberry-pi-desktop) 

TIP: You can use `rpi-imager` to edit the image settings before writing to the Pi and you might want to put in a default WiFi SSID and password so you can easily connect, and also consider adding a public key for security if you are comfortable with these things.

