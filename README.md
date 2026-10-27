
# Termux:X11

> **Fork notice:** This repository is a fork of [termux/termux-x11](https://github.com/termux/termux-x11) maintained by ESTRIN217 as part of the [Terminal](https://github.com/ESTRIN217/Terminal) fork. Here the project is built as Android libraries (`:termux-x11-lorie`, `:termux-x11-stub`) embedded into the Terminal app, not distributed as a standalone app. Binaries, Java namespaces (`com.termux.x11`), intent actions and the `termux-x11` command keep their upstream names. Not affiliated with the Termux team or with X.Org.

## About
Termux:X11 is a fully fledged X server. It is built with Android NDK and optimized to be used with Android GPL Linux environments — in this fork, with the [Terminal](https://github.com/ESTRIN217/Terminal) app.

## Submodules caveat
This repo uses submodules. Use 

```
git clone --recurse-submodules https://github.com/ESTRIN217/termux-x11 
```
or
```
git clone https://github.com/ESTRIN217/termux-x11
cd termux-x11
git submodule update --init --recursive
```

If you only want to build the Terminal app you don't need to clone this repo: it is integrated as a git submodule of [Terminal](https://github.com/ESTRIN217/Terminal) (clone it with `--recurse-submodules` too).

## How does it work?
Just like any other X server.

## Setup Instructions
Termux:X11 requires Android 8 or later. In this fork you don't install anything separately: the X server and the companion command are compiled into the [Terminal](https://github.com/ESTRIN217/Terminal) app (as the Android libraries `:termux-x11-lorie` / `:termux-x11-stub`), so installing the Terminal APK from [its releases](https://github.com/ESTRIN217/Terminal/releases) is all you need.

The companion `termux-x11` command is spawned by the app itself (`app_process` from the in-app companion spawner), not by a separate Termux package. If you work inside a proot/chroot guest, the app wires the environment (`DISPLAY`, `TMPDIR`, XKB paths) and you can launch `termux-x11 :0` from inside the guest. Usage documentation for this fork: <https://terminal-gnulinux-docs.web.app/x11.html>.

Finally, most people will want to use a desktop environment with Termux:X11. In this fork you would install it in the guest distribution (for example, `apt install xfce4` in the Debian rootfs shipped by Terminal). The rest of these instructions will assume that your goal is to run an XFCE desktop, or that you can modify the instructions as you follow them for your actual goal.

## Running Graphical Applications
You can start your desired graphical application by doing:
```
termux-x11 :1 -xstartup "dbus-launch --exit-with-session xfce4-session"
```
or
```
termux-x11 :1 -- dbus-launch --exit-with-session xfce4-session
```
or
```
termux-x11 :1 &
env DISPLAY=:1 dbus-launch --exit-with-session xfce4-session
```
You may replace `xfce4-session` if you use other than Xfce

`dbus-launch` does not work for some users so you can start session with
```
termux-x11 :1 -xstartup "xfce4-session"
```

Also you can do 
```
export TERMUX_X11_XSTARTUP="xfce4-session"
termux-x11 :1
```
In this case you can save TERMUX_X11_XSTARTUP somewhere in `.bashrc` or other script and not type it every time you invoke termux-x11.  


If you're done using Termux:X11 just simply exit it through its notification drawer by expanding the Termux:X11 notification then "Exit"
But you should pay attention that `termux-x11` command is still running and can not be killed this way.

For some reason some devices output only black screen with cursor instead of normal output so you should pass `-legacy-drawing` option.
```
termux-x11 :1 -legacy-drawing -xstartup "xfce4-session"
```

For some reason some devices show screen with swapped colours, in this case you should pass `-force-bgra` option.
```
termux-x11 :1 -force-bgra -xstartup "xfce4-session"
```

## Running multiple displays as separate windows
Each display normally shares the single Termux:X11 window. Pass `-tag <tag>` to open a display as its own Android window instead:
```
termux-x11 :1 -tag main -xstartup "xfce4-session"
termux-x11 :2 -tag second -xstartup "xfce4-session"
```
Re-running a tag brings its window back to front instead of opening a duplicate.

To bring up a tagged window's activity:
```
am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity -d main
```

## Using with proot environment
If you plan to use the program with proot, keep in mind that you need to launch proot/proot-distro with the --shared-tmp option. 

If passing this option is not possible, set the TMPDIR environment variable to point to the directory that corresponds to /tmp in the target container.

If you are using a guest distribution (as distributed in the Terminal fork, which
boots a Debian rootfs with proot), the app runs the companion for you and sets the
environment; the command form from inside the guest is:
```
termux-x11 :1 &
```
Then, inside the container:
```
export DISPLAY=:1
dbus-launch --exit-with-session xfce4-session
```

## Using with chroot environment
If you plan to use the program with chroot or unshare, you must run it as root and set the TMPDIR environment variable to point to the directory that corresponds to /tmp in the target container.

This directory must be accessible from the shell from which you launch termux-x11, i.e. it must be in the same SELinux context, same mount namespace, and so on.

Also you must set `XKB_CONFIG_ROOT` environment variable pointing to container's `/usr/share/X11/xkb` directory, otherwise you will have `xkbcomp`-related errors.

You can get the loader from the Terminal GitHub release artifacts; normally the app spawns it for you (`X11CompanionSpawner`), the manual invocation below is a fallback for advanced use.

Do not forget to disable SELinux
```
setenforce 0
export TMPDIR=/path/to/chroot/container/tmp
export CLASSPATH=$(/system/bin/pm path com.estrin217.terminal | cut -d: -f2)
/system/bin/app_process / --nice-name=termux-x11 com.termux.x11.CmdEntryPoint :0
```
In this fork the classes live in the Terminal APK (`com.estrin217.terminal`), but the entry-point class keeps its upstream name (`com.termux.x11.CmdEntryPoint`), same as the intent actions and the `termux-x11` process name.

### Force stopping X server (running in termux background, not an activity)

termux-x11's X server runs in process with name "termux-x11". You can kill it by
```
pkill -f termux-x11
```

### Closing Android activity (running in foreground, not X server)

```
am broadcast -a com.termux.x11.ACTION_STOP -p com.estrin217.terminal
```

To close a window opened with `-tag`:
```
am broadcast -a com.termux.x11.ACTION_STOP -p com.estrin217.terminal --es document_tag main
```

### Opening Termux:X11 activity from command line

```
am start --user 0 -n com.estrin217.terminal/com.termux.x11.MainActivity
```

> In this fork the activities live inside the Terminal app (`com.estrin217.terminal`), so the activity commands target that package while the class names keep their upstream `com.termux.x11` namespace. Normal session/window management is done from the Terminal UI (X11 tab), which internally uses the same actions.

### Logs
If you need to obtain logs from the X11 server (in this fork it belongs to the
`com.estrin217.terminal` application),
set the `TERMUX_X11_DEBUG` environment variable to 1, like this:
`TERMUX_X11_DEBUG=1 termux-x11 :0`

The log obtained in this way can be quite long.
It's better to redirect the output of the command to a file right away.

### Notification
In Android 13, posting notifications was restricted so you should explicitly let Termux:X11 show you notifications.
<details>
<summary>Video</summary>

[img_enable-notifications.webm](https://user-images.githubusercontent.com/9674930/227760411-11d440eb-90b8-451e-9024-d5a194d10b16.webm)

</details>

Preferences:
You can access preferences menu three ways:
<details>
<summary>By clicking "PREFERENCES" button on main screen when no client connected.</summary>

![image](./.github/static/1.jpg)
</details>
<details>
<summary>By clicking "Preferences" button in notification, if available.</summary>

![image](./.github/static/2.jpg)
</details>
<details>
<summary>By clicking "Preferences" application shortcut (long tap `Termux:X11` icon in launcher). </summary>

![image](./.github/static/3.jpg)
</details>

> **In this fork** there is no standalone Termux:X11 launcher presence, so only the notification path applies (when connected); the same underlying preferences are also exposed directly in the Terminal app at **Settings → Ajustes X11** (Output / Pointer / Keyboard sections).

## Toggling keyboard
Just press "Back" button.

## Touch gestures
### Touchpad emulation mode.
In touchpad emulation mode you can use the following gestures:
* Tap for click
* Double tap for double click
* Two-finger tap for right click
* Three-finger tap for middle click
* Two-finger vertical swipe for vertical scroll
* Two-finger horizontal swipe for horizontal scroll
* Three-finger swipe down to show-hide additional keys bar.
### Simulated touchscreen mode.
In simulated touchscreen mode you can use the following gestures:
* Single tap for left button click.
* Long tap for mouse holding.
* Double tap for double click
* Two-finger tap for right click
* Three-finger tap for middle click
* Two-finger vertical swipe for vertical scroll
* Two-finger horizontal swipe for horizontal scroll
* Three-finger swipe down to show-hide additional keys bar.

## Font or scaling is too big!
Some apps may have issues with X server regarding DPI. please see https://wiki.archlinux.org/title/HiDPI on how to override application-specific DPI or scaling.

You can fix this in your window manager settings (in the case of xfce4 and lxqt via Applications Menu > Settings > Appearance). Look for the DPI value, if it is disabled enable it and adjust its value until the fonts are the appropriate size.
<details>
<summary> Screenshot </summary>

![image](./.github/static/dpi-scale.png) 
</details>

Also you can start `termux-x11` with `-dpi` option.
```
termux-x11 :1 -xstartup "xfce4-session" -dpi 120
```

## Changing, dumping and restoring preferences from commandline

> **In this fork** the guests don't ship the `termux-x11-preference` tool (no
> `termux-x11-nightly` package is installed): manage the same preferences from the
> Terminal app (**Settings → Ajustes X11**). The upstream tool documented below
> still exists for Termux environments, where the package is available.

It is possible to change preferences of termux-x11 from command line.
`termux-x11-nightly` package contains `termux-x11-preference` tool which can be used like 
```shell
termux-x11-preference [list] {key:value} [{key2:value2}]...
```

Use `termux-x11-preference list` to dump current preferences.
Use `termux-x11-preference list > file` to dump current preferences to file.
Use `termux-x11-preference < file` to restore preferences from file.
Use `termux-x11-preference "fullscreen"="false" "showAdditionalKbd"="true"` to disable fullscreen and enable additional key bar. The full list of preferences you can modify is available with `termux-x11-preference list` command. You can specify one or more preferences here.

Termux:X11 activity should be available in background or foreground, otherwise `termux-x11-preference` tool will hang indefinitely.
In the case if there is `Store preferences for secondary displays separately` preference active `termux-x11-preference` will use/modify preferences of display where Termux:X11 activity is currently opened.

## Using with 3rd party apps
It is possible to use Termux:X11 with 3rd party apps.
Check how `shell-loader/src/main/java/com/termux/x11/Loader.java` works.

# License
Released under the [GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.html).

<p align="center">
  Desarrollado con pasión por <b>ESTRIN217</b>.
</p>

<p align="center">
  Hecho con ❤️ en Venezuela.
</p>