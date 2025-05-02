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

- `upower -d` can display information about your power inputs/outputs.
- `powertop` diagnoses power consumption and settings that could potentially save battery life

There is a couple of tools you can use to manage the battery life of your laptop.

`power-profiles-daemon` (`ppd`) and `tuned-ppd` are both power profile switcher, with `tuned-ppd` being Redhat's `ppd` drop-in replacement.

```sh
# add tuned-ppd to the systemctl daemon list
systemctl enable tuned

# lists all available power profiles
tuned-adm list
# select a profile (most commonly used defaults are balanced-battery, powersave, and throughput-performance)
tuned-adm profile balanced-battery
```

Use `powerprofilesctl list` and `powerprofilesctl set` if you are using `ppd`.
Waybar provides a built-in module for `tuned-ppd` and `ppd` that lets you display the current profile and cycle through different profiles.

`tlp` dynamically adjust CPU performance and other IO/network settings to save battery life.
Follow the [installation guide](https://linrunner.de/tlp/installation/index.html) for your distribution.

```sh
sudo systemctl enable tlp.service

# Get tlp inforation
tlp-stat -s
# list all batteries
sudo tlp-stat -b
# get the current configuration
sudo tlp-stat -c
# view the difference between the defaults and the user config
tlp-stat --cdiff
```

Works great out of the box, change `/etc/tlp.conf` based on the [configuration guide](https://linrunner.de/tlp/settings/index.html) as needed.

## Backlight

Use `brightnessctl`:

```
bindsym XF86MonBrightnessUp exec --no-startup-id brightnessctl set +10%
bindsym XF86MonBrightnessDown exec --no-startup-id brightnessctl set 10%-
```

In sway, `--no-startup-id` is not needed and `--locked` option can be used to enable brightness control even when swaylock is active.

```
bindsym --locked XF86MonBrightnessUp exec brightnessctl set +10%
bindsym --locked XF86MonBrightnessDown exec brightnessctl set 10%-
```

## Bluetooth

Install `blueman` and launch `blueman-manager`.

## Clipboard History

### i3

`clipit` or `copyq`.

### Sway

Install `clipman` and add the following the your config.

```
# record clipboard history
exec wl-paste -t text --watch clipman store --no-persist

# pick a clipboard content using rofi
bindsym $mod+Shift+v clipman pick -t rofi
# reset clipboard history
bindsym $mod+Shift+r clipman clear --all
```

There is currently no good ways to save images or other rich format contents in Wayland.

## Compositor

### i3

X11 server directly draws a window to the display buffer.
This is fine for most of the time, but if you're experiencing screen tearing or want to enable transparency/blur, you need to install a compositor.
`picom` is a great compositor that works out of the box.

### Sway

In Wayland, the compositor doubles as a window manager, meaning Sway is Wayland "compositor" that manages the window as well.

## Display Scaling

### i3

Make `.Xresources` in your home (`~`) directory and append `Xft.dpi: <DPI-value>` (e.g., `Xft.dpi: 120` for 120%).
Most applications will follow the value specified in the DPI.

### Sway

`swaymsg -t get_outputs`, add `output <output_name> scale <DPI-value>` (e.g, `output eDP-1 scale 1.2` for 120%) to your config.

## External Monitors

How i3 and Sway manage external monitors:

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

- `xrandr --output HDMI-2 --auto --right-of eDP-1` projects the screen to HDMI-2 with the resolution set to auto.
- You can then execute `i3 move workspace to output right` to move the current workspace to the external monitor.
- You can use `--same-as` flag to mirror the display.
- `xrandr --output HDMI-2 --off` will stop the projection.

### Sway

`swaymsg -t get_outputs` lists all the output devices.
When the monitor connects, Sway automatically turns the monitor on.
However, there is no easy way to set the absolute position of the monitor like wiht `--right-of`, etc. flag in `xrandr`; you need to know the resolution of the monitors.

- `swaymsg 'output HDMI-2 pos 0 0'` to render `HDMI-2` to the far left
- `swaymsg 'output eDP-1 pos 0 0'; swaymsg 'output HDMI-2 pos 1920 0` to render `HDMI-2` to the right of `eDP-1`, which has the resolution of 1920 x 1080
- `swaymsg 'output HDMI-2 toggle` toggles the display

## Executing Lock before Suspend

### i3

`xss-lock` can automatically execute a command before suspending.

```
xss-lock --transfer-sleep-lock -- <i3-lock-command-that-you-want-to-execute> --nofork
```

### Sway

`swayidle` is the dependency of `Sway` that can execute commands after a certain time of idle or before a suspension.
Following command in your configuration locks the machine after 300 seconds, turn off the display after another 300 seconds, and launches swaylock before suspending.

```
exec swayidle -w \
  timeout 300 'swaylock -f' \
  timeout 600 'swaymsg "output * power off"' resume 'swaymsg "output * power on"' \
  before-sleep 'swaylock -f'
```

`swaylock -f` runs swaylock in daemonized mode, which is preferred method to prevent it being triggered multiple times.

## Keyboard: Getting Keycodes

### i3

Install `xenv` and use the following command

```sh
xenv -event keyboard | egrep -o 'keycode.*\)'
```

### Sway

Run `wev` and look for the output of the following format:

```
[14:     wl_keyboard] key: serial: 3516; time: 1048491; key: 246; state: 0 (released)
                      sym: XF86WLAN     (269025173), utf8: ''
```

If `wev` does not respond with an input, it is likely that Sway already has a keybinding of the key and is hijacking the input.

## Keyboard: Layout Control

For handling the input of non-Roman characters, read the next section.

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

To input non-Roman characters, you need an input tool such as `ibus` or `fcitx`.
I will show you the example of setting Korean character (Hangul) input on Fedora using `fcitx5` (setting `ibus` should very similar, as I will indicate in the comments, but I found `ibus-hangul` to be less reliable than `fcitx5-hangul`).

First, install dependencies and configure `fcitx5`:

```sh
# 1. Install a font
sudo dnf intall adobe-source-han-sans-kr-fonts

# 2. Install `fcitx5` and Hangul package for it
sudo dnf install fcitx5 fcitx5-hangul
#sudo dnf install ibus ibus-hangul

# 3. Use a GUI frontend to configure fcitx5
#    Add Hangul input and set up keybindings (default Ctrl-SPC)
fcitx5-configtool
#ibus-setup

# 4. Launch the daemon. Add it to your i3 config
fcitx5 -d --replace
#ibus-daemon --daemonize --xim --replace
```

Once you installed and configured `fcitx5`, head to your `~/.bash_profile` and export environment variables to let the X11 know what input source to use.
You can have this in other files such as `~/.xprofile`, `/etc/environment`, or `/etc/profile`, but [`fcitx` wiki recommends `bash_profile`](https://fcitx-im.org/wiki/Setup_Fcitx_5#Login_shell_profile).

```
$ cat ~/.bash_profile
...

export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export SDL_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx

# - this variable is only used by the Kitty terminal emulator,
#   and Kitty only supports ibus
# - however, fcitx has ibus compatibility,
#   meaning you can set this (or any other IM_MODULE variables, in fact)
#   to ibus and still use fcitx as your input source
export GLFW_IM_MODULE=ibus

```

## Network Management

With `nmcli`:

```sh
# lists available networking device
nmcli device status
# lists all available wifi
nmcli d wifi list --rescan yes
# connects to a wifi (replace $SSID and $PASSWORD)
nmcli d wifi connect $SSID password $PASSWORD
# shows the password of the currently connected wifi
nmcli d wifi show-password

# lists active connection profiles (e.g., all attempted wifi connections)
nmcli c show
# switch to another registered connection (e.g., different wifi)
nmcli c up $CONNECTION_NAME
# disconnect from connection
nmcli c down $CONNECTION_NAME
# delete information associated with the connection
# this deletes the profile file located in /etc/NetworkManager/system-connections/$SSID.nmconnection
nmcli c delete $CONNECTION_NAME

# turns wifi device off
nmcli radio wifi off
```

You can also install `network-manager-applet` to have a GUI frontend for `nmcli`.

## Nightlight/Nightshift/Bluelight filter/whatever it's called

### i3

`redshift -P`

`-P` tells redshift to reset the current color settings before executing.

Create a configuration file as `~/.config/redshift.conf`

```
[redshift]
temp-day=5600
temp-night=3500
gamma=0.8
adjustment-method=randr
location-provider=manual

[manual]
lat=<decimal degree of your loc>
lon=<decimal degree of your loc>
```

### Sway

`gammastep -P`

Create a configuration file as `~/.config/gammastep/config.ini`

```
[general]
temp-day=5600
temp-night=3500
gamma=0.8
adjustment-method=wayland
location-provider=manual

[manual]
lat=<decimal degree of your loc>
lon=<decimal degree of your loc>
```

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

Reference the [backight](#backlight) section for using `--locked` flag for Sway config.

## Screenshot

### i3

X11 is blessed with a wide selection of screenshot utilities.
My favorite is Flameshot.
Bind `flameshot launcher` to `Print` key or another keybinding of your choice.

### Sway

Use `grimshot`

```
set $screenshot_mode Screenshot MODE: (w) Window (s) Entire screen (a) Area (esc, Return) Exit
mode "$screenshot_mode" {
  bindsym w exec grimshot --notify save window; mode "default"
  bindsym s exec grimshot --notify save screen; mode "default"
  bindsym a exec grimshot --notify save area; mode "default"
  bindsym Escape mode "default"
  bindsym Return mode "default"
}
bindsym $mod+Shift+s mode "$screenshot_mode"
```

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

Below disables the trackpad while typing, enables tap to click, and enables the natural scrolling.

```
input "type:touchpad" {
  dwt enabled
  tap enabled
  natural_scroll enabled
  middle_emulation enabled
}
```

