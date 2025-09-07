# Silverblue configuration for mada


## Home Assistant

[Home Assistant](https://www.home-assistant.io/) installed via layering of [fedora-iot-assistant](https://github.com/jskov/fedora-iot-assistant).



## Updating

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

## Switching Release

See [package releases](https://packages.fedoraproject.org/pkgs/fedora-release/fedora-release-iot/) and specific version for [#41](https://packages.fedoraproject.org/pkgs/fedora-release/fedora-release-iot/fedora-41.html).



```console
$ rpm-ostree status
State: idle
Deployments:
* fedora-iot:fedora/stable/x86_64/iot
                  Version: 40.20240910.0 (2024-09-10T12:18:20Z)
               BaseCommit: 82cc29e1f63170edced08252f50ea014b3a4863f6b8a65c12f9aafa9aa7ad768
             GPGSignature: Valid signature by 115DF9AEF857853EE8445D0A0727707EA15B79CC
          LayeredPackages: ansible ansible-collection-ansible-posix.noarch ansible-collection-community-general.noarch bind-utils cockpit-ostree cockpit-podman cockpit-system net-tools power-profiles-daemon toolbox
            LocalPackages: assistant-shim-1.0-1.fc40.x86_64

  fedora-iot:fedora/stable/x86_64/iot
                  Version: 40.20240910.0 (2024-09-10T12:18:20Z)
               BaseCommit: 82cc29e1f63170edced08252f50ea014b3a4863f6b8a65c12f9aafa9aa7ad768
             GPGSignature: Valid signature by 115DF9AEF857853EE8445D0A0727707EA15B79CC
          LayeredPackages: ansible ansible-collection-ansible-posix.noarch ansible-collection-community-general.noarch bind-utils cockpit-ostree cockpit-podman cockpit-system net-tools power-profiles-daemon toolbox

  fedora-iot:fedora/stable/x86_64/iot
                  Version: 39.20240312.0 (2024-03-12T17:45:27Z)
               BaseCommit: 9c62419d6f8a8947edf6bb3f1a69c31235ac43c43bcd20f2141e58f050068dd2
             GPGSignature: Valid signature by E8F23996F23218640CB44CBE75CF5AC418B8E74C
          LayeredPackages: ansible ansible-collection-ansible-posix.noarch ansible-collection-community-general.noarch bind-utils cockpit-ostree cockpit-podman cockpit-system net-tools power-profiles-daemon toolbox
                   Pinned: yes
```

```console
$ ostree remote refs fedora-iot
fedora-iot:fedora/devel/aarch64/iot
fedora-iot:fedora/devel/x86_64/iot
fedora-iot:fedora/rawhide/aarch64/iot
fedora-iot:fedora/rawhide/x86_64/iot
fedora-iot:fedora/stable/aarch64/iot
fedora-iot:fedora/stable/x86_64/iot
```

Rawhide is Fedora #42 at this point. So try devel:

```console
$ sudo rpm-ostree rebase fedora-iot:fedora/devel/x86_64/iot
...
$ sudo systemctl reboot
```

```console
$ cat /etc/os-release
NAME="Fedora Linux"
VERSION="41.20241017.0 (IoT Edition Prerelease)"
ID=fedora
VERSION_ID=41
VERSION_CODENAME=""
PLATFORM_ID="platform:f41"
PRETTY_NAME="Fedora Linux 41.20241017.0 (IoT Edition Prerelease)"
ANSI_COLOR="0;38;2;60;110;180"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:41"
HOME_URL="https://fedoraproject.org/"
DOCUMENTATION_URL="https://docs.fedoraproject.org/en-US/fedora/f41/system-administrators-guide/"
SUPPORT_URL="https://ask.fedoraproject.org/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Fedora"
REDHAT_BUGZILLA_PRODUCT_VERSION=41
REDHAT_SUPPORT_PRODUCT="Fedora"
REDHAT_SUPPORT_PRODUCT_VERSION=41
SUPPORT_END=2025-05-13
VARIANT="IoT Edition"
VARIANT_ID=iot
OSTREE_VERSION='41.20241017.0'
```

Using RPM 4.20, so can continue with layered RPMs.

## Ansible Bootstrap

Ansible and toolbox needs to be installed layered:

```console
$ sudo rpm-ostree install ansible ansible-collection-ansible-posix.noarch ansible-collection-community-general.noarch toolbox
$ sudo systemctl reboot
```

## Running Ansible scripts from KB

Use toolbox to clone KB:

```console
$ toolbox create
$ toolbox enter
$ git clone https://github.com/jskov/kb.git
$ exit
```

In the toolbox, git can be used for diff'ing local tweaks.
There is no intention of making pushes from mada.

### Extra layers

Script to add layers to SilverBlue. Not much point, since bootstrap is required anyway.

But shows how.

```console
$ LANG=C.UTF-8 sudo ansible-playbook kb/mada/silverblue/add-layers.yaml
```
