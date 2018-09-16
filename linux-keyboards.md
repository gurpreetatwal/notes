# How keyboards work in Linux
## XKB
> handles keyboard settings and layouts

- can to be configured via XKB options in `/etc/X11/xorg.conf`, however on most modern distros this file does not exist - the file just gets generated on the fly
- keyboard settings instead should be configured in:
  - `/etc/default/keyboard`: X looks here for options to pass to XKB
  - `/usr/share/X11/xkb`: Directory which stores configuration files for XKB
    - used window managers to populate their keyboard settings pages
