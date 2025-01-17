# OctoPrint-Klipper-mjpg-Dockerfile
A Dockerfile for running OctoPrint and Klipper in a single container.

I run this on a Raspberry Pi 4 with 8G of RAM and an external SSD with my Creality CR-10S Pro.
For the Display delivered with the printer I need a forked version of klipper (https://github.com/Desuuuu/klipper)
This is used in the Dockerfile. So If you want to use plain klipper. Go to the repo, where this is forked from.

This is very much written for what I needed, so you'll likely need to hack this up for your setup. I've been using it for a little while now and it's going well.

Also included are some udev rules for reference that I use. These will need to be updated with your API key etc, however it makes connecting/disconnecting (power on/off) of the printer much less painful.

## Running the container

Once the container is built (the usual `docker build . -t okmd`), I use the following command to run it.

```
docker kill octoprint2
docker rm octoprint2
docker run --name octoprint2 -d -v /etc/localtime:/etc/localtime:ro -v /home/ubuntu/octoprint-config:/home/octoprint/.octoprint \
    --device /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_AM00OW9S-if00-port0:/dev/ttyUSB0 \
    -p 5000:5000 -p 8080:8080 -p 8081:8081 -p 8082:8082 \
    octoprint-klipper
```

Your Klipper `printer.cfg` should be kept in the OctoPrint config directory (this is where it looks for it at startup).


And run with something like:
```
docker run -d -v /etc/localtime:/etc/localtime:ro -v /home/user/Documents/octoprint-config:/home/octoprint/.octoprint \
    --device /dev/ttyUSB0:/dev/ttyUSB0 \
    -p 5000:5000 \
    ko
```

This is basically untested, but maybe a good start for someone who wants a simpler base container.

# Building the Klipper firmware

To build the firmware for the 3D printer, enter the container, go to the klipper directory, run `make menuconfig`, then build, kill klipper, upload, restart. For example:
```
docker exec -ti octoprint2 bash
cd /home/octoprint/klipper
make menuconfig
make
pkill -f klippy; pkill -f klipper
avrdude -p atmega1284p -c arduino -b 57600 -P /dev/ttyUSB0 -U out/klipper.elf.hex

# Now exit and restart the container
```

