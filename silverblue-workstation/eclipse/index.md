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
