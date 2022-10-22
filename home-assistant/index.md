# Home Assistant

Stuff related to setup of [home assistant](https://www.home-assistant.io/) (or HASS among friends).



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

### Switch to Podman Compose

Need to tweak the image before it starts. Easiest to do with compose.

```console
sudo rpm-ostree install podman-compose
```

Copy './hass-compose.yaml' and './Containerfile' to '/home/assistant/' on casaskov.

And use new service:


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

ExecStart=/bin/bash -c "exec /usr/bin/podman-compose -f /home/assistant/hass-compose.yaml up"
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


## Configuration

### Ikea Smart Plug

TODO

Ikea Plug works as is.

The switch E1743 not:

https://www.zigbee2mqtt.io/devices/E1743.html


When binding switch to plug, the plug does not react to controller.


### Eloverblik


Downloaded v0.4.0 from https://github.com/JonasPed/homeassistant-eloverblik

Created `/opt/data/services/assistant/custom_components` folder and placed 'custom_components/eloverblik' in it. Restarted the service.

Had to make the folders and files owned by the 'assistant' user.

Add 'eloverblik' in console's Intergrations.

Configuration fails, needs 'pyeloverblik' (changed to use Podman compose to allow its installation).

Log onto https://eloverblik.dk.

Click user, Datadeling. Create new token (no to CPR, name elmaaler)
Enter new token and maalernr.

#### Data

Manuals https://energinet.dk/Energidata/DataHub/Eloverblik
API https://api.eloverblik.dk/CustomerApi/index.html

Have to repeat calls. One of the servers return 503 all the time.

```console
$ curl -v -H "authorization: Bearer eyJhbGci" https://api.eloverblik.dk/customerapi/api/token
J0b2tlblR5


curl -H 'accept: application/json' -H 'content-type: application/json' -d '{"meteringPoints": { "meteringPoint": [ "x" ] } }' -v -H "authorization: Bearer J0b2tlblR5cGUi" https://api.eloverblik.dk/customerapi/api/MeterData/GetTimeSeries/2022-10-14/2022-10-22/Hour


{"result":[{"MyEnergyData_MarketDocument":{"mRID":"80","createdDateTime":"2022-10-22T01:24:32Z","sender_MarketParticipant.name":"","sender_MarketParticipant.mRID":{"codingScheme":null,"name":null},"period.timeInterval":{"start":"2022-10-13T22:00:00Z","end":"2022-10-20T22:00:00Z"},"TimeSeries":[{"mRID":"x","businessType":"A04","curveType":"A01","measurement_Unit.name":"KWH","MarketEvaluationPoint":{"mRID":{"codingScheme":"A10","name":"x"}},"Period":[{"resolution":"PT1H","timeInterval":{"start":"2022-10-13T22:00:00Z","end":"2022-10-14T22:00:00Z"},"Point":[{"position":"1","out_Quantity.quantity":"0.188","out_Quantity.quality":"A04"},{"position":"2","out_Quantity.quantity":"0.216","out_Quantity.quality":"A04"},{"position":"3","out_Quantity.quantity":"0.152","out_Quantity.quality":"A04"},{"position":"4","out_Quantity.quantity":"0.132","out_Quantity.quality":"A04"},{"position":"5","out_Quantity.quantity":"0.162","out_Quantity.quality":"A04"},{"position":"6","out_Quantity.quantity":"0.685","out_Quantity.quality":"A04"},{"position":"7","out_Quantity.quantity":"1.087","out_Quantity.quality":"A04"},{"position":"8","out_Quantity.quantity":"0.287","out_Quantity.quality":"A04"},{"position":"9","out_Quantity.quantity":"0.229","out_Quantity.quality":"A04"},{"position":"10","out_Quantity.quantity":"0.192","out_Quantity.quality":"A04"},{"position":"11","out_Quantity.quantity":"0.192","out_Quantity.quality":"A04"},{"position":"12","out_Quantity.quantity":"0.24","out_Quantity.quality":"A04"},{"position":"13","out_Quantity.quantity":"0.325","out_Quantity.quality":"A04"},{"position":"14","out_Quantity.quantity":"0.203","out_Quantity.quality":"A04"},{"position":"15","out_Quantity.quantity":"0.267","out_Quantity.quality":"A04"},{"position":"16","out_Quantity.quantity":"0.339","out_Quantity.quality":"A04"},{"position":"17","out_Quantity.quantity":"0.336","out_Quantity.quality":"A04"},{"position":"18","out_Quantity.quantity":"0.422","out_Quantity.quality":"A04"},{"position":"19","out_Quantity.quantity":"0.296","out_Quantity.quality":"A04"},{"position":"20","out_Quantity.quantity":"0.286","out_Quantity.quality":"A04"},{"position":"21","out_Quantity.quantity":"0.413","out_Quantity.quality":"A04"},{"position":"22","out_Quantity.quantity":"0.42","out_Quantity.quality":"A04"},{"position":"23","out_Quantity.quantity":"0.424","out_Quantity.quality":"A04"},{"position":"24","out_Quantity.quantity":"0.353","out_Quantity.quality":"A04"}]},{"resolution":"PT1H","timeInterval":{"start":"2022-10-14T22:00:00Z","end":"2022-10-15T
```

So access clearly works...


But plugin fails with:

```text
Oct 22 13:42:56 casaskov assistant[8067]: 2022-10-22 13:42:56.543 WARNING (SyncWorker_7) [custom_components.eloverblik] Exception: HTTPSConnectionPool(host='api.eloverblik.dk', port=443): Max retries exceeded with url: /CustomerApi//api/MeterData/GetTimeSeries/2022-10-14/2022-10-22/Hour (Caused by ResponseError('too many 400 error responses'))
Oct 22 13:42:56 casaskov assistant[8067]: 2022-10-22 13:42:56.694 WARNING (SyncWorker_0) [custom_components.eloverblik] Error from eloverblik when getting tariff data: 400 - "#20013: No meteringpoints in request conforms to valid meteringpoint format."
Oct 22 14:43:36 casaskov assistant[8067]: 2022-10-22 14:43:36.068 WARNING (MainThread) [homeassistant.helpers.entity] Update of sensor.eloverblik_energy_total is taking over 10 seconds
Oct 22 14:43:56 casaskov assistant[8067]: 2022-10-22 14:43:56.066 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:44:26 casaskov assistant[8067]: 2022-10-22 14:44:26.068 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:44:56 casaskov assistant[8067]: 2022-10-22 14:44:56.070 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:45:26 casaskov assistant[8067]: 2022-10-22 14:45:26.072 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:45:56 casaskov assistant[8067]: 2022-10-22 14:45:56.073 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:46:26 casaskov assistant[8067]: 2022-10-22 14:46:26.074 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:46:56 casaskov assistant[8067]: 2022-10-22 14:46:56.075 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:47:26 casaskov assistant[8067]: 2022-10-22 14:47:26.076 WARNING (MainThread) [homeassistant.components.sensor] Updating eloverblik sensor took longer than the scheduled update interval 0:00:30
Oct 22 14:47:26 casaskov assistant[8067]: 2022-10-22 14:47:26.612 WARNING (SyncWorker_0) [custom_components.eloverblik] Exception: HTTPSConnectionPool(host='api.eloverblik.dk', port=443): Max retries exceeded with url: /CustomerApi//api/MeterData/GetTimeSeries/2022-10-14/2022-10-22/Hour (Caused by ResponseError('too many 400 error responses'))

```


