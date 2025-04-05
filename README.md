# A Guide for Configuring i3/Sway WM

> When you mess something up, access tty using `<C-M,<F2 - F?>` (control alt F2).

## App Launcher

- `Dmenu` is the default launcher that comes as a weak dependency of `i3` by many package managers.
- `Rofi` is a popular launcher that can be themed and take custom input to be used as not only an app launcher but also as a window switcher, power menu, music control, etc.
    - `rofi -show run` lists all the executable files on the computers, and `rofi -modi drun -show drun` shows only the executables with `.desktop` files.
    - In Sway, install `rofi-wayland`

You can also make a custom launcher using `fzf` (Inspired by [A Guide to Switching From i3 to Sway](https://luxagraf.net/src/guide-to-switching-i3-to-sway)).

```
# For i3, replace "swaymsg" with "i3msg"
bindsym $mod+d exec <terminal-of-your-choice> --class 'launcher' --command bash -c 'compgen -c | sort -u | fzf | xargs -r swaymsg -t command exec'
# For i3, use "class" instead of "app_id"
for_window [app_id="^launcher$"] floating enable, sticky enable, resize set 30 ppt 60 ppt, border pixel 10, move center
```

[sway-launcher-desktop](https://github.com/Biont/sway-launcher-desktop) is an extension of the command above that supports highlighting desktop apps and history features.

## Battery Management

If you are installing i3/Sway along with other DEs such as Gnome or KDE, a chance is that it installed `powerprofilesctl` that integrates with DE UI and lets you choose among three power profiles.
Type `powerprofilesctl list` to view the current profile setting and available options and `powerprofilesctl set` to choose a profile.
Although its `power-saver` option throttles like crazy.

Another solution is `tlp`.
`tlp` works great out of the box, so start with `sudo tlp start`.
Follow the instruction to disable `powerprofilesctl` and how to add to `systemctl` daemon.
Execute `sudo tlp-stat` to see various information, including whether the `tlp` is enabled (the first string `TLP_ENABLE="1"`).

`powertop` can be installed to give more information about the battery, though I don't know how to interpret the data.

## Backlight Control

Use `brightnessctl`:

```
bindsym XF86MonBrightnessUp exec --no-startup-id brightnessctl set +10%
bindsym XF86MonBrightnessDown exec --no-startup-id brightnessctl set 10%-
```

[`light`](https://gitlab.com/dpeukert/light) is a dependency-free package you can use in lieu of `brightnessctl`

## Bluetooth

Install `blueman` and launch `blueman-manager`.

## Clipboard Management

### i3

Install your favorite clipboard manager (`clipit` or `copyq`) and enjoy.

### Sway

You probably want to install `wl-clipboard` to get the clipboard working. To save a clipboard history, install `clipman` (not XFCE plugin version, one for Wayland).
You can start storing clipboard history from `wl-clipboard` by executing `wl-paste -t text --watch clipman store`.
To view the clipboard history, you need to send the clipboard history to an external tool.
You can do this with `wofi`, with `clipman pick -t wofi`, or my preferred way, utilizing `fzf` and opening it up as a floating window by utilizing the command below.

```
bindsym <your-key-binding> <choice-of-a-terminal-emulator> --class=clipboard clipman pick --print0 --tool=CUSTOM --tool-args="fzf --prompt 'pick > ' --bind 'tab:up' --cycle --read0"
for_window [app_id="^clipboard$"] floating enable, sticky enable, resize set 30 ppt 60 ppt, border pixel 10, move position 1300px 50px

```

## Compositor

### i3

X11 server directly draws a window to the display buffer.
This is fine for most of the time, but if you're experiencing screen tearing or want to enable transparency/blur, you need to install a compositor.
`picom` is a great compositor that works out of the box.

### Sway

In Wayland, the compositor doubles as a window manager, meaning Sway is Wayland "compositor" that manages the window as well.
What does that mean?
That means you do not need a separate compositor for transparency to work!
Although I found Sway compositing to be limited, transparency for certain applications (Emacs) and rounded corners, blurs, etc are yet to come.

## Display Scaling

### i3

Make `.Xresources` in your home (`~`) directory and append `Xft.dpi: <DPI-value>` (in percentage: e.g., `Xft.dpi: 120` for 120%).
Most applications will follow the value specified in the DPI.

### Sway

`swaymsg -t get_outputs`, add `output <output_name> scale <DPI-value>` (in the scale: e.g, `output eDP-1 scale 1.2` for 120%) to your config.

## External Monitors

How i3 and Sway manage external monitos:

i3/Sway creates new workspaces in the current monitor that is focused on.
Let's suppose workspace 1 is in the left monitor and 2 is in the right monitor, and you are currently focused in the workspace 1.

- If you input `$mod + 3`, i3/Sway will create the workspace 3 in the left monitor.
- If you input `$mod + 2` to focus the right monitor then input `$mod + 3`, the workspace 3 will be created in the right monitor.

Following keybindings will be helpful in switching workspaces between multiple monitors.

```
bindsym $mod+Shift+braceright move workspace to output right
bindsym $mod+Shift+braceleft move workspace to output left
```

Now, if you are currently in the workspace 1 and input `$mod + Shift + ]`, the workspace 1 will be moved to the left monitor (now the left monitor has both workspace 1 and 2).

Fun fact: each monitor will have at least one workspace, so if you move all 10 workspaces to one monitor, the other will get the workspace 11.

### i3

`xrandr`, which should be installed as a dependency to X11 server, gives you the list of displays.
`xrandr --output HDMI-2 --auto --right-of eDP-1` projects the screen to HDMI-2 with the resolution set to auto.
You can then execute `i3 move workspace to output right` to move the current workspace to the externam monitor.
You can use `--same-as` flag to mirror the display.
`xrandr --output HDMI-2 --off` will stop the projection.

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

## Keyboard: Getting Keycodes

### i3

Install `xenv` and use the following command

```
xenv -event keyboard | egrep -o 'keycode.*\)'
```

## Keyboard: Layout Control

For setting up i3/Sway to handle input of non-Roman character, read the next section.

### i3

`setxkbmap` can be used to configure the keyboard layout.

```
# Swapping Ctrl and Capslock
exec --no-startup-id setxkbmap -option ctrl:swapcaps
# alt-space toggles between US QWERTY and French AZERTY
exec --no-startup-id setxkbmap -layout 'us,fr' -option 'grp:alt_space_toggle'
```

### Sway

Execute `swaymsg -t get_inputs` to list the input devices.
Once you get the name or id, you can utilize those to configure a specific device.
Or you can configure the entire set of a device to behave a certain way.

```
# Same setting as setxkbmap example above
input "type:keyboard" {
  xkb_layout us,fr
  xkb_options grp:alt_space_toggle
  xkb_options ctrl:swapcaps
  
}
```

## Keyboard: Non-Roman Input


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

`Dunst` is a lightweight notification daemon that can be used for both X11 and Wayland.
There are many settings you can configure, but it should work well out of the box.
To toggle the notification on and off ("Do Not Disturb mode"), execute `dustctl set-paused <true/false/toggle>`.

You can send notifications using `notify-send` command.
`notify-send` supports REGEX and multi-line notification.
Below is a simple Polybar module to display the calendar when clicked.

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

To view the previous notification, use `dunstctl history-pop`.
You can repeat this command until you get the desired notification.
Newer version of dunst supports `dunstctl history`, which returns previous notifications as JSON format.
The number of notification to be saved is controled by `history_length` variable in `dunstrc`.

## Polkit

Some GUI application that requires the Polkit authentication framework to work correctly (e.g., Fedora Media Writer) neeeds Polkit frontend installed.
`lxpolkit` is a lightweight one you can use.

## Pulseaudio/Pipewire Volume Control

Pipewire has a frontend for Pulseaudio, so it should work like Pulseaudio for the most part.
`pavucontrol`, a graphical manager for Pulseaudio, and `pactl`, which can be used to control volume, can be used with Pipewire.
Below are the i3 keybindings for controlling volume using the hotkeys.

```
bindsym XF86AudioRaiseVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ +10%
bindsym XF86AudioLowerVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ -10%
bindsym XF86AudioMute exec --no-startup-id pactl set-sink-mute @DEFAULT_SINK@ toggle
bindsym XF86AudioMicMute exec --no-startup-id pactl set-source-mute @DEFAULT_SOURCE@ toggle
```

## Screenshot

### i3

X11 is blessed with a wide selection of screenshot utilities.
My favorite is Flameshot.
Bind `flameshot launcher` to `Print` key or another keybinding of your choice.

### Sway

Wayland does not have many good screenshot tools.
One simple and popular solution is using `grim`, a screenshot tool, and `slurp`, a region selector.
Bind a key of your choice to `grim -g "$(slurp)"` to take a screenshot of a certain region.

## Startup Application

### i3

You can use `exec` command in the (preferably) bottom of the i3 configuration to launch commands in the startup.
`--no-startup-id` disables startup-notification, which is not supported by some applications (indicated by the cursor hourglass that will hang for 60 seconds).

### Sway

Exactly the same as i3, except startup ID is not a thing in Wayland, so no need for `--no-startup-id` tag.

## Trackpad Settings

### i3

`xinput` can be used to list and configure input devices.
`xinput list` to list all the input devices, and use `xinput list-props <device-id-or-name>` to list "props" associated with it.
Below are my i3 exec commands to enable natural scrolling and tapping.

```
set $trackpad_id <trackpad-id-or-name>
exec --no-startup-id xinput --set-prop $trackpad_id "libinput Tapping Enabled" 1
exec --no-startup-id xinput --set-prop $trackpad_id "libinput Natural Scrolling Enabled" 1
```

### Sway

Execute `swaymsg -t get_inputs` to list the input devices.
Once you get the name or id, you can utilize those to configure a specific device.
Or you can configure the entire set of devices to behave a certain way.

```
input "type:touchpad" {
  dwt enabled
  tap enabled
  natural_scroll enabled
  middle_emulation enabled
}
```

