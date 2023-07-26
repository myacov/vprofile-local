# vprofile-local
## Objective: 


## Desired Learning outcomes


## The tools (the stack)
| CHOSEN TOOL  | USE | WHY |
| ------------- | ------------- | ------------- |
| 🧊 VirtualBox  | Virtualisation   | Easy to use  |
| **v** Vagrant  | Automation   | Lightweight  |
| 🐧 Linux (Ubuntu) | OS | popular Linux distribution |
| 🐧 Linux (centos9) | OS | popular Linux distribution |
| 🌏 Nginx | Load Balancer  |  |
| 🌏 Apache tomcat | Web server  | popular web server |
| 🌏 RabbitMQ | message broker  |  |
| 🌏 Memcached | Cache service  |  |
| 🌏 MySQL Server | db server  |  |

![Project diagram](images/proj2.jpg)

## Prerequisites:
-	Computer with 8GB RAM and approximately 10GB free disk space, (for Linux in a virtual machine).
- Oracle VM Virtualbox 
- Vagrant 
- Vagrant plugin - vagrant-hostmanager
   > adds host name and an ip addres from the vagrant file to every VM hostfile (/etc/host)
- Git bash

## Creating a virtual machine using Vagrantfile
1. Clone source code.
2. Bring up the virtual machines
    ```bash
    $ vagrant up
    ```

## Install the web server
using  Ubuntu’s package manager **apt**.
1.  search for Apache HTTP Server:
```bash
apt search apache 
```
2. Before we install Apache, we should do a package update.
```bash
sudo apt update
```
3. Install Apache:
```bash
sudo apt install apache2 -y
```
4.	check the status of the Apache HTTP Server “service”:
```bash
systemctl status apache2
```
5.	make a test request using the curl program. connect to localhost and port 80 (default port for HTTP):
```bash
curl -v localhost:80
```
## Set up networking
To access our web server from outside the VM, we’ll need to give the VM an IP address that we can reach from outside the VM.
### How to see your VM’s network configuration
```bash
ip addr
```

1. Back on the terminal In Vagrantfile, add this line to set the network configuration
 specify an IP address manually :
  ```bash
  config.vm.network "private_network", ip: "192.168.33.10"
  ```
2. reload the configuration and restart the VM:
   ```bash
   vagrant reload
   ```
3. SSH into it again and run ip addr
4. 4.	SSH into it again: vagrant ssh
5.	Now if you re-run the IP addr command:
   ```bash
   ip addr
   ```
6. A new network interface has appeared
## Access the website from a web browser


## Add some website content

### Copy across the new website
1.	Create a directory called tmp underneath the location where you 
mkdir tmp

2.	Inside the tmp directory, download a wesite template.

wget https://www.tooplate.com/zip-templates/2136_kool_form_pack.zip
3.	Unzip the file
unzip 2136_kool_form_pack.zip 
(if needed install unzip: sudo apt install unzip -y)

4.	copy the website files to /var/www/html.:
(the target directory is owned by root.)
sudo cp -r tmp/2136_kool_form_pack/* /var/www/html/
5.	Now when we reload the web browser, the web server now shows our web site, instead of the default page.

## Automating this process - Provisioning





