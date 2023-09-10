# proxmox-omv-lxc
Notes for setting up OpenMediaVault as an LXC container on Proxmox

1. Start with tteck's helper script for OMV from here: Use https://tteck.github.io/Proxmox/
2. Setup a ZFS volume that will be used as the base mount for the ZFS filesystem in OMV LXC

```bash
# Create a small zvol (1MB) base mount
zfs create -V 1MB deadpool3/omv-root
# Partition the new zvol - create dos disklabel and a primary partition
fdisk /dev/zd0
# Create a FS for 1st partition of zvol
mkfs.ext4 /dev/zd0p1
# Find major and minor number for zvol and partition
lsblk -o name,fstype,fssize,maj:min,model,serial
# Enable the LXC container to access the zvol block devices
nano /etc/pve/lxc/<CT_ID>.conf
# Add the following lines
# lxc.cgroup2.devices.allow: b <MAJ>:* rwm
# mp0: /dev/zd0p1,mp=/srv/dev-sda1
# mp1: /<ZFS_POOL_NAME>,mp=/srv/dev-sda1/<ZFS_POOL_NAME>
# lxc.hook.autodev: /var/lib/lxc/<CT_ID>/mount-hook.sh
# Create mount hook for LXC container
nano /var/lib/lxc/<CT_ID>/mount-hook.sh
# Add the following lines
# #!/bin/sh
# mknod -m 777 ${LXC_ROOTFS_MOUNT}/dev/sda b <MAJOR> <MINOR>
# mknod -m 777 ${LXC_ROOTFS_MOUNT}/dev/sda1 b <MAJOR> <MINOR>
```

