Simple bash script (and systemd service) to automatically reconnect the serial connection of Klipper using REST whenever the USB cable of the printer is plugged in. This is accomplished by a custom udev rule that hooks into the USB subsystem and gets triggered whenever a certain device is found.

## Installation

1. On your Pi/Klipper Host, clone this repository - make sure you're working as `root` user:

3. Symlink (or copy) script and service:

    ```shell
    cp klipper-moonraker-usb-autoconnect /usr/local/bin/klipper_usb_autoconnect
    cp klipper-moonraker-usb-autoconnect/klipper_usb_autoconnect.service /etc/systemd/system/klipper_usb_autoconnect.service
    ```

4. Create the `udev` USB hook:

    ```shell
    editor /etc/udev/rules.d/40-klipper_usb_autoconnect.rules
    ```

    ```
    ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6001", TAG+="systemd", ENV{SYSTEMD_WANTS}="klipper_usb_autoconnect.service"
    ```

    (Please note that the hook above is for an Creality Ender-3, if you have a different printer you can use `lsusb` and `lsusb -vs 00X:00Y` to find the proper `idVendor` and `idProduct` numbers. See "[Find out idVendor / idProduct](#find-out-idvendor--idproduct)" for more more detailed guide.)

5. Activate new `udev` rules and restart service

    ```shell
    udevadm control --reload-rules
    systemctl restart systemd-udevd.service
    ```

## Troubleshooting

### Find out `idVendor` / `idProduct`

The script was made to work with an Creality Ender 3. In case you have a different printer, you might have to adjust the `udev` rules to reflect the proper information of your printers USB interface. Follow the steps below to find out the bus and device information of said port:

1. Use `lsusb` to identify your printer interface:

	```shell
	pi@octopi:~ $ lsusb
	Bus 001 Device 025: ID 0403:6001 Future Technology Devices International, Ltd FT232 USB-Serial (UART) IC
	Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
	Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. SMC9514 Hub
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	```

	I'm going to use my Ender 3 in this example again, so in this case it's the first line in the command output. Write down the bus and device information:

	- Bus: `001`
	- Device: `025`

2. Display the desired `idVendor` and `idProduct` values. Make sure to pass your bus and device information to the `-s` switch from the previous command:

	```shell
	pi@octopi:~ $ lsusb -vs 001:025 | grep "idVendor\|idProduct"
	Couldn't open device, some information will be missing
	  idVendor           0x0403 Future Technology Devices International, Ltd
	  idProduct          0x6001 FT232 USB-Serial (UART) IC
	```

	The numberes are prefixed with a `0x` to indicate that this is a hex value. You can ignore that and just note down everything that follows:

	- `idVendor`: `0403`
	- `idProduct`: `6001`

3. Now we just need to adjust (or create) the `udev` rule to reflect the new vendor and product values. Open the `/etc/udev/rules.d/40-octoprint_usb_autoconnect.rules` file with a text editor of your choice:


	Take a look at [step 4 in the installation guide](#installation).

### Run script manually, verbose mode

```shell
pi@octopi:~ $ bash -x /usr/local/bin/klipper_usb_autoconnect
{"result": "ok"
pi@octopi:~ $
```

### Run service manually

```
pi@octopi:~ $ sudo systemctl --no-block start octoprint_usb_autoconnect.service
pi@octopi:~ $
```

### Check `udev` monitor

Show all events of the USB subsystem:

```
pi@octopi:~ $ udevadm monitor --subsystem-match=usb
monitor will print the received events for:
UDEV - the event which udev sends out after rule processing
KERNEL - the kernel uevent

[plug in USB device]

KERNEL[4754.758134] add      /devices/platform/soc/3f980000.usb/usb1/1-1/1-1.3 (usb)
KERNEL[4754.766885] add      /devices/platform/soc/3f980000.usb/usb1/1-1/1-1.3/1-1.3:1.0 (usb)
UDEV  [4754.817972] add      /devices/platform/soc/3f980000.usb/usb1/1-1/1-1.3 (usb)
UDEV  [4754.821559] add      /devices/platform/soc/3f980000.usb/usb1/1-1/1-1.3/1-1.3:1.0 (usb)
^C
pi@octopi:~ $
```

## Contributing

1. Fork it
2. Create your feature branch:

```shell
git checkout -b feature/my-new-feature
```

3. Commit your changes:

```shell
git commit -am 'Add some feature'
```

4. Push to the branch:

```shell
git push origin feature/my-new-feature
```

5. Submit a pull request

## Requirements / Dependencies

* Klipper with Fluidd or Mainsail (they use Moonraker API for printer comms)
* systemd / udev compatible system

## Version

0.1

## License

[MIT](LICENSE)
