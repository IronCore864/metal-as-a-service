# SKU/OBM

First, create SKU, to automate BMC/OBM settings for different brands.

```
# get all catalogs
curl -s -X GET 192.168.1.2:8080/api/current/nodes/<node_id_here>/catalogs
 
# in the result, find Manufacturer
 
# Create a SKU to Auto-Configure IPMI Settings
curl -X POST -H 'Content-Type: application/json' -d @default_ipmi_lenovo.json 192.168.1.2:8080/api/current/skus
 
# content of the payload default_ipmi_lenovo.json:
{
    "name": "Default IPMI settings for Lenovo servers",
    "discoveryGraphName": "Graph.Set.Bmc.Credentials",
    "discoveryGraphOptions": {
        "defaults": {
            "user": "USERID",
            "password": "PASSW0RD"
        }
    },
    "rules": [
        {
            "path": "bmc.IP Address"
        },
        {
            "path": "dmi.Base Board Information.Manufacturer",
            "equals": "Lenovo"
        }
    ]
}
# note that the "name" field can not be duplicated
 
# query all SKUs
curl -s -X GET 192.168.1.2:8080/api/current/skus | jq
```

More info see:

https://rackhd.readthedocs.io/en/latest/rackhd_api/skus.html

# Raid Config

## 2.1 Build docker that contains storcli

Go to `/home/ubuntu/src/on-imagebuilder/oem/raid`

Download storcli_1.17.08_all.deb to this folder.

(User can download it from http://docs.avagotech.com/docs/1.17.08_StorCLI.zip.)

```
sudo docker build -t rackhd/micro --build-arg STORCLI=storcli_1.17.08_all.deb .
sudo docker save rackhd/micro | xz -z > raid.docker.tar.xz
cp raid.docker.tar.xz ~/src/on-http/static/http/common/
# These steps already done
# Issue: maybe need more recent version of storcli. If change file name, needs to update Dockerfile as well.
```

## 2.2 Raid Workflow

```
curl -X POST -H 'Content-Type: application/json' -d @raid_config.json 192.168.1.2:8080/api/current/nodes/<node_id_here>/workflows?name=Graph.Raid.Create.MegaRAID | jq '.'
 
# content of the payload:
{
    "options": {
        "bootstrap-rancher":{
            "dockerFile": "raid.docker.tar.xz"
        },
        "create-raid": {
            "raidList": [
                {
                    "enclosure": 255,
                    "type": "raid1",
                    "drives": [0, 1],
                    "name": "VD0"
                }
            ]
        }
    }
```

Issue: currently, it seems the "drives" parameter did not work for Inspur servers. No matter what raid type or drive number is provided, it always create one virtual disk raid0 on each physical disk. Need to fix this.

More info see:

https://rackhd.readthedocs.io/en/latest/server_workflow/raid_configuration.html

https://github.com/RackHD/on-imagebuilder

Notes: raid config task finishes very fast, so it's not possible to do rancher login then login to docker to see the logs. In order to make this possible, you can edit the Graph.Bootstrap.Megaraid.Configure workflow, and delete the last two tasks. In this way, it won't restart immediately after config.

# OS Install Workflow:

```
curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_iso_minimal.json 192.168.1.2:8080/api/current/nodes/<node_id_here>/workflows?name=Graph.InstallUbuntu | jq '.'
 
# content of the payload:
{
    "options": {
        "defaults": {
            "repo": "{{ file.server }}/ubuntu",
            "version": "xenial",
            "baseUrl": "install/netboot/ubuntu-installer/amd64",
            "kargs": {
                "live-installer/net-image": "{{ file.server }}/ubuntu/install/filesystem.squashfs"
            },
            "installDisk": "/dev/sda"
        }
    }
}
```

Note: when it hanged at "syslogd", press ALT+F4 to switch tty and it continues. Do not switch is also fine, the process goes well on Inspur. After installation it will reboot again. When login to the new OS, if nothing is seen in IPMI, also switch ALT+F4.
