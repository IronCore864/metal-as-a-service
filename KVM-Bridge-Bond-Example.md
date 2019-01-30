# 1 Network Config

For this network config to work, we need to ensure that the correct kernel module bonding/802.1q are present, and loaded at boot time:

Edit your `/etc/modules` configuration:

`sudo vi /etc/modules`

Ensure that the `bonding/8021q` module are loaded:

```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
bonding
8021q
```
Ensure that your network is brought down:

`sudo stop networking`

Then load the bonding kernel module:

```
sudo modprobe bonding
sudo modprobe 8021q
```

Then do the network config:

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).
 
source /etc/network/interfaces.d/*
 
# The loopback network interface
auto lo
iface lo inet loopback
 
# br1 setup with static wan IPv4 with ISP router as a default gateway
auto br1
iface br1 inet manual
    bridge_ports enp88s0f0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
 
auto bond0
iface bond0 inet manual
    bond-lacp-rate 1
    post-up ifenslave bond0 enp134s0f0 enp134s0f1
    pre-down ifenslave -d bond0 enp134s0f0 enp134s0f1
    bond-slaves none
    bond-mode 4
    bond-lacp-rate fast
    bond-miimon 100
    bond-downdelay 0
    bond-updelay 0
    bond-xmit_hash_policy 1
 
auto enp134s0f0
iface enp134s0f0 inet manual
bond-master bond0
 
auto enp134s0f1
iface enp134s0f1 inet manual
bond-master bond0
 
auto br0
iface br0 inet static
    address 172.16.2.3
    netmask 255.255.255.0
    gateway 172.16.2.1
    broadcast 172.16.2.255
    dns-nameservers 8.8.8.8 8.8.4.4 172.16.2.1
    # set static route for LAN
    post-up route add -net 172.16.2.0 netmask 255.255.255.0 gw 172.16.2.1
    up ifconfig $IFACE promisc
    bridge_ports bond0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
    down ifconfig $IFACE promisc
```

In this example, br1 is anonymous, 1 NIC, connected to outbound router; br0 is a bond of 2 NICs.

Finally, bring up your network again:

`sudo start networking`

Then update `/etc/sshd/sshd_config` to listen to br0 IP only.

# 2 KVM Installation

Steps see: https://help.ubuntu.com/community/KVM/Installation

Package installation might have some dependency version issue, need some uninstall existing version and reinstall.

# 3 VM Creation

Here we create 3 VM:

## 1 opnsense vm, firewall, WAN to br1, LAN to br0

```
sudo virt-install \
--virt-type=kvm \
--name opnsense \
--ram 4096 \
--vcpus=4 \
--os-variant=ubuntu16.04 \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/OPNsense-18.7-OpenSSL-dvd-amd64.iso \
--network=bridge=br0,model=virtio \
--network=bridge=br1,model=virtio \
--disk path=/var/lib/libvirt/images/opnsense.qcow2,size=40,bus=virtio,format=qcow2 \
--graphics vnc
```

## 2 bootstrap server vm, br0

```
sudo virt-install \
--virt-type=kvm \
--name rackhd \
--ram 4096 \
--vcpus=4 \
--os-variant=ubuntu16.04 \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/ubuntu-16.04-server-amd64.iso \
--network=bridge=br0,model=virtio \
--disk path=/var/lib/libvirt/images/rackhd.qcow2,size=40,bus=virtio,format=qcow2 \
--graphics vnc
```
## 3 dns vm, br0 and br1 (or NAT)

```
sudo virt-install \
--virt-type=kvm \
--name infra \
--ram 4096 \
--vcpus=4 \
--os-variant=ubuntu16.04 \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/ubuntu-16.04-server-amd64.iso \
--network=bridge=br0,model=virtio \
--network=bridge=br1,model=virtio \
--disk path=/var/lib/libvirt/images/infra.qcow2,size=40,bus=virtio,format=qcow2 \
--graphics vnc
```

# 4 VM Config

Need SSH portforwarding 5900/5901/5902 in order to access the VNC, do it like:

`ssh ubuntu@172.16.2.3(infra server IP) -L 5900:127.0.0.1:5900`
