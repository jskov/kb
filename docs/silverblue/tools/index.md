# Flatpak


## Study

### Security

Would like to be able to use [Idea from flatpak](https://github.com/flathub/com.jetbrains.IntelliJ-IDEA-Community), and possibly AIs inside it.

But letting (Idea control) AI access to anything is a major yuck.
So probably keep using cop to cage AI for now.

How about the rest of the 4GB Idea installation?

Idea pak is configured with default access to `xdg-run/keyring`. This is the [D-Bus Secret Service (Keyring)](https://specifications.freedesktop.org/secret-service/latest/).
As per [spec](https://specifications.freedesktop.org/secret-service/latest/ch10.html) there is no access control.
This is why the [Portal for Secrets](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Secret.html) should be used instead.

But this would require Idea to know how to handle that.



As for the `ssh-agent` socket, allows use of ssh to other servers, signing and of course git server pushes.
The only thing I want to allow is the last of those.

So maybe issue a token to the IDE, and use https cloning? This would ergonomically annoying, since push/pull would then be problematic from (non-IDE) shells.
But probably a workable solution. Must try it out in practice...






## Listing

$ flatpak remote-ls | grep openjdk

## Configuration

Configuration of individual paks.

See user overrides

```sh
flatpak permission-show NAME
flatpak --user override --show NAME
```


### Eclipse

Allow access to SSH agent:

```sh
mkdir -p ~/eclipse-workspace ~/.m2 ~/.gradle ~/_tools
flatpak install org.eclipse.Java
flatpak --user override org.eclipse.Java --socket=ssh-auth
flatpak --user override org.eclipse.Java --persist=.ssh
flatpak --user override org.eclipse.Java --persist=.p2
flatpak --user override org.eclipse.Java --nofilesystem=host
flatpak --user override org.eclipse.Java --filesystem=~/eclipse-workspace
flatpak --user override org.eclipse.Java --filesystem=~/git
flatpak --user override org.eclipse.Java --filesystem=~/.m2
flatpak --user override org.eclipse.Java --filesystem=~/.gradle
flatpak --user override org.eclipse.Java --filesystem=~/.gitconfig
flatpak --user override org.eclipse.Java --filesystem=~/_tools:ro
flatpak --user override org.eclipse.Java --env=HOME=/var/home/jskov
```

When it fails to start, remove '.var/app/org.eclipse.Java/eclipse/configuration/org.eclipse.osgi' (or '.var/app/org.eclipse.Java').

Install plugins via Marketplace (not via suggested popups like for java 25).

Additional JDKs, download and install in ~/_tools and add them in Eclipse (use a path like '/var/home/jskov/_tools/jdk-25'):
 - Java runtimes
 - Gradle java home

### Zed

```sh
flatpak install dev.zed.Zed
flatpak --user override dev.zed.Zed --nosocket=gpg-agent
flatpak --user override dev.zed.Zed --socket=ssh-auth
flatpak --user override dev.zed.Zed --persist=.ssh
flatpak --user override dev.zed.Zed --nofilesystem=host
flatpak --user override dev.zed.Zed --filesystem=~/git
flatpak --user override dev.zed.Zed --filesystem=~/.gitconfig:ro
```

### Idea

```sh
flatpak install dev.zed.Zed
flatpak --user override dev.zed.Zed --nosocket=gpg-agent
flatpak --user override dev.zed.Zed --socket=ssh-auth
flatpak --user override dev.zed.Zed --persist=.ssh
flatpak --user override dev.zed.Zed --nofilesystem=host
flatpak --user override dev.zed.Zed --filesystem=~/git
flatpak --user override dev.zed.Zed --filesystem=~/.gitconfig:ro
```

### Vscodium

```sh
flatpak install com.vscodium.codium
flatpak --user override com.vscodium.codium --socket=ssh-auth
flatpak --user override com.vscodium.codium --persist=.ssh
flatpak --user override com.vscodium.codium --nofilesystem=host
flatpak --user override com.vscodium.codium --filesystem=~/git
flatpak --user override com.vscodium.codium --filesystem=~/.gitconfig:ro
```

### Rhythmbox

Music player

```sh
flatpak install flathub com.github.taiko2k.tauonmb
flatpak --user override org.gnome.Rhythmbox3 --filesystem=/opt/music:ro
```

### Picard

```sh
flatpak install flathub org.musicbrainz.Picard
flatpak --user override org.musicbrainz.Picard --nodevice=all
flatpak --user override org.musicbrainz.Picard --nofilesystem=home
flatpak --user override org.musicbrainz.Picard --nofilesystem=/tmp
flatpak --user override org.musicbrainz.Picard --filesystem=~/Downloads/_opus
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

### Steam

```sh
flatpak install com.valvesoftware.Steam com.valvesoftware.Steam.CompatibilityTool.Proton-GE
```
