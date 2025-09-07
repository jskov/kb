# Silverblue Workstation

## Systemd Homectl

https://discussion.fedoraproject.org/t/building-a-new-home-with-systemd-homed-on-fedora/72690

```console
$ sudo restorecon -rv /var/cache/systemd/home/
$ sudo homectl create myuser --disk-size=10G --enforce-password-policy=no
```

Does not appear to work properly yet.

## SELinux

After an update (new podman?) needed to relabel disk, because
container failed to start:

```console
$ sudo semanage fcontext -a -e /var/lib/containers /var/home/dns/.local/share/containers
$ sudo restorecon -R -v /var/home/dns/.local/share/containers
```
