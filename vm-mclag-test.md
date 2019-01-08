# 1. Server VMs

Download ubuntu 18.04 server iso

Create 2 VMs with default settings.

Check:

`VBoxManage list vms`

Should have server1, server2 in the result.

# 2. Config Server VM NIC

```
VBoxManage modifyvm server1 --nic2 intnet
VBoxManage modifyvm server1 --nic3 intnet
VBoxManage modifyvm server1 --nic4 intnet
VBoxManage modifyvm server1 --nic5 intnet

VBoxManage modifyvm server2 --nic2 intnet
VBoxManage modifyvm server2 --nic3 intnet
VBoxManage modifyvm server2 --nic4 intnet
VBoxManage modifyvm server2 --nic5 intnet

VBoxManage modifyvm server1 --intnet2 s1swp1
VBoxManage modifyvm server1 --intnet3 s2swp1
VBoxManage modifyvm server1 --intnet4 s1swp2
VBoxManage modifyvm server1 --intnet5 s2swp2

VBoxManage modifyvm server2 --intnet2 s1swp3
VBoxManage modifyvm server2 --intnet3 s2swp3
VBoxManage modifyvm server2 --intnet4 s1swp4
VBoxManage modifyvm server2 --intnet5 s2swp4
```

Check:

```
VBoxManage showvminfo server1 | grep NIC | grep MAC
VBoxManage showvminfo server2 | grep NIC | grep MAC
```

Should have 4 Internal Network for each server.

Port forwarding:

```
VBoxManage modifyvm server1 --natpf1 "guestssh,tcp,,2223,,22"
VBoxManage modifyvm server2 --natpf1 "guestssh,tcp,,2224,,22"
```

Boot the servers and install ubuntu server 18.04, all default settings, with names "server1" and "server2", and username "ubuntu".

# 3. Switch VMs

Download Cumulus VX image.

https://cumulusnetworks.com/products/cumulus-vx/download/

Here VirtualBox is used. 

Create a Cumulus VX VM with VirtualBox:

- Open VirtualBox and click File > Import Appliance.
- Browse for the downloaded VirtualBox image, click the Open button, then click Continue.
- Review the Appliance settings. Change the name of the VM to switch1, then click Import to begin the import process.
- Make sure the new VM is named switch1, if not, update settings.
- Right click the created VM switch1, then select Clone.
- Change the name of the VM to switch2, choose reinitialize MAC address, then click Continue. Select Full Clone and click Clone.

Check:

`VBoxManage list vms`

Should have switch1, switch2 in the result.

#4. Config Switch VM NIC

```
VBoxManage modifyvm switch1 --intnet2 s1swp1
VBoxManage modifyvm switch1 --intnet3 s1swp2
VBoxManage modifyvm switch1 --intnet4 s1swp3
VBoxManage modifyvm switch1 --intnet5 s1swp4
VBoxManage modifyvm switch1 --intnet6 peer1
VBoxManage modifyvm switch1 --intnet7 peer2
VBoxManage modifyvm switch1 --nic8 none

VBoxManage modifyvm switch2 --intnet2 s2swp1
VBoxManage modifyvm switch2 --intnet3 s2swp2
VBoxManage modifyvm switch2 --intnet4 s2swp3
VBoxManage modifyvm switch2 --intnet5 s2swp4
VBoxManage modifyvm switch2 --intnet6 peer1
VBoxManage modifyvm switch2 --intnet7 peer2
VBoxManage modifyvm switch2 --nic8 none
```

Check:

```
VBoxManage showvminfo switch1 | grep NIC | grep MAC
VBoxManage showvminfo switch2 | grep NIC | grep MAC
```

Should have 6 Internal Network NIC with the corresponding names.

Port forwarding:

```
VBoxManage modifyvm switch1 --natpf1 "guestssh,tcp,,2221,,22"
VBoxManage modifyvm switch2 --natpf1 "guestssh,tcp,,2222,,22"
```

# 5. Start All Servers and configure SSH/SUDO

```
VBoxManage startvm switch1 --type=headless
VBoxManage startvm switch2 --type=headless
VBoxManage startvm server1 --type=headless
VBoxManage startvm server2 --type=headless

ssh-copy-id -p 2221 cumulus@localhost
ssh-copy-id -p 2222 cumulus@localhost
(PWD CumulusLinux!)
```

Login to all hosts and do:

`sudo vi /etc/sudoers`

Make sure `%sudo   ALL=(ALL:ALL) NOPASSWD:ALL` exists, for password-less sudo.

# 6. Config Switch VM FRR Settings

Do it on both switch1 and switch2:

Cumulus VX login:

username: cumulus

password: CumulusLinux!

Use sudo edit `/etc/frr/daemons`, set:

```
zebra=yes
bgpd=yes
ospfd=yes
```

Then:

`sudo systemctl restart frr.service`

# 7. LLDPD for servers

On server1 and server2:

```
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu bionic main universe"
sudo apt update
sudo apt install -y lldpd
```

# 8. Using LLDP to disover which NIC should be a bond.

Generally, for each server, we need to find two NIC, one is connected to switch1, and the other is connected to switch2, and then configure them as a LAG.

On switch, same story, we need one port on switch1, another port on switch2, which are connected to the same server, then configure it as a bond and add VLAN access for it.

On server:

`sudo lldpcli show neighbor -f keyvalue | grep -E "chassis\.name|port\.ifname"`

```
lldp.enp0s8.chassis.name=switch1
lldp.enp0s8.port.ifname=swp4
lldp.enp0s9.chassis.name=switch2
lldp.enp0s9.port.ifname=swp4
lldp.enp0s10.chassis.name=switch1
lldp.enp0s10.port.ifname=swp3
lldp.enp0s16.chassis.name=switch2
lldp.enp0s16.port.ifname=swp3
```

```
sudo apt -y install ifupdown2
sudo apt -y purge netplan.io
sudo rm -vfr /usr/share/netplan /etc/netplan
sudo modprobe bonding
```

`sudo vim /etc/modules`

add:

```
bonding
loop
lp
```

switch1:

`net add clag peer sys-mac 44:38:39:FF:01:01 interface swp5-6 primary backup-ip 10.0.0.2`

switch2:

`net add clag peer sys-mac 44:38:39:FF:01:01 interface swp5-6 secondary backup-ip 10.0.0.1`

Both:

```
net add vlan 10-20
net add clag port bond svc-svr-1 interface swp1 clag-id 1
net add clag port bond sto-svr-1 interface swp2 clag-id 2
net add clag port bond svc-svr-2 interface swp3 clag-id 3
net add clag port bond sto-svr-2 interface swp4 clag-id 4
net add bond svc-svr-1 bridge access 10
net add bond svc-svr-2 bridge access 10
net add bond sto-svr-1 bridge access 20
net add bond sto-svr-2 bridge access 20
net pending
net commit
```

Test: ping each other.
