# How keyboards work in Linux

<!-- toc -->

- [Terminology](#terminology)
- [XKB](#xkb)
  * [Configuration](#configuration)
    + [Geometry](#geometry)
    + [Keycodes/Symbols](#keycodessymbols)
    + [Rules](#rules)
  * [Practical Usage](#practical-usage)
    + [Remapping Keys](#remapping-keys)
      - [Custom Layout](#custom-layout)
      - [xmodmap](#xmodmap)
  * [Quirks](#quirks)

<!-- tocstop -->

Source: https://medium.com/@damko/a-simple-humble-but-comprehensive-guide-to-xkb-for-linux-6f1ad5e13450

## Terminology
- Keyboard Layout: set of keys and their arrangement
  - ex: QWERTY, colemak, etc.
- Layout Variant: adjusts a "keyboard layout" to work with a given language
  - ex: English and Italian have same character set, but Italian uses some accents
- Key Code: the unique number for each key on the keyboard
- Key Mapping: mapping from a key code to a character
  - ex: Key 38 is mapped to the character "a"
  - can be observed via `xev` program
- Modifier Keys: act as "modifiers" to keys, when pressed along with another key lead to a different behavior
  - ex: SHIFT + <letter> leads to a capital letter
  - `alt`, `alt_gr` (not on US keyboards), `escape`, `caps lock`, `insert`, `num lock`, `ctrl`, `shift`

## XKB
> handles keyboard settings and layouts

- can to be configured via XKB options in `/etc/X11/xorg.conf`, however on most modern distros this file does not exist - the file just gets generated on the fly
- keyboard settings instead should be configured in:
  - `/etc/default/keyboard`: X looks here for options to pass to XKB
  - `/usr/share/X11/xkb`: Directory which stores configuration files for XKB
    - window managers use info from here to populate their keyboard settings pages

### Configuration
> `/usr/share/X11/xkb`

*The author of this article mentions that they have not found any use for the compat and types directories*

The top level directories can be divided into 3 high levels groups:
1. files that graphically display the keyboard layout (`geometry`)
2. files to configure the key mapping and keyboard layout (`keycodes`, `symbols`)
3. files to enable the configuration (`rules`)

#### Geometry
- these are used by GUIs to show a visual representation of the keyboard
- need to be modified if using something like a Kinesis keyboard

#### Keycodes/Symbols
- `keycodes/evdev`: map a binding between a "key code" and a "key symbol"
  ```
   <TLDE> = 49;
   <BKSP> = 22;
   ```

- `symbols/us`: used to translate a "key symbol" to an actual character for `us` layouts
  - each variant in the file `xkb_symbols` block can be written from scratch or can inherit from parent layouts
   ```
   key <TLDE> { [ grave, asciitilde ] };
   key <BKSP> { [ BackSpace, BackSpace ] };
   ```
   *The second character is what is mapped if `shift` is pressed*

#### Rules
> Inform XKB about available layouts
- `base.lst`: lists layouts and their children
- `base.xml`: identical to `base.lst`
- `evdev.lst`: add more context to the information in `base.lst`
- `evdev.xml`: identical to `evdev.lst`

### Practical Usage
The `setxkbmap` utility can be used to quickly activate layouts and their variants as well as activating options.

Ex:
```bash
# activates the base us layout
setxkbmap -layout us

# activates the base us layout and the dvorak variant of the us layout
setxkbmap -layout us,us -variant ,dvorak

# print the xkb config
setxkbmap -print -verbose 10

# activate the "alt_space_toggle" option which loops through activated layouts
setxkbmap -layout us,us -variant ,dvorak -option "grp:alt_space_toggle"
```

#### Remapping Keys
There are two ways to remap keys:
- create a custom layout-variant for the US layout
- use `xmodmap` to specify mappings between key codes/key symbols and key symbols

##### Custom Layout
1. [If needed] Create/edit bindings for key codes in `keycodes/evdev`
2. Edit the `symbols/<file>` where <file> is the name of the parent layout that is going to be edited
3. Try out the changes using `setxkbmap`
4. Make changes permanent by editing `/etc/default/keyboard`

See https://github.com/damko/xkb_kinesis_advantage_dvorak_layout/compare/master...hack for inspiration

##### xmodmap
1. Find key code for key to remap using `xev -event keyboard`
2. Fine key symbol name for the key to map to (see https://cgit.freedesktop.org/xorg/proto/x11proto/tree/keysymdef.h)
3. Try out the mapping by running `xmodmap -e` in the terminal
   Ex:
   ```bash
   xmodmap -e "keycode 66 = Escape"
   ```
4. If it works correctly, write the mapping to `~/.Xmodmap` by running `xmodmap -pke` (prints the current keymap table)
   ```bash
   xmodmap -pke > ~/.Xmodmap
   ```
See https://wiki.archlinux.org/index.php/xmodmap for more information

### Quirks
Gnome and Cinnamon override XKB config, they needed to be adjusted to respect the XKB config
