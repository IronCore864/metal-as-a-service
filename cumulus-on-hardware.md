# 0 How to Install Cumulus on a Switch

There are many ways to install cumulus on a switch, see here for more details:

https://docs.cumulusnetworks.com/display/DOCS/Installation+Management

Of all of them, the easiest way to install just one (or two, or a small number) switch is to use the usb stick way. More details see here:

https://cumulusnetworks.com/cumulus-on-a-stick/

And the detailed steps to do so see here:

https://docs.cumulusnetworks.com/display/DOCS/Installing+a+New+Cumulus+Linux+Image#InstallingaNewCumulusLinuxImage-usb

# 1 Installing Cumulus on the Switch with the USB Drive

- Format a USB stick to MS-DOS FAT32
- Unzip the file, select the contents of the expanded ZIP file and drag them to your USB thumb drive
- Plugin the USB stick in the front panel of a new switch and power-on

The installation goes automatically and without any user interaction

# 2 Trouble Shooting

Normally the installation goes well fully automatic.

To see the installation process and for trouble shooting, you need to access the switch console.

To do so, you need:

- RS-232 - Serial cable, which is included in the switch most likely
- serial - usb adapter, which is not included
- a terminal app, for example, zterm

Note: in the terminal app, you need to set the speed and other configs according to the switch manual.

# 3 Login

After installation, plugin the switch management port, and you can ssh login to the Cumulus Linux.

Default user/pwd: cumulus/CumulusLinux!

# 4 Configuration

https://docs.cumulusnetworks.com/display/DOCS/Quick+Start+Guide

Example:

```
net add vlan 10-20
net add int swp1-6 bridge access 10
net add int swp17-22 bridge access 20
net add vlan 10 ip address 10.1.1.1/24
net add vlan 20 ip address 10.1.2.1/24
net pending
net commit
net show interface
net show bridge macs
```

```
vim /etc/dhcp/dhcpd.conf
```

Set:

```
ddns-update-style none;
  
default-lease-time 600;
max-lease-time 7200;
 
subnet 10.1.1.0 netmask 255.255.255.0 {
}
subnet 10.1.1.0 netmask 255.255.255.0 {
        range 10.1.1.1 10.1.1.16;
}
 
subnet 10.1.2.0 netmask 255.255.255.0 {
}
subnet 10.1.2.0 netmask 255.255.255.0 {
        range 10.1.2.17 10.1.2.32;
}
```

```
vim /etc/default/isc-dhcp-server
```

Set:

```
DHCPD_CONF="-cf /etc/dhcp/dhcpd.conf"
INTERFACES="swp1 swp2 swp3 swp4 swp5 swp6 swp17 swp18 swp19 swp20 swp21 swp22"
```
