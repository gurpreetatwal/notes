# Mouse & Touchpad Info

Tradationally, X11 has used the synaptics driver to provide support for touchpads. Wayland uses libinput. However, these two are not explictly tied together, it is possible to use libinput with X11 and vice versa. Ubuntu 18.04 enables X11 by default and ships the `xserver-xorg-input-libinput` package (build of the upstream `xf86-input-libinput` package) to provide support for libinput in X11. This means that the tradational configuartion tools like `synclient` do not work. Instead `xinput` must be used.


In order to be able to query libinput and supported properties for each devices, the `libinput-tools` package must be installed. This provides the `libinput`, `libinput-debug-events` and `libinput-list-devices` binaries.


Use `xinput` to list the devices on the system, if multiple trackpads show up, one is touch screen, one is trackpad and other is legacy synaptics.


In order test settings using `xinput`, use `xinput list-props` to print out all the properties supported by the device, then get the option configuartion number (its in parens) and do `xinput set-prop <device-id> <option-number> <value>`

sav
