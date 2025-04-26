# Flatpak

Configuration of individual paks.

See user overrides

```console
$ flatpak --user override --show NAME
```


## Eclipse

Allow access to SSH agent:

```console
$ sudo flatpak install org.eclipse.Java
$ flatpak --user override org.eclipse.Java --socket=ssh-auth
$ flatpak --user override org.eclipse.Java --persist=.ssh
$ flatpak --user override org.eclipse.Java --nofilesystem=host
$ flatpak --user override org.eclipse.Java --filesystem=~/_eclipse
$ flatpak --user override org.eclipse.Java --filesystem=~/git
$ flatpak --user override org.eclipse.Java --filesystem=~/.m2
$ flatpak --user override org.eclipse.Java --filesystem=~/.gradle
$ flatpak --user override org.eclipse.Java --filesystem=~/.gitconfig
```

## Zed

```console
$ sudo flatpak install dev.zed.Zed
$ flatpak --user override dev.zed.Zed --socket=ssh-auth
$ flatpak --user override dev.zed.Zed --persist=.ssh
$ flatpak --user override dev.zed.Zed --nofilesystem=host
$ flatpak --user override dev.zed.Zed --filesystem=~/git
$ flatpak --user override dev.zed.Zed --filesystem=~/.gitconfig:ro
```

## Vscodium

```console
$ sudo flatpak install com.vscodium.codium
$ flatpak --user override com.vscodium.codium --socket=ssh-auth
$ flatpak --user override com.vscodium.codium --persist=.ssh
$ flatpak --user override com.vscodium.codium --nofilesystem=host
$ flatpak --user override com.vscodium.codium --filesystem=~/git
$ flatpak --user override com.vscodium.codium --filesystem=~/.gitconfig:ro
```

## Rhythmbox

Music player

```console
$ flatpak install flathub com.github.taiko2k.tauonmb
$ flatpak --user override org.gnome.Rhythmbox3 --filesystem=/opt/music:ro
```
