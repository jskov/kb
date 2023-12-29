# Home Assistant

Stuff related to setup of [home assistant](https://www.home-assistant.io/) (or HASS among friends).

## Mada setup

Installed with Ansible:

```console
$ LANG=C.UTF-8 sudo ansible-playbook kb/mada/assistant/assistant-playbook.yaml
```

## Controller

USB controller [ZBDongle-P from Sonoff](https://sonoff.tech/product-document/diy-smart-switch-doc/zigbee-dongle-plus-cc2652p-doc/)

Mounted from USB2 port with a long cable, removed from computer.

### Access

Need membership of group `dialout` to access the USB controller:

```console
$ sudo gpasswd -a jskov dialout
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

## First setup

```console
$ mkdir /opt/data/home-assistant
```
Start container. Then visit console at [mada:8123](http://mada:8123)

1. Create user account
2. Add installation: Home  
. Europe/Copenhagen  
. Metric  
. DKK
3. Allow Basic analytics
4. Click Finish (the other option hangs the system)

Add device (if necessary)

1. Settings, Devices and Services, Add Integration
2. Select ZHA (Zigbee Home Automation)
3. Pick USB device (which should be shown), click Submit
4. Select ZNP, click Submit
5. Click Submit with input as is.


Configure (shows Discovered Sonoff Zigbee 3.0 USB Dongle Plus)


## Configuration

### Ikea Smart Plug

Add devices under Zigbee Home Automation.

First bind the switch and the plug with 10 second reset on switch close to the plug.

Ikea Plug works as is. Reset pin to register - click + hold for 4 seconds, then start scanning for devices.
Configure first, then take out of wall while configuring I/O switch


The switch E1743, register with 4 x fast(ish) clicks on its button to register (maybe 1/0 first)

Add events on the switch to make it control the plug.

Select the device, click on Automations +.

When binding switch to plug, the plug does not react to controller.


#### Data

Manuals https://energinet.dk/Energidata/DataHub/Eloverblik
API https://api.eloverblik.dk/CustomerApi/index.html

