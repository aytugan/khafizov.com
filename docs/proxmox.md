# Proxmox Virtual Environment

### Corosync

Corosync is a distributed configuration storage where proxmox stores cluster information

Apply update to `/etc/pve/coronosync.conf` - `corosync-cfgtool -R`

After major changes of `corosync.conf` file `corosync` service must be restarted on all nodes

### Network

Example of Bridges and NAT in network configuration

```
auto lo
iface lo inet loopback

iface enp0s25 inet manual

auto vmbr0
iface vmbr0 inet static
        address 172.16.12.10/24
        gateway 172.16.12.1
        bridge-ports enp0s25
        bridge-stp off
        bridge-fd 0

auto vmbr1
iface vmbr1 inet static
        address 172.16.111.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up -A POSTROUTING -s 172.16.122.0/24 -o vmbr0 -j MASQUERADE
        post-down -A POSTROUTING -s 172.16.122.0/24 -o vmbr0 -j MASQUERADE

```

**Tips:**

- Proxmox node IP address must use IP of internal bridge interface

DHCP server setup - <https://computingforgeeks.com/using-dnsmasq-dhcp-server-proxmox-vms/>

### Commands

- Rescan disks for VM `qm rescan --vmid <VMID>`
- Rescan all disks `qm rescan`

### ZFS Cache

ZFS cache does not counts in Linux cache and can get memory from other apps. 

Show ZFS cache statistics `arcstat` , very detailed stat: `arc_summary`

Limit ZFS cache to 8Gb on the fly - `echo 8589934592 >> /sys/module/zfs/parameters/zfs_arc_max`

Limit ZFS cache permanently - create file `/etc/modprobe.d/zfs`

```
options zfs zfs_arc_max=8589934592
options zfs zfs_arc_min=1073741824
```