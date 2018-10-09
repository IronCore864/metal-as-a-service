https://github.com/puppetlabs/razor-server

Razor is an advanced provisioning application which can deploy both bare-metal and virtual systems. It's aimed at solving the problem of how to bring new metal into a state where your existing DevOps/configuration management workflows can take it over.

Installation:

https://github.com/puppetlabs/razor-server/wiki/Installation

Server:

```
sudo apt-get update
sudo apt-get install -y postgresql postgresql-contrib openjdk-8-jdk
 
sudo -i -u postgres
createuser -P razor
createdb -O razor razor_prd
psql -h localhost -l -U razor razor_prd
 
wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
sudo dpkg -i puppetlabs-release-pc1-xenial.deb
sudo apt-get update
sudo apt-get install -y razor-server
 
sudo vim /etc/puppetlabs/razor-server/config.yaml
(update pwd for db_
 
sudo su - razor
razor-admin -e production migrate-database
sudo service razor-server start
```

Install the Microkernel, check the above installation link.

```
sudo apt-get install -y dhcpd dnsmasq
```

Then setup PXE, also according to the above installation link.

Client:

```
sudo gem install razor-client
```

Commands and VM test:

https://github.com/initcron-devops/razor
