# Logitech Media Server

Stuff related to setup of [LMS](https://wiki.slimdevices.com/index.php/Logitech_Media_Server.html).

## Mada setup

Installed with Ansible:

```console
$ LANG=C.UTF-8 sudo ansible-playbook kb/mada/lms/lms-playbook.yaml
```

## Configuration

Configuration on mada:9000

### Initial Setup

* Skip mysqueezebox.com login
* Select Music Folder: /music
* Select Playlist Folder: /playlist

XXX: Try reproduce locally with Rhymebox client before playing firewall
XXX: Need to have persistence for configuration
XXX: Need to enable DLNA?


XXXXX TODO

 * Container expose broadcast ports
 * play with enclosure

#### Test service

Use [DLNA client in rhymebox](https://askubuntu.com/questions/552929/dlna-client-for-music) to test service.

*Local*

```console
$ podman run --name lms --rm \
  -p 3483:3483 \
  -p 3483:3483/udp \
  -p 9000:9000 \
  -v /opt/data/lms/squeezeboxserver:/var/lib/squeezeboxserver:rw,Z \
  -v /opt/data/lms/music:/music:ro,Z \
  -v /opt/data/lms/playlist:/playlist:rw,Z \
  -it lms
```






## Service interaction

```console
$ sudo su - lms -c "podman ps"

$ sudo systemctl --user -M lms@ status lms

$ sudo su - lms -c "journalctl --user -u lms -f"
```
