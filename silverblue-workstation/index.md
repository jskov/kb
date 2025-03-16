# Silverblue Workstation


## Toolbox / Distrobox

Started from https://fedoramagazine.org/alternative-way-of-saving-toolboxes-for-later-use/

But really limited - cannot create volumes, and GitHub project looks like a ghost town.
Guess I could just use Podman directly. But first have a go with distrobox:

```console
$ sudo rpm-ostree install distrobox
```

### Instances

* `backup`: for working with data backups. Mounts /opt
* `java`: for java development. Installs OpenJDK 21+latest


### Setup

Activated with

```console
$ ln -s ~/kb/silverblue-workstation/bashrc.d ~/.bashrc.d
```

And ensuring .bashrc contains:

```shell
# User specific aliases and functions
if [ -d ~/.bashrc.d ]; then
    for rc in ~/.bashrc.d/*; do
        if [ -f "$rc" ]; then
            . "$rc"
        fi
    done    
fi      
unset rc
```


