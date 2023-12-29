# Silverblue configuration for mada

## Bootstrap

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

Bur shows how.

```console
$ LANG=C.UTF-8 sudo ansible-playbook kb/mada/silverblue/add-layers.yaml
```
