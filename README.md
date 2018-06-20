# AI Gateway Setup
Some notes on how to set up an AI gateway using the Aaeon UP Squared.
This system can perform _both_ IoT gateway functions (for example, acting as an IoT system
orchestrator, resource manager, and secure access portal) and "fog computing" offload engine functions
(for instance, running AI inferencing jobs supporting other local device in the network).

Hardware requirements: Aaeon UP2, N4200 processor (which supports OpenCL 2.1; other variants do not),
min 4GB RAM, min 32GB of EMMC (64GB recommended).
You probably also want a WiFi card and antennas, and if you want AI hardware acceleration, the AI Core PCIe card.

It will look something like this without a case:

![UP2 AI Gateway](images/inside2.jpg)

Put a nice case on it and it looks like this:

![UP2 AI Gateway](images/outside.jpg)

## OS Installation
1. Install Ubuntu Server 16.04.4, 64-bit version.
   This assumes you only have a 32GB gateway and want to install the minimal GUI manually (see below).
   If you have a 64GB gateway you might want to install Desktop Ubuntu instead and skip the minimal
   GUI installation.
   Assuming you do install Ubuntu server, if you select OpenSSH during the install, you can skip step 3.1.
   In the following, I assume the machine is called `gateway`.
   Substitute a different name as appropriate.
2. Update respository, packages, and distribution.  Reboot.
   ```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get dist-upgrade
   sudo reboot
   ```
3. To support HW, including I2C, SPI, etc,
   [install UP2 kernel](
       http://wiki.up-community.org/Ubuntu
   )
   including upboard-extras and groups.   Note: the standard UP2 case shown in the picture above does not
   provide external access to these ports, although you can run cables through the "extra" antenna ports.
   If you don't plan to use these ports you can probably skip this step, although for security reasons
   (eg getting the benefit of Spectre and Meltdown fixes) you may want to set up your system to track the HWE kernel.

## Networking Setup
1. To support mDNS, install `avahi-daemon`
   ```
   sudo apt-get install avahi-daemon
   ```
2. To browse for other mDNS services, install `avahi-utils`
   ```
   sudo apt-get install avahi-utils
   ```
   You now can scan for services as follows:
   ```
   avahi-browse -alr
   ```

## Enable Remote Access via SSH
1. Install OpenSSH server (if did not install during Ubuntu Server install)
   ```
   sudo apt-get install openssh-server
   ```
2. Configure firewall to allow ssh, block everything else
   ```
   sudo ufw allow ssh
   sudo ufw limit ssh
   sudo ufw enable
   ```
3. On machine you want to login from, do
   ```
   ssh-copy-id gateway.local
   ```
   then test with
   ```
   ssh gateway.local
   ```

## Setup GUI: Minimal X Windows (for VNC and occasional direct use)
1. Install OpenBox (minimal X window manager; about 380MB)
   The following [allows for a GUI but with the minimum use of storage](
       https://www.addictivetips.com/ubuntu-linux-tips/desktop-environment-with-openbox-window-manager/
   )
   ```
   sudo apt install xinit openbox tint2 pcmanfm
   ```
2. Install a terminal but avoid pulling in too many gnome "extras"
   ```
   sudo apt-get install gnome-terminal --no-install-recommends
   ```
3. Install a webbrowser (note: 250MB! might want to try --no-install-recommends here too)
   ```
   sudo apt-get install chromium-browser
   ```
   Alternatives to consider: `midori`, `firefox`, `google-chrome`
4. Install network manager applet and start daemon
   ```
   sudo apt-get install network-manager-gnome
   sudo systemctl start network-manager
   ```
   Note: the daemon is automatically enabled after install, but is not started for some reason.
   Rebooting will therefore also work.
5. Put the following in `.config/openbox/autostart`
   ```
   xset -b
   nm-applet &
   tint2 &
   ```
6. Fire up X, right click, select obconf from the menu, pick a theme (or not; the default, Clearlooks, is fine).
   ```
   startx
   ```
   You can open a terminal either by using the right-click menu or using Alt-Ctrl-T.
7. Configure theme (Optional; the default theme, Clearlooks, is fine).
   Right click, select obconf from the menu, and pick a theme.
8. Configure panel (Optional, just for convenience).
   Click on the button down at the lower left to configure the panel.  For example, you can
   add launchers for `gnome-terminal`, `chromium` (or whatever web browser you picked), and the file manager `pcmanfm`.
7. Install [TurboVNC](
       https://github.com/otcshare/ros-fetchbot/blob/master/sawr_master/VNC.md   
   ).
   This will start an offscreen X server on demand, using the defaults you set above.

## Setup Development Tools
1. Install build tools
   ```
   sudo apt-get install build-essential git llvm clang g++ cmake scons
   ```
2. Install [Docker CE](
       https://docs.docker.com/install/linux/docker-ce/ubuntu/
   )
   including [post-install steps](
          https://docs.docker.com/install/linux/linux-postinstall/
   ): non-sudo access, systemd daemon autostart, disable dnsmasq, cgroup limits

3. Install nodejs (version 6):
   ```
   cd ~
   curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
   ```
   Take a look at the above file with an editor if you are paranoid, then...
   ```
   sudo bash nodesource_setup.sh
   sudo apt-get install nodejs
   ```
   Note this also sets up `node` to point to the same thing as `nodejs` so you don't need to install `nodejs-legacy`.

## Install Libraries (AI, Computer Vision, OpenCL, Intel Python3, etc.)
Note: these take a LOT of space; 16GB+.  The reason for the minimal GUI install above is to
make room for this.  Even so the final system just barely fits (disk at 92%) on a gateway with 32GB.
A 64GB system would be better, and forget about using a 16GB system.  You can save a little space 
(resulting in only 80% disk usage) by omitting the 32-bit versions of the libraries.
1. Download [IPP, MKL, TBB, and DAAL](
       https://software.intel.com/en-us/performance-libraries
   ).
   Unpack and install.
2. Download [OpenCL Drivers](
       https://software.intel.com/en-us/articles/opencl-drivers
   ) and [SDK](
       https://software.intel.com/en-us/intel-opencl/download
   ) and install.
   In order to benefit from this with OpenVINO, you need hardware that
   can support OpenCL 2.1.  For example, the N4200 version of the UP2 does... but
   the CHD on the UP Board does not, and lower-end versions of the UP2 may not.
3. Download and install [OpenVINO](
      https://software.intel.com/en-us/openvino-toolkit
   ).

