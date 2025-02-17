# way-displays: Auto Manage Your Wayland Displays

<img align="right" width="427" height="189" title="credit: Stephen Barratt" src="doc/layouts.png?raw=true">

1. Set resolution/refresh: preferred, highest or custom

1. Enable VRR / adaptive sync

1. Arrange in a row or a column

1. Auto scale based on DPI: 96 is a scale of 1

1. Update when displays plugged/unplugged

1. Update when laptop lid closed/opened

Works out of the box: no configuration required.

Wayland successor to [xlayoutdisplay](https://github.com/alex-courtis/xlayoutdisplay), inspired by [kanshi](https://sr.ht/~emersion/kanshi/).

<!-- gh-md-toc --no-backup --hide-footer README.md -->
<!--ts-->
   * [Requirements](#requirements)
   * [Quick Start](#quick-start)
   * [Usage](#usage)
   * [Installation](#installation)
      * [Package Manager](#package-manager)
      * [From Source](#from-source)
   * [Known Issues with Workarounds](#known-issues-with-workarounds)
   * [What Is Preferred Mode?](#what-is-preferred-mode)
   * [On Scale And Blurring](#on-scale-and-blurring)
   * [Questions, Suggestions And Ideas](#questions-suggestions-and-ideas)
   * [Help Wanted - GUI Configurator](#help-wanted---gui-configurator)
<!--te-->

## Requirements

way-displays is known to work on the [sway](https://swaywm.org/), [river](https://github.com/riverwm/river) and [Hpyrland](https://hyprland.org/) compositors. It may work on any wlroots compositor that supports the WLR Output Management protocol.

The user must be a member of the `input` group.

## Quick Start

[CONFIGURATION](doc/CONFIGURATION.md)

```
mkdir -p ~/.config/way-displays
cp /etc/way-displays/cfg.yaml ~/.config/way-displays/cfg.yaml
```

### Sway

Remove any `output` commands from your sway config file and add the following:
```
exec way-displays > /tmp/way-displays.${XDG_VTNR}.${USER}.log 2>&1
```

### River

Add the following to your `init`:
```
way-displays > /tmp/way-displays.${XDG_VTNR}.${USER}.log 2>&1 &
```

### Hyprland

Create a launcher: `${HOME}/.config/hypr/start-way-displays.sh`
```sh
#!/bin/sh

sleep 1 # give Hyprland a moment to set its defaults

way-displays > "/tmp/way-displays.${XDG_VTNR}.${USER}.log" 2>&1
```

Make it executable:
```sh
chmod 755 ${HOME}/.config/hypr/start-way-displays.sh
```

Add the following to your `hyprland.conf`:
```
exec-once = ${HOME}/.config/hypr/start-way-displays.sh
```

### Configure

Restart the compositor and run `way-displays -g` or look at `/tmp/way-displays.1.me.log`.

[Tweak cfg.yaml](doc/CONFIGURATION.md#cfgyaml) to your liking and save it. Changes will be immediately applied.

Alternatively, use the [command line](doc/CONFIGURATION.md#command-line) to make your changes then persist them with `way-displays -w`.

You might want to `tail -f /tmp/way-displays.1.me.log` whilst you are tweaking.

## Usage

See [CONFIGURATION](doc/CONFIGURATION.md) for details on `cfg.yaml` and the command line.

Start the `way-displays` server by running once with no arguments after your wayland compositor has been started.

It will remain in the background, responding to changes, such as plugging in a display, and will terminate when you exit the compositor.

It will print messages to inform you of everything that is going on.

You can interact with the server via the [command line](doc/CONFIGURATION.md#command-line).

The server responds to [IPC](doc/IPC.md) requests to fetch and mutate state.

## Installation

### Package Manager

[![Packaging status](https://repology.org/badge/vertical-allrepos/way-displays.svg)](https://repology.org/project/way-displays/versions)

### From Source

[![CI](https://github.com/alex-courtis/way-displays/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/alex-courtis/way-displays/actions/workflows/ci.yml?query=branch%3Amaster)

See [CONTRIBUTING](CONTRIBUTING.md)

#### Install / Uninstall

```
sudo make install
sudo make uninstall
```

## Known Issues with Workarounds

### Laptop Lid Not Detected - Permission Denied

```
W [10:09:44.542] WARNING: open '/dev/input/event0' failed 13: 'Permission denied'
```

User must be in the `input` group to monitor libinput events.

### Laptop Lid Not Closed At Startup

Fixed in [libinput 1.21.0](https://gitlab.freedesktop.org/libinput/libinput/-/releases/1.21.0).

### Unusable Displays Following MODE

One or many displays may be rendered unusable after setting a `MODE`. This has occurred when a higher resolution/refresh than the preferred has been selected, particularly when using a HDMI cable.

It may be possible to work around this by setting `WLR_DRM_NO_MODIFIERS=1`. See [wlroots documentation](https://gitlab.freedesktop.org/wlroots/wlroots/-/blob/master/docs/env_vars.md) for details.

You can set it when directly starting sway e.g.
```shell
WLR_DRM_NO_MODIFIERS=1 sway ...
```

If you use a display manager, you will need to export it from your non-login shell environment e.g. `.zshenv`.
```shell
export WLR_DRM_NO_MODIFIERS=1
```

### Scaling Breaks X11 Games

When a display is scaled (X11) linux games will render at the display's scaled resolution, rather than the monitor's native resolution. There is [work underway](https://gitlab.freedesktop.org/wlroots/wlroots/-/issues/2125) to fix this.

In the meantime, you may temporarily disable scaling via `way-displays -s SCALING off`

## What Is Preferred Mode?

Displays advertise their available modes when plugged in. Some displays specify a mode as "preferred".

The preferred mode is usually the highest resolution/refresh available and it's a good default. You shouldn't need to tweak this.

In some cases the preferred mode is a horrid "compatibility" mode e.g. `1024x768@60Hz`. You could fix this by setting `MODE` to `MAX` for that display.

## On Scale And Blurring

When using a display scale that is not a whole number, the result will not be a pixel perfect rendition of the unscaled content. There are no fractional pixels so there will be rounding and thus some blurring.

To ameliorate this, we always round our scale to a multiple of one eighth. This results in a nice round binary number, which minimises some of the rounding and results in a smoother image. If you're interested, our rounded scale is a [wl_fixed_t](https://wayland.freedesktop.org/docs/html/apb.html).

## Questions, Suggestions And Contributions

Please create a [github issue](https://github.com/alex-courtis/way-displays/issues) and attach the log `/tmp/way-displays.1.me.log`

[Contributions](CONTRIBUTING.md) are always welcome.

[Milestone 1.10](https://github.com/alex-courtis/way-displays/milestone/2) contains a list of issue I would be eternally grateful for you to work on.
