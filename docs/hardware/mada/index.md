The box running mada.dk is a headless box with [Fedora Silverblue IoT](https://docs.fedoraproject.org/en-US/iot/).

## Layered Packages

Missing:

 * Pihole
 * music
 * photos
 * booklore

### distrobox

Prefer this to toolbox since it can mount volumes.

```console
$ sudo rpm-ostree install distrobox
$ sudo systemctl reboot
```

To allow use of *rsync* to sync media:

```console
$ distrobox create --name fedora --image registry.fedoraproject.org/fedora-toolbox:43
$ distrobox enter fedora -- rsync --help
```

Now this is possible: `$ rsync -av --rsync-path="distrobox enter fedora -- rsync" /opt/music/ mada:/home/jskov/music/`.

## Operation

### Updating

Documentation: https://docs.fedoraproject.org/en-US/iot/applying-updates-UG/

```console
# See if update is available
$ sudo rpm-ostree upgrade --check

# Update
$ sudo rpm-ostree upgrade
$ sudo systemctl reboot

# Rollback
$ sudo rpm-ostree rollback --reboot
```

### Switching Release

*TODO*: deleted old obsoleted notes, since remote release channels have new names now.

See [package releases](https://packages.fedoraproject.org/pkgs/fedora-release/fedora-release-iot/) and specific version for [#43](https://packages.fedoraproject.org/pkgs/fedora-release/fedora-release-iot/fedora-43.html).

```console
$ rpm-ostree status
$ ostree remote refs fedora
$ sudo rpm-ostree rebase...
...
$ sudo systemctl reboot
```

### Firewall debugging

```console
# Enable
$ sudo firewall-cmd --set-log-denied=all
$ sudo firewall-cmd --get-log-denied

# Monitor
$ journalctl -x -e -f

# Disable
$ sudo firewall-cmd --set-log-denied=off
```

## Installation

Installed [Fedora 43/IoT](https://docs.fedoraproject.org/en-US/iot/) with a fresh, default, *btrfs* disk layout.

## Obsoleted

### Power

On previous versions I layered `power-profiles-daemon`, which has later been replaced by `tuned`.

After layering `tuned` the *powersave* profile does not appear to give (idle) advantage over the *balanced* (default) profile.

Which again does not improve on the clean (idle) IoT state (which consumes 6.5 Watts).

So leave it out.

### Home Assistant

Not presently have use for the Assistant.

[Home Assistant](https://www.home-assistant.io/) installed via layering of [fedora-iot-assistant](https://github.com/jskov/fedora-iot-assistant).
