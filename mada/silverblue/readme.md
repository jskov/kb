# Silverblue configuration for mada

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
