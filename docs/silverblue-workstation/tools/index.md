# Flatpak

Configuration of individual paks.

See user overrides

```console
$ flatpak permission-show NAME
$ flatpak --user override --show NAME
```


## Eclipse

Allow access to SSH agent:

```console
$ flatpak install org.eclipse.Java
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
$ flatpak install dev.zed.Zed
$ flatpak --user override dev.zed.Zed --nosocket=gpg-agent
$ flatpak --user override dev.zed.Zed --socket=ssh-auth
$ flatpak --user override dev.zed.Zed --persist=.ssh
$ flatpak --user override dev.zed.Zed --nofilesystem=host
$ flatpak --user override dev.zed.Zed --filesystem=~/git
$ flatpak --user override dev.zed.Zed --filesystem=~/.gitconfig:ro
```

## Vscodium

```console
$ flatpak install com.vscodium.codium
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

## Picard

```console
$ flatpak install flathub org.musicbrainz.Picard
$ flatpak --user override org.musicbrainz.Picard --nodevice=all
$ flatpak --user override org.musicbrainz.Picard --nofilesystem=home
$ flatpak --user override org.musicbrainz.Picard --nofilesystem=/tmp
$ flatpak --user override org.musicbrainz.Picard --filesystem=~/Downloads/_opus
```

And set cleanup configuration

Use `MusicBrainz Picard` to cleanup file naming and annotations on tracks.

File Naming Script Editor should contain (Preset 3):

```text
$if2(%albumartist%,%artist%)/
$if(%albumartist%,%album%/,)
$if($gt(%totaldiscs%,1),%discnumber%-,)
$if($and(%albumartist%,%tracknumber%),$num(%tracknumber%,2) ,)
$if(%_multiartist%,%artist% - ,)
%title%
```

In `Options` select:
 Y: Rename Files
 N: Move Files
 Y: Save Tags

## Steam

```console
$ flatpak install com.valvesoftware.Steam com.valvesoftware.Steam.CompatibilityTool.Proton-GE
```
