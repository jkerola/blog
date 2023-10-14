---
title: "Installing Hyprland on Ubuntu 23.10"
date: 2023-10-12T16:07:03+03:00
draft: false
---

I've been keen on trying out tiling window managers for a while now. Sometimes a cool setup from [r/unixporn](https://reddit.com/r/unixporn) crops up in my feed, featuring either [SwayWM](https://swaywm.org/) or [Hyprland](https://hyprland.org) as the driving force. Hyprland is currently under active development and depends on the very latest packages for its releases. As of writing, the current version is v0.30.0 and the latest up-to-date tutorial for installing Hyprland on Ubuntu is for v0.24.1.

I figured I might as well document my attempt at building the latest version for others' sake. If you find this document helpful, share it with others who may find it useful.

---

**Note** -- Most of this is not my own work. I'm just compiling everything I come across during my attempts into a single document. I've also listed the _bare minimum_ commands here into a single executable script if you don't want to read stuff. See [hyprland-ubuntu-2310.sh](https://gist.github.com/jkerola/0db73208f9578683cc6c8f071a603fc4) gist.

---

# Setup

I will be using an old personal HP Probook laptop with a AMD Radeon GPU as the target machine. If you are an Nvidia user, please see the wiki on [Nvidia and Hyprland.](https://wiki.hyprland.org/Nvidia/)

I'm using a fresh install of the newly released [Ubuntu Server 23.10](https://ubuntu.com/download/server) as the starting point. If you are following along with a desktop release, note that some of the steps detailed here may already be complete on your device.
After configuring the network and firewall, we should install the recommended packages shown in the [Hyprland wiki:](https://wiki.hyprland.org/Getting-Started/Installation/)

```shell
$ sudo apt-get install -y meson wget build-essential \
        ninja-build cmake-extras cmake gettext gettext-base \
        libfontconfig-dev libffi-dev libxml2-dev libdrm-dev \
        libxkbcommon-x11-dev libxkbregistry-dev libxkbcommon-dev \
        libpixman-1-dev libudev-dev libseat-dev seatd libxcb-dri3-dev \
        libvulkan-dev libvulkan-volk-dev vulkan-validationlayers-dev \
        libvkfft-dev libgulkan-dev libegl-dev libgles2 libegl1-mesa-dev \
        glslang-tools libinput-bin libinput-dev libxcb-composite0-dev \
        libavutil-dev libavcodec-dev libavformat-dev libxcb-ewmh2 \
        libxcb-ewmh-dev libxcb-present-dev libxcb-icccm4-dev \
        libxcb-render-util0-dev libxcb-res0-dev libxcb-xinput-dev \
        xdg-desktop-portal-wlr fontconfig
```

For Ubuntu 23.04, [Vertecedoc4545's gist](https://gist.github.com/Vertecedoc4545/3b077301299c20c5b9b4db00f4ca6000) instructs building the `libliftoff`, `libwayland`, `wayland-protocols` and `libdisplay-info` packages from source. However, in Ubuntu 23.10 this is not necessary! With `apt search <package>` we can quickly see that the required packages have been updated and are readily available through apt.

Furthermore, the Hyprland build process requires packages which are not documented in the gist. I found these out by trial and error during the build, and have included them here.

```shell
$ sudo apt-get install -y libliftoff-dev libdisplay-info-dev \
        libpango1.0-dev libsystemd-dev libgbm-dev hwdata \
        xwayland wayland-protocols libwayland-dev
```

**Note** -- Future versions of Hyprland will require manually building updated packages not in apt from source.

# Build and install Hyprland

Once the build dependencies are met, we just need to download the Hyprland source code and extract it:

```shell
$ wget https://github.com/hyprwm/Hyprland/releases/download/v0.30.0/source-v0.30.0.tar.gz
$ tar -xzvf source-v0.30.0.tar.gz
```

then build and install Hyprland:

```shell
$ cd hyprland-source
$ sudo make install
```

If you run into errors, check the printed log to see if you missed installing a package somewhere.

> **Pro tip:** Pipe the installation process into a log file with
>
> `sudo make install > install.log 2>&1`,
>
> then run
>
> `$ cat install.log | less`
>
> to view the log file in a server environment!

Once the process is complete, test that the installation finished succesfully by launching Hyprland:

```shell
$ Hyprland
```

If everything went as expected, you should see the Hyprland splash-screen and be able to exit the environment with `SUPER + M`.

# Configuring Hyprland

In order to start using Hyprland, we should first configure some defaults. I'm trying something new with the [Alacritty terminal](), but any other terminal emulator should work just fine as well. I'll be using [pavucontrol](https://freedesktop.org/software/pulseaudio/pavucontrol/) to manage my sound devices. I'm also using the snap version of [Firefox](https://www.mozilla.org/en-US/firefox/new/) for convenience. For managing files, I prefer Nautilus over Dolphin.

[Waybar](https://github.com/Alexays/Waybar) creates a neat top-bar filled with status icons for battery, volume etc. It works with a lot of programs out of the box and has a ton of customization options. Requires `fonts-font-awesome` package to display icons properly.

[Fuzzel](https://codeberg.org/dnkl/fuzzel) is a neat customizeable application launcher. Don't skip this unless you want to launch everything from the terminal.

```shell
$ sudo apt install alacritty nautilus waybar \
        fuzzel pavucontrol fonts-font-awesome
$ sudo snap install firefox
```

Now, lets start configuring things! The default configuration for Hyprland is found in _~/.config/hypr/hyprland.conf_.

```shell
$ nano ~/.config/hypr/hyprland.conf
...
# Remove this line to get rid of the splash-screen warning
autogenerated = 1
...
# Let's launch waybar on startup!
exec-once = waybar
...
# Update the keyboard configuration
input {
    kb_layout = fi # defaults to 'us', change it before you get stuck ;)
    # ...
    touchpad {
        # keep this as 'no' if you want to drag down to scroll down
        natural_scroll = yes
    }
}
...
# By default, Hyprland uses win/super key as the main modifier
$mainMod = Super # CTRL, ALT etc. are all valid options
...
# Modify the binds to your hearts desire
bind = CONTROL_ALT, T, exec, alacritty # I like gnome keybinds :)
bind = CONTROL, W, killactive
bind = CONTROL, SPACE, exec, fuzzel --show-actions
...
```

Save the configuration, launch Hyprland again and test out your changes! Use fuzzel to launch firefox and test your audio. You may need to run `pulseaudio --start` in order to get things going.

This document is meant to get you started with a few reasonable defaults. The [Hyprland wiki](https://wiki.hyprland.org/) is an _excellent_ resource for further customizing the environment to your specific needs.

# Conclusion

Writing tutorials is hard. I've spent more time trying to write clear instructions than it took to execute them. I hope the fruit of my efforts still applies in 6 months when Ubuntu 24.04 is released!

I'm thinking of writing more these things in the future. I've got some of useful tips I think others could benefit from.

-- Janne
