# Install mininet in wsl

## Table of contents

* [Overview](#overview)
* [Environment Check](#env)
* [Build Kernel](#kernel)
* [Install Mininet](#mininet)


<a name="overview"></a>

## Overview

To make mininet run normally in wsl, you need to 
* build an openvswitch enabled kernel
* install mininet and its components.


<a name="env"></a>

## Environment Check

If any of the **bolded** parts is inconsistent, I strongly recommend against using this tutorial.

In cmd, run
```bash
# Update your WSL version to the latest version.
wsl --update
# Check the version information about WSL and its components.
wsl -v
```

This tutorial runs with the following environment
- WSL version： **1.2.5.0**
- kernel version： 5.15.90.1
- WSLg version： 1.0.51
- MSRDC version： 1.2.3770
- Direct3D version： 1.608.2-61064218
- DXCore version： 10.0.25131.1002-220531-1700.rs-onecore-base2-hyp
- Windows version： 10.0.19045.3448

In wsl, run
```bash
# Print all system information
uname -a
# Print all districution information
lsb_release -a
```

This tutorial runs with the following enviornment

- Linux LAPTOP 5.15.90.1-microsoft-standard-WSL2 #1 SMP Fri Jan 27 02:56:13 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux

- No LSB modules are available.
- Distributor ID: Ubuntu
- Description:    **Ubuntu 22.04.2 LTS**
- Release:        22.04
- Codename:       jammy

Still in wsl, run
```bash
cat /etc/wsl.conf
```

You should get
- **[boot]**
- **systemd=true**


<a name="kernel"></a>

## Build Kernel

### 1. Install essential tools to build the kernel. In wsl, run
```bash
sudo apt update
sudo apt upgrade
sudo apt install make autoconf build-essential flex bison bc dwarves libncurses-dev libssl-dev libelf-dev libtool
```

### 2. Download the source of kernel. Go to the [releases](https://github.com/microsoft/WSL2-Linux-Kernel/releases) of the Linux kernel used in WSL2. Download the release that matches your kernel version, which is in the output of `wsl -v`.
```bash
# Choose any working directory
cd ~
# Downlaod the kernel source using the link from github.
wget https://github.com/microsoft/WSL2-Linux-Kernel/archive/refs/tags/linux-msft-wsl-<your kernel version>.tar.gz
# Unzip tarball
tar -xf linux-msft-<your kernel version>.tar.gz
# Change directory
cd WSL2-Linux-Kernel-linux-msft-wsl-<your kernel version>
```

### 3. Configure kernel
```bash
# Copy the current kernel configuration
zcat /proc/config.gz > .config
# Configure in a special user interface
make menuconfig
```
**Read the instructions at the top of the screen.** You can use &larr; and &rarr; to switch among `select`, `Exit`, `Help`, `Save`, `Load`. You can use &uarr; and &darr; to switch among features. You need to include/modularize the following features. `[=y]` means includes (pressing `<Y>`) and `[=m]` means modularizes (pressing `<M>`).

- Symol: OPENVSWITCH [=m]
    - Name: Open vSwitch
    - Location:
    - -> Networking support (NET [=y])
        - -> Networking options

- Symol: NET_SCH_HTB [=y]
    - Name: Hierarchical Token Bucket (HTB)
    - Location:
    - -> Networking support (NET [=y])
        - -> Networking options
            - -> QoS and/or fair queueing (NET_SCHED [=y])

- Symol: NET_SCH_TBF [=y]
    - Name: Token Bucket Filter (TBF)
    - Location:
    - -> Networking support (NET [=y])
        - -> Networking options
            - -> QoS and/or fair queueing (NET_SCHED [=y])

- Symol: NET_SCH_NETEM [=y]
    - Name: Network emulator (NETEM)
    - Location:
    - -> Networking support (NET [=y])
        - -> Networking options
            - -> QoS and/or fair queueing (NET_SCHED [=y])

### 4. Build kernel
Check the number of available CPU cores/threads
```bash
nproc
```
Make, where `<N>` should be the result of `nproc`
```bash
make -j<N>
```
Install modules
```bash
make modules_install
```
Copy the kernel to Windows, replace `<username>` with your user name in Windows.
```bash
cp arch/x86/boot/bzImage /mnt/c/users/<username>/mn-bzImage
```
Clean
```bash
cd ..
rm -rf WSL2-Linux-Kernel-linux-msft-wsl-<your kernel version>
```
After you complete this tutorial, you can also delete the tarball.

### 5. Configure wsl to use the new kernel
Now you need to switch back to Windows. First shutdown the wsl
```bash
# Run this in cmd
wsl --shutdown
```
Create a file `.wslconfig` in your `%UserProfile%` directory. Write the following into the file. Replace `<username>` with your user name in Windows.
```
[wsl2]
kernel=c:\\users\\<username>\\mn-bzImage
```

<a name="mininet"></a>

## Install Mininet
Open your wsl. Run
```bash
# Install mininet and its components
sudo apt install mininet openvswitch-switch openvswitch-testcontroller
# Disable run at startup of controller to fix an issue with python
systemctl disable openvswitch-testcontroller
```

## Finished
Remember to delete the tarball of kernel source.

## References
https://zenn.dev/takai404/articles/9c96d5d1bcc9d0