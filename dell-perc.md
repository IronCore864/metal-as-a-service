https://www.dell.com/support/home/us/en/04/drivers/driversdetails?driverid=f48c2

Download and install:

```
wget https://downloads.dell.com/FOLDER04470715M/1/perccli_7.1-007.0127_linux.tar.gz .
mkdir perc
tar zxf perccli_7.1-007.0127_linux.tar.gz -C ./perc
sudo apt-get install alien
sudo alien ./perc/Linux/perccli-007.0127.0000.0000-1.noarch.rpm
sudo dpkg -i ./perccli_007.0127.0000.0000-2_all.deb
rm -rf perc perccli_007.0127.0000.0000-2_all.deb
```

Run:

```
cd /opt/MegaRAID/perccli && sudo ./perccli show
```

Add drive:

```
sudo ./perccli /c0 add vd type=raid0 size=all names=vdisk1 drives=64:0-5
```
