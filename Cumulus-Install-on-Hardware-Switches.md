# 0 Different Approaches to Install

There are many ways to install cumulus on a switch, if you want to know all the ways, see here for more details, but you don't have to:

https://docs.cumulusnetworks.com/display/DOCS/Installation+Management

## 1 ONIE

Of all of them, to install multiple instances, it's better to use ONIE boot.

Using ONIE boot is simple and straight-forward:

### 1 Download

First, download binary from Cumulus: https://cumulusnetworks.com/downloads/

Choose desired version, x86 CPU and Broadcom SOC, for both Dell S5048F-ON and Dell S3048-ON.

### 2 Upload to TFTP Server

Rename the binary installer to `onie-installer` and put it under the root folder of the tftp.

Since we are using RackHD and it has already a tftp, we can put the installer under `path-to-on-tftp/static/tftp/`. For example:

```
ubuntu@rackhd:~/src/on-tftp/static/tftp$ pwd
/home/ubuntu/src/on-tftp/static/tftp
ubuntu@rackhd:~/src/on-tftp/static/tftp$ ls onie-installer
onie-installer
```

### 3 Install License

Copy and paste the license key into the cl-license command:

```
cumulus@switch:~$ sudo cl-license -i
<paste license key>
^+d
```

You need to get at least a trial license key.

## 2 USB Stick

The easiest way to install just a small number of switches manually is to use the usb stick.

More details see here, again, you don't have to: https://cumulusnetworks.com/cumulus-on-a-stick/

If you want to follow the original detailed steps, see this page:

https://docs.cumulusnetworks.com/display/DOCS/Installing+a+New+Cumulus+Linux+Image#InstallingaNewCumulusLinuxImage-usb.

Again this is not mandatory; following this page should be enough.

### 1 Prepare a USB Drive

For installation, we need an empty USB drive.

No particular requirements, size at least 2GB.

Need to format to MS-DOS FAT32, mentioned in below steps as well.

### 2 Getting the USB Disk Content

This needs contact with Cumulus.

### 3 Installing Cumulus on the Switch with the USB Drive

- Format a USB stick to MS-DOS FAT32
- Get the content
- Unzip the file from Attilla de Groot, select the contents of the expanded ZIP file and drag them to your USB thumb drive
- Plugin the USB stick in the front panel of a new switch and power-on

The installation goes automatically and without any user interaction.

### 4 Monitoring the Installation Process

The installation goes fully automatic once you plugged in the USB Drive and powered it on.

To see the installation process, you should use the switch console port. To do so, you need:

- a "RS-232 to Serial" cable, which is included in the switch most likely
- a "serial to usb adapter", which is not included, and you need the driver for that
- a terminal app, for example, zterm on mac os, or just regular terminal.

Note: in the terminal app, you need to set the speed and other configs according to the switch manual. For example, in S5048-ON installation guide:

https://www.dell.com/support/manuals/de/de/debsdt1/networking-s5048f-on/s5048f-on_install_pub/rs-232-console-port-access?guid=guid-a0455eff-cff3-42bf-9f64-4a2e562b88e9&lang=en-us

And here is S3048-ON:

https://www.dell.com/support/manuals/de/de/debsdt1/force10-s3048-on/s3048_on_install_pub/rs-232-console-port-access?guid=guid-bd85ebcb-1072-4b6b-aec2-025dba1504c3&lang=en-us

Set terminal settings according to the manual.

# 5 Login

After installation, you can ssh login to the Cumulus Linux via the management port.

Default user/pwd:

cumulus/CumulusLinux!

In case you don't have all the tools (described in step 3) that are required to use the console port, you can simply wait for a couple of minutes after powering it on, then plug in RJ45 to the management port, to one of your existing network that has DHCP. Go to DHCP and see the new IP address that the switch got, then you can access remotely via the management port SSH login as well.

After installation is successful, repeat the process for all of our switches (2xS5048, 3xS3048).

# 6 Example Config

```
# s3048-1, mgmt 192.168.1.135
net add clag peer sys-mac 44:38:39:FF:01:01 interface swp51-52 primary backup-ip 192.168.1.136
net add vlan 20
# following commands probably should be done by ansible for all the hosts
net add clag port bond bond-to-ctl1 interface swp49 clag-id 49
net add bond bond-to-ctl1 bridge access 20
# for testing rackhd/dhcp/pxe without bond
net add int swp1-3 bridge access 20
net commit
 
# s3048-2, mgmt 192.168.1.136
net add clag peer sys-mac 44:38:39:FF:01:01 interface swp51-52 secondary backup-ip 192.168.1.135
net add vlan 20
# following commands probably should be done by ansible for all the hosts
net add clag port bond bond-to-ctl1 interface swp49 clag-id 49
net add bond bond-to-ctl1 bridge access 20
net commit

# northbound
net add int swp1
net add int swp48
net add vlan 2
net add int swp1 bridge access 2
net add int swp48 bridge access 2
net pending
net commit
```
