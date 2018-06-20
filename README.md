# AI/IoT Gateway/Fog Node Setup
Some notes on how to set up an AI/IoT gateway/fog node using the Aaeon UP Squared.

## OS Installation
1. Install Ubuntu Server 16.04.4, 64-bit version
     If select OpenSSH during install, can skip step 3.1.
     In the following, assume machine is called "gateway".
     Substitute a different name as appropriate.
2. Update 
       ```
       sudo apt-get update
       sudo apt-get upgrade
       sudo apt-get dist-upgrade
       sudo reboot
       ```
3. To support HW, install UP2 kernel as described here:
       http://wiki.up-community.org/Ubuntu
     including upboard-extras and groups.

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
       sudo apt-get install openssh-server
2. Configure firewall to allow ssh, block everything else
       sudo ufw allow ssh
       sudo ufw limit ssh
       sudo ufw enable
3. On machine you want to login from, do
       ssh-copy-id gateway.local
     then test with
       ssh gateway.local

## Setup GUI: Minimal X Windows (for VNC and occasional direct use)
1. Install OpenBox (minimal X window manager; about 380MB)
     The following allows for a GUI but with the minimum use of storage
     See the following for more information on a similar configuration: 
       https://www.addictivetips.com/ubuntu-linux-tips/desktop-environment-with-openbox-window-manager/
     Summary:
       sudo apt install xinit openbox tint2 pcmanfm
2. Install a terminal but avoid pulling in too many gnome "extras"
       sudo apt-get install gnome-terminal --no-install-recommends
3. Install a webbrowser (note: 250MB! might want to try --no-install-recommends too)
       sudo apt-get install chromium-browser
     Alternatives: midori, firefox, google-chrome
4. Install network manager applet and start daemon
       sudo apt-get install network-manager-gnome
       sudo systemctl start network-manager
     Note: the daemon is automatically enabled after install, but is not started for some reason
5. Put the following in .config/openbox/autostart
       xset -b
       nm-applet &
       tint2 &
6. Fire up X, right click, select obconf, pick a theme (or not; the default, Clearlooks, is fine)
       startx
       -> obconf
     Set up panel, etc.
7. Install TurboVNC
     Follow instructions at 
       https://github.com/otcshare/ros-fetchbot/blob/master/sawr_master/VNC.md   
     this will start an remote X server on demand, using the defaults you set above

## Setup Development Tools
1. Install build tools
       sudo apt-get install build-essential git llvm clang g++ cmake scons
2. Install docker ce
       https://docs.docker.com/install/linux/docker-ce/ubuntu/
     including post-install steps: non-sudo access, systemd daemon autostart, disable dnsmasq, cgroup limits
       https://docs.docker.com/install/linux/linux-postinstall/
3. Install nodejs (version 6):
       cd ~
       curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
     Take a look at the above file with an editor if you are paranoid, then...
       sudo bash nodesource_setup.sh
       sudo apt-get install nodejs
     Node this also sets up "node" to point to the same thing as nodejs so you don't need nodejs-legacy

## Install Libraries (AI, Computer Vision, OpenCL, Optimized Python, etc.)
Note: these take a LOT of space; 16GB+.  The reason for the minimal install above is to
make room for this.  Even so the final system just barely fits (disk at 92%) on a gateway with 32GB.  A 64GB
system would be better, and forget about using a 16GB system.  You can save a little space 
(resulting in only 80% disk usage) by omitting the 32-bit versions of the libraries.
1. Download IPP, MKL, TBB, and DAAL from
       https://software.intel.com/en-us/performance-libraries
     Unpack and install.
2. Download OpenCL Drivers and SDK and install:
       https://software.intel.com/en-us/articles/opencl-drivers
       https://software.intel.com/en-us/intel-opencl/download
     Note: in order to benefit from this with OpenVINO, you need hardware that
     can support OpenCL 2.1.  For example, the N4200 version of the UP2 does... but
     the CHD on the UP Board does not, and lower-end versions of the UP2 may not.
3. Download and install OpenVINO:
      https://software.intel.com/en-us/openvino-toolkit

