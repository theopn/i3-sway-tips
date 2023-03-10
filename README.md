# A Guide for Configuring i3/Sway WM

## App Launcher

### i3

`Dmenu` is the default launcher that comes as a weak dependency of `i3` by many package managers.
`Rofi` is a popular launcher that can be themed and take custom input to be used as not only an app launcher but also as a window switcher, power menu, music control, etc. `rofi -show run` lists all the executable files on the computers, and `rofi -modi drun -show drun` shows only the executables with `.desktop` files.

### Sway

> This tip applies for i3 or other WM as well

You can make a custom launcher using `fzf`. (Inspired by [A Guide to Switching From i3 to Sway](https://luxagraf.net/src/guide-to-switching-i3-to-sway)). This is popular among Sway users who don't want to use `rofi` replacements like `wofi`.

```
# For i3, replace "swaymsg" with "i3msg"
bindsym $mod+d exec <terminal-of-your-choice> --class 'launcher' --command bash -c 'compgen -c | sort -u | fzf | xargs -r swaymsg -t command exec'
# For i3, use "class" instead of "app_id"
for_window [app_id="^launcher$"] floating enable, sticky enable, resize set 30 ppt 60 ppt, border pixel 10, move center
```

[sway-launcher-desktop](https://github.com/Biont/sway-launcher-desktop) is an extension of the command above that supports highlighting desktop apps and history features.

## Battery Management

If you are installing i3/Sway along with other DEs such as Gnome or KDE, a chance is that it installed `powerprofilesctl` that integrates with DE UI and lets you choose among three power profiles. Type `powerprofilesctl list` to view the current profile setting and available options and `powerprofilesctl set` to choose a profile. Although its `power-saver` option throttles like crazy.

Another solution is `tlp`. `tlp` works great out of the box, so start with `sudo tlp start`. Follow the instruction to disable `powerprofilesctl` and how to add to `systemctl` daemon. Execute `sudo tlp-stat` to see various information, including whether the `tlp` is enabled (the first string `TLP_ENABLE="1"`).

`powertop` can be installed to give more information about the battery, though I don't know how to interpret the data.

## Backlight Control

I personally use `brightnessctl`. Keybindings are

```
bindsym XF86MonBrightnessUp exec --no-startup-id brightnessctl set +10%
bindsym XF86MonBrightnessDown exec --no-startup-id brightnessctl set 10%-
```

`light`, `brightlight`, `xbacklight`, and other many programs control the backlight.

## Bluetooth

Just install `blueman` and type `blueman-manager`.

## Clipboard Management

### i3

Nothing much to do. Just install your favorite clipboard manager, whether it's `clipit` or `copyq` and enjoy.

### Sway

More complicated. You probably want to install `wl-clipboard` just to get the clipboard working. To save a clipboard history, install `clipman` (not XFCE plugin version, one for Wayland). You can start storing clipboard history from `wl-clipboard` by executing `wl-paste -t text --watch clipman store`. To view the clipboard history, you need to send the clipboard history to an external tool. You can do this with `wofi`, with `clipman pick -t wofi`, or my preferred way, utilizing `fzf` and opening it up as a floating window by utilizing the command below.

```
bindsym <your-key-binding> <choice-of-a-terminal-emulator> --class=clipboard clipman pick --print0 --tool=CUSTOM --tool-args="fzf --prompt 'pick > ' --bind 'tab:up' --cycle --read0"
for_window [app_id="^clipboard$"] floating enable, sticky enable, resize set 30 ppt 60 ppt, border pixel 10, move position 1300px 50px

```

## Compositor

### i3

X11 server directly draws a window to the display buffer. This is fine for most of the time, but if you're experiencing screen tearing or want to enable transparency/blur, you need to install a compositor. `picom` is a great compositor that works out of the box.

### Sway

In Wayland, the compositor doubles as a window manager, meaning Sway is Wayland "compositor" that manages the window as well. What does that mean? That means you do not need a separate compositor for transparency to work. Although I found Sway compositing to be limited, transparency for certain applications (Emacs) and rounded corners, blurs, etc are yet to come.

## Display Scaling

### i3

Make `.Xresources` in your home (`~`) directory and append `Xft.dpi: <DPI-value>`. Most applications will follow the value specified in the DPI.

### Sway

I don't know.

## External Monitors

### i3

`xrandr`, which should be installed as a dependency to X11 server, gives you the list of displays. `xrandr --output HDMI-2 --auto --right-of eDP-1` projects the screen to HDMI-2 with the resolution set to auto. You can then execute `i3 move workspace to output right` to move the current workspace to the externam monitor. You can use `--same-as` flag to mirror the display. `xrandr --output HDMI-2 --off` will stop the projection.

### Sway

`swaymsg -t get_outputs` lists all the output devices.
/* TODO */

## Executing Lock before Suspend

### i3

`xss-lock` can automatically execute a command before suspending.

```
xss-lock --transfer-sleep-lock -- <i3-lock-command-that-you-want-to-execute> --nofork
```

### Sway

`swayidle` is the dependency of `Sway` that can execute commands after a certain time of idle or before a suspension.

```
exec swayidle -w \
  timeout 300 '<sway-lock-command-to-be-executed>' \
  timeout 600 'swaymsg "output * dpms off"' resume 'swaymsg "output * dpms on"' \
  before-sleep '<sway-lock-command-to-be-executed'
```

You can of course customize behavior after idling.

## Keyboard Settings

I don't know much about keyboard layout, but I know one thing: I don't like the location of the control key. Let's swap them with CapsLock. Emacs pinky is not a myth.

### i3

`setxkbmap` can be used to configure the keyboard layout.

```
exec --no-startup-id setxkbmap -option ctrl:swapcaps
```

### Sway

Execute `swaymsg -t get_inputs` to list the input devices. Once you get the name or id, you can utilize those to configure a specific device. Or you can configure the entire set of a device to behave a certain way.

```
input "type:keyboard" {
  xkb_options ctrl:swapcaps
}
```

## Network Management

`network-manager-applet`

## Nightlight/Nightshift/Bluelight filter/whatever it's called

In X11, `redshift`, and in Wayland, `gammashift`.

```
redshift -P -l 39.2:-86.5 -t 5600:3500 -m randr
gammastep -P -l 39.2:-86.5 -t 5600:3500
```

`-P` is telling the program to reset the current color settings before executing, `-l` is followed by lattitude:longitude, `-t` is followed by the color temperature for day and night, 6700k is the natural color temperature, and `-m randr` is a certain mode I think.

## Notification

`Dunst` is a lightweight notification daemon that can be used for both X11 and Wayland. There are many settings you can configure, but it should work well out of the box. To toggle the notification on and off ("Do Not Disturb mode"), execute `dustctl set-paused <true/false/toggle>`. You can also send a kill signal using `#kill -USR<1-to-pause-2-to-resume> $(pidof dunst)`. Note that sending notifications with summary (`notify-send DUNST_COMMAND_PAUSE`) has been removed due to its security (or loack of it).

You can send notifications using `notify-send` command. `notify-send` supports REGEX and multi-line notification. Below is a simple Polybar module to display the calendar when clicked.

```
[module/date]
type = internal/date
interval = 1
date = "%a %m-%d"
time = "%H:%M:%S"
label = %date% %time%
; A1 Left click, A2 middle, A3 right click, A4 Scroll up, A5 scroll down, etc
format = %{A1:notify-send "$(cal)"):}<label>%{A}
```

## Pulseaudio/Pipewire Volume Control

It looks like Pipewire, which is now default in Fedora, pretends to be Pulseaudio. It means that `pavucontrol`, a graphical manager for Pulseaudio, and `pactl`, which can be used to control volume, can be used. Below are the i3 keybindings for controlling volume using the hotkeys.

```
bindsym XF86AudioRaiseVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ +10%
bindsym XF86AudioLowerVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ -10%
bindsym XF86AudioMute exec --no-startup-id pactl set-sink-mute @DEFAULT_SINK@ toggle
bindsym XF86AudioMicMute exec --no-startup-id pactl set-source-mute @DEFAULT_SOURCE@ toggle
```

## Screenshot

### i3

X11 is blessed with a wide selection of screenshot utilities. My favorite is Flameshot. All you need to do is bind `flameshot launcher` to `Print` key or another keybinding of your choice.

### Sway

Wayland does not have many good screenshot tools. One simple and popular solution is using `grim`, a screenshot tool, and `slurp`, a region selector. Bind a key of your choice to `grim -g "$(slurp)"` to take a screenshot of a certain region.

## Startup Application

### i3

You probably know this, but you can use `exec` command in the (preferably) bottom of the i3 configuration to launch commands in the startup. `--no-startup-id` disables startup-notification, which is not supported by some applications (indicated by the cursor hourglass that will hang for 60 seconds).

### Sway

The reason why this section exists. Startup ID is not a thing in Wayland, so no need for `--no-startup-id` tag.

## Trackpad Settings

### i3

`xinput` can be used to list and configure input devices. `xinput list` to list all the input devices, and use `xinput list-props <device-id-or-name>` to list "props" associated with it. Pay close attention to things like "natural scrolling" and "tapping". Below are my i3 exec commands to enable natural scrolling and tapping.

```
set $trackpad_id <trackpad-id-or-name>
exec --no-startup-id xinput --set-prop $trackpad_id "libinput Tapping Enabled" 1
exec --no-startup-id xinput --set-prop $trackpad_id "libinput Natural Scrolling Enabled" 1
```

### Sway

Execute `swaymsg -t get_inputs` to list the input devices. Once you get the name or id, you can utilize those to configure a specific device. Or you can configure the entire set of devices to behave a certain way.

```
input "type:touchpad" {
  dwt enabled
  tap enabled
  natural_scroll enabled
  middle_emulation enabled
}
```

