# Home Assistant

Stuff related to setup of [home assistant](https://www.home-assistant.io/).

## Controller

USB controller [ZBDongle-P from Sonoff](https://sonoff.tech/product-document/diy-smart-switch-doc/zigbee-dongle-plus-cc2652p-doc/)


Cannot see [S26R2 Smart Plug](https://sonoff.tech/product/s26r2/)


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

## Podman running home-assistant

Allow podman to access devices:

```console
$ sudo setsebool -P container_use_devices 1
```

```console
$ podman run  --group-add 18 --name homeassistant   -e TZ=Europe/Copenhagen   -v /opt/data/home-assistant:/config:Z,rw   --cap-add=CAP_NET_RAW,CAP_NET_BIND_SERVICE   -p 8123:8123   --device /dev/ttyUSB0 -it  ghcr.io/home-assistant/home-assistant:stable
```

Stop it with:

```console
$ podman kill --signal TERM homeassistant
```

Need to investigate access problems to the ttyUSB. For now on the host:

```console
$ sudo chmod o+rw /dev/ttyUSB0
```


## First setup

```console
$ mkdir /opt/data/home-assistant
```
Start container. Then visit console at [localhost:8123](http://localhost:8123)

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

1. 



XXX Failed to connect

>investigate podman rootless access to tty:

    https://github.com/containers/podman/issues/4477
    https://github.com/containers/podman/issues/10166


## Reading Temperatures

[Aqara Temperature and Humidity Sensor](https://www.aqara.com/us/temperature_humidity_sensor.html) can provide data from living room.

## Reading Meter

https://robotnet.dk/2020/eloverblik-i-home-assistant.html

Elpriser https://community.home-assistant.io/t/updated-api-on-energidataservices-dk/438430



## Podman on Mada

In cockpit, add new user: assistant (disallow password authentication)

And allow this user to run service after a reboot:

```console
$ sudo loginctl enable-linger assistant
```

### Setup permisisons for controller

Cannot get Podman to allow access to `dialout` group, event though a member of it:

```console
$ ls -l /dev/ttyUSB0 
crw-rw-r--. 1 root dialout 188, 0 Oct  8 14:16 /dev/ttyUSB0
```

So make the /dev/ttyUSB0 be read/write for all users.

```console
$ rpm-ostree install usbutils
$ lsusb
...
Bus 002 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge

$ udevadm info -q all -n /dev/ttyUSB0
...
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=7034455
E: ID_BUS=usb
E: ID_VENDOR_ID=10c4
E: ID_MODEL_ID=ea60

```

Create udev rule:

```text
#/etc/udev/rules.d/99-sonoff-controller.rules
SUBSYSTEM=="tty", ENV{ID_VENDOR_ID}=="10c4", ENV{ID_MODEL_ID}=="ea60", MODE="0666"
```

Then reload rules:

```console
$ sudo udevadm control --reload
```

Remove and insert device

```console
$ ls -l /dev/ttyUSB0 
crw-rw-rw-. 1 root dialout 188, 0 Oct  8 14:16 /dev/ttyUSB0
```

### Firewall setup

Add firewall rule for TCP port 8123.

```console
$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s25
  sources: 
  services: dhcpv6-client mdns ssh
  ports: 8080/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
        port=53:proto=udp:toport=8053:toaddr=
  source-ports: 
  icmp-blocks: 
  rich rules: 

$ sudo firewall-cmd --add-port=8123/tcp
$ sudo firewall-cmd --add-port=8123/tcp --permanent

$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s25
  sources: 
  services: dhcpv6-client mdns ssh
  ports: 8080/tcp 8123/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
        port=53:proto=udp:toport=8053:toaddr=
  source-ports: 
  icmp-blocks: 
  rich rules: 
```



### SElinux problems

Allow podman to access devices:

```console
$ sudo setsebool -P container_use_devices 1
```

Try to run the container manually first:

```console
$ /usr/bin/podman run --log-driver=journald --log-opt tag=assistant --replace --name assistant -i -e TZ=Europe/Copenhagen -v /home/jskov/test:/config:Z,rw --cap-add=CAP_NET_RAW,CAP_NET_BIND_SERVICE -p 8123:8123 --device /dev/ttyUSB0 -t ghcr.io/home-assistant/home-assistant@sha256:544cbac872d2d97ad081d39a432cc9ce216bd4bbc372b4c3f320e4600db17eb9
Error relocating /lib/ld-musl-x86_64.so.1: RELRO protection failed: Permission denied
Error relocating /init: RELRO protection failed: Permission denied
```

```console
$ journalctl -b 0 | grep AVC
Oct 08 14:36:00 casaskov audit[3322]: AVC avc:  denied  { read } for  pid=3322 comm="init" path="/lib/ld-musl-x86_64.so.1" dev="dm-1" ino=6292924 scontext=system_u:system_r:container_t:s0:c695,c846 tcontext=unconfined_u:object_r:data_home_t:s0 tclass=file permissive=0
Oct 08 14:36:00 casaskov audit[3322]: AVC avc:  denied  { read } for  pid=3322 comm="init" path="/bin/busybox" dev="dm-1" ino=6292755 scontext=system_u:system_r:container_t:s0:c695,c846 tcontext=unconfined_u:object_r:data_home_t:s0 tclass=file permissive=0
```

Search gave: https://github.com/containers/podman/issues/3234

This fixed it:

```console
$ podman system reset
```

Seems to be working and is ready for onboarding.

### Setup service

```text
# /etc/systemd/system/assistant.service
[Unit]
Description=assistant
Wants=network.target
After=network-online.target

[Service]
User=assistant
Group=assistant
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=on-failure

# Version 2022.10.1
ExecStart=/bin/bash -c "exec /usr/bin/podman run --log-driver=journald --log-opt tag=assistant --replace --name assistant -i -e TZ=Europe/Copenhagen -v /opt/data/services/assistant:/config:Z,rw --cap-add=CAP_NET_RAW,CAP_NET_BIND_SERVICE -p 8123:8123 --device /dev/ttyUSB0 -t ghcr.io/home-assistant/home-assistant@sha256:544cbac872d2d97ad081d39a432cc9ce216bd4bbc372b4c3f320e4600db17eb9"
ExecStop=/usr/bin/podman stop --ignore assistant
ExecStopPost=/usr/bin/podman rm --ignore -f assistant

[Install]
WantedBy=multi-user.target default.target
```

```console
$ sudo systemctl daemon-reload

$ sudo systemctl start assistant

$ journalctl -f -t assistant --since today

$ sudo systemctl enable assistant

```

