# About this repository

This repository is forked from the [Beckhoff TwinCAT/BSD Hypervisor repository](https://github.com/Beckhoff/TCBSD_Hypervisor_Samples). The files are adapted for the EV BDL at Circu Li-ion.

## Getting started

1. Clone this repository onto the IPC home directory.

The following steps are adapted from the [official documentation](https://download.beckhoff.com/download/Document/ipc/embedded-pc/embedded-pc-cx/TwinCAT_BSD_en.pdf)

2. Setup VM files (chapter 10.11 Steps 1-4)
``` console
doas zfs create -p -o mountpoint=/vms/chimera_vm zroot/vms/chimera_vm
doas fetch -o /vms/ubuntu-installer.iso https://releases.ubuntu.com/24.04/ubuntu-24.04-desktop-amd64.iso
doas zfs create -V 50G zroot/vms/chimera_vm/disk0
doas cp /usr/local/share/uefi-firmware/BHYVE_BHF_UEFI_VARS.fd /vms/chimera_vm/EFI_VARS.fd
```

3. Setup network (chapter 10.11 Step 5 + chapter 10.9.3)

``` console
# bridge0/tap0 - connects VM to internet
doas ifconfig bridge create
doas ifconfig tap create

# bridge1/tap1 - connects VM to 
doas ifconfig bridge create
doas ifconfig tap create

# setup network
doas ifconfig bridge0 addm em0 addm tap0 up
doas ifconfig bridge1 addm igc2 addm tap1 up

# create at system startup
doas sysrc cloned_interfaces+="bridge0 tap0 bridge1 tap1"
doas sysrc ifconfig_bridge0="addm em0 addm tap0 up"
doas sysrc ifconfig_bridge1="addm igc2 addm tap1 up"
```

4. PCI device passthrough (chapter 10.10)

````
doas devctl set driver -f pci0:0:2:0 ppt
doas devctl set driver -f pci0:0:20:0 ppt
```

3. Autostart VM (chapter 10.4)

```console
cd ~/TCBSD_Hypervisor/vm_autostart
doas make
doas service chimera_vm enable
doas service chimera_vm start
```

4. Enable ssh (inside VM)

``` console
sudo apt update
sudo apt install openssh-server -y
```

## Backup & Restore

```
doas snapshot zfs zroot/vms/chimera_vm/disk0@latest
doas zfs send zroot/vms/chimera_vm/disk0@latest > /tmp/YYMMDD_chimera_vm.zfs
```

Then backup to external storage

Restore: Refer to manual