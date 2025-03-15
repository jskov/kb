# Silverblue Workstation


## Toolbox

Started from https://fedoramagazine.org/alternative-way-of-saving-toolboxes-for-later-use/

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


