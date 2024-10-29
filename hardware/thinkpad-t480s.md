# Hardware ThinkPad T480s

See [arch wiki](https://wiki.archlinux.org/title/Lenovo_ThinkPad_T480s)


## Fingerprint Reader

https://www.reddit.com/r/Fedora/comments/s2l6i4/how_to_enabling_the_fingerprint_reader_on_the/
https://www.reddit.com/r/Fedora/comments/oik8sq/comment/h4xvrqv/




## Card Reader

Disabled since it is hardly ever used.

Comment out the below and reboot if is becomes necessary.

```console
$ cat /etc/udev/rules.d/ignore-cardreader.rules
# Disable SD cardreader
SUBSYSTEM=="usb", ACTION=="add", ATTR{idVendor}=="0bda", ATTR{idProduct}=="0316", ATTR{authorized}="0"

$ cat /etc/udev/rules.d/ignore-smartcardreader.rules
# Disable SmartCard reader
SUBSYSTEM=="usb", ACTION=="add", ATTR{idVendor}=="058f", ATTR{idProduct}=="9540", ATTR{authorized}="0"
```

From https://forums.linuxmint.com/viewtopic.php?t=309559

