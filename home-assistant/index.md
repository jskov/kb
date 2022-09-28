# Home Assistant

Stuff related to setup of [home assistant](https://www.home-assistant.io/).

## Controller

USB controller [ZBDongle-P from Sonoff](https://sonoff.tech/product-document/diy-smart-switch-doc/zigbee-dongle-plus-cc2652p-doc/)


Cannot see [S26 Smart Plug](https://sonoff.tech/product/smart-plug/s26/)

### Access

Need membership of group `dialout` to access the USB controller:

```console
$ sudo usermod -G dialout jskov
```

Access from `/dev/ttyUSB0`.


### Prepare firmware update

Follow [instructions](https://www.zigbee2mqtt.io/guide/adapters/flashing/flashing_via_cc2538-bsl.html)


```console
$ sudo dnf install python3-intelhex python3-pyserial
```

Download [coordinator firmware](https://github.com/Koenkk/Z-Stack-firmware/raw/master/coordinator/Z-Stack_3.x.0/bin/CC1352P2_CC2652P_launchpad_coordinator_20220219.zip) (sha256sum:2679fa21ff0347acbef733d82fba4d010ae9270b256c7d09c5dc183cea860a5d)

```console
$ python3 cc2538-bsl.py -ewv -p /dev/ttyUSB0 --bootloader-sonoff-usb ~/Downloads/x/CC1352P2_CC2652P_launchpad_coordinator_20220219.hex

sonoff
Opening port /dev/ttyUSB0, baud 500000
Reading data from /home/jskov/Downloads/x/CC1352P2_CC2652P_launchpad_coordinator_20220219.hex
Your firmware looks like an Intel Hex file
Connecting to target...
CC1350 PG2.0 (7x7mm): 352KB Flash, 20KB SRAM, CCFG.BL_CONFIG at 0x00057FD8
Primary IEEE Address: 00:12:4B:00:24:C8:62:D3
    Performing mass erase
Erasing all main bank flash sectors
    Erase done
Writing 360448 bytes starting at address 0x00000000
Write 104 bytes at 0x00057F988
    Write done                                
Verifying by comparing CRC32 calculations.
    Verified (match: 0xddfc152d)
```

## Podman running home-assistant

Allow podman to access devices:

```console
$ sudo setsebool -P container_use_devices 1
```

```console
$ podman run  --group-add 18 --name homeassistant   -e TZ=Europe/Copenhagen   -v /opt/data/home-assistant:/config:Z,rw   --cap-add=CAP_NET_RAW,CAP_NET_BIND_SERVICE   -p 8123:8123   --device /dev/ttyUSB0 -it  ghcr.io/home-assistant/home-assistant:stable
```

## First setup

```console
$ mkdir /opt/data/home-assistant
```
Start container. Then visit console at [localhost:8123](http://localhost:8123)

1. Create user account
2. Add installation: Home

Add device

1. Settings, Devices and Services, Add Integration
2. Select ZHA (Zigbee Home Automation)
3. Pick USB device (which should be shown), click Submit
4. Select ZNP, click Submit
5. Click Submit with input as is.
XXX Failed to connect
