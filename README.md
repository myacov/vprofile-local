# vprofile-local
## Objective: 

![Project diagram](images/proj2.jpg)
## Desired Learning outcomes


## The tools (the stack)
| CHOSEN TOOL  | USE | WHY |
| ------------- | ------------- | ------------- |
| 🧊 VirtualBox  | Virtualisation   | Easy to use  |
| **v** Vagrant  | Automation   | Lightweight  |
| 🐧 Linux (Ubuntu) | OS | popular Linux distribution |
| 🐧 Linux (centos9) | OS | popular Linux distribution |
|  Nginx | Load Balancer  | Web Service |
|  Apache tomcat | Application Server  | popular for java apps |
|  RabbitMQ | Broker/Queuing Agent  |  |
|  Memcached | DB Caching  |  |
|  MySQL Server | SQL Database  |  |



## Prerequisites:
-	Computer with 8GB RAM and approximately 10GB free disk space, (for Linux in a virtual machine).
- Oracle VM Virtualbox 
- Vagrant 
- Vagrant plugin - vagrant-hostmanager
   > adds host name and an ip addres from the vagrant file to every VM hostfile (/etc/host)
- Git bash

## Creating the virtual machines using Vagrantfile
1. Clone source code.
2. Bring up the virtual machines
    ```bash
    vagrant up
    ```

## Setup VMs
### MYSQL Setup
1. Login to the db vm
    ```bash
    vagrant ssh db01
    ```
2. Verify Hosts entry, if entries missing update the it with IP and hostnames
    ```bash
    cat /etc/hosts
    ```
3. Update OS with latest patches
    ```bash
    sudo yum update -y
    ```
4. Set Repository
    ```bash
    sudo yum install epel-release -y
    ```
5. Install Maria DB Package
    ```bash
    sudo yum install git mariadb-server -y
    ```
6. Starting & enabling mariadb-server
    ```bash
    sudo systemctl start mariadb
    sudo systemctl enable mariadb
    ```
    > to check if active we can run : sudo systemctl status mariadb
7. RUN mysql secure installation script
    ```bash
    sudo mysql_secure_installation
    ```
    > NOTE: Set db root password
### Setup Database name and users.
1. log in to the db
    ```bash
    mysql -u root -padmin123
    ```
2. create database: "accounts"
    ```sql
    create database accounts;
    ```
3.  grant privileges to user admin (% = remote)
    ```sql
    grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123' ;
    ```
4. reload:
    ```sql
    FLUSH PRIVILEGES;
    ```
5. Exit
    ```sql
    exit;
    ```
### Download Source code & Initialize Database.
```bash
sudo -i
git clone -b main https://github.com/myacov/vprofile-local.git
cd vprofile-local
mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
mysql -u root -padmin123 accounts
```
sql>
    ```
    show tables;
    ```

### Restart mariadb-server
```bash
systemctl restart mariadb
```
### Memcache Setup
1. Login to the mc01 vm
    ```bash
    vagrant ssh mc01
    ```
2. Install, start & enable memcache on port 11211
```bash
sudo -i
dnf install epel-release -y
dnf install memcached -y
systemctl start memcached
systemctl enable memcached
systemctl status memcached
```
> allow to listen for connection from different VM (for tomcat)
```bash 
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
``` 
```bash 
sudo systemctl restart memcached
``` 

### RabbitMQ Setup
1. Login to the RabbitMQ vm
```bash
    vagrant ssh rmq01
```
2. Update OS and Set EPEL Repository
```bash
    sudo -i
    yum update -y
    yum install epel-release -y
```
3. Install Dependencies and RabbitMQ repository
 ```bash
    dnf -y install centos-release-rabbitmq-38
```
4. Enable RabbitMQ repositry and instll rabbitMQ Server
```bash
    dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
```
5. Start and Enable RabbitMQ Server
```bash
    systemctl start rabbitmq-server
    systemctl enable rabbitmq-server
    systemctl status rabbitmq-server
```
6. Specifically for Vprofile : create file and redirect output 
 ```bash
    sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
```
7. RabbitMQ commands - add user and set "administrator" tag . then restart the service
 ```bash
    rabbitmqctl add_user test test
    rabbitmqctl set_user_tags test administrator
    systemctl restart rabbitmq-server
```
### Tomcat Setup
1. Setting up Tomcat Service
    1. Login to the Tomcat vm
    ```bash
        vagrant ssh app01
    ```
    2. Update OS
    ```bash
        sudo yum update -y
    ```
    3. Set Repository
    ```bash
        sudo yum install epel-release -y
    ```
    4. Install Dependencies : openjdk11 git, wget and maven
    ```bash
        dnf -y install java-11-openjdk java-11-openjdk-devel
        dnf install git maven wget -y
    ```
    5. Download & Tomcat Package
    ```bash
        cd /tmp/
        wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
        tar xzvf apache-tomcat-9.0.75.tar.gz
        useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
        cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/
        chown -R tomcat.tomcat /usr/local/tomcat
    ```
    6. Setup systemctl command for tomcat
    ```bash
        vi /etc/systemd/system/tomcat.service
    ```
        - copy script:
    ```bash
        [Unit]
        Description=Tomcat
        After=network.target
        [Service]
        User=tomcat
        WorkingDirectory=/usr/local/tomcat
        Environment=JRE_HOME=/usr/lib/jvm/jre
        Environment=JAVA_HOME=/usr/lib/jvm/jre
        Environment=CATALINA_HOME=/usr/local/tomcat
        Environment=CATALINE_BASE=/usr/local/tomcat
        ExecStart=/usr/local/tomcat/bin/catalina.sh run
        ExecStop=/usr/local/tomcat/bin/shutdown.sh
        SyslogIdentifier=tomcat-%i
        [Install]
        WantedBy=multi-user.target
    ```
    7. Reload systemd files then Start & Enable service
    ```bash
        systemctl daemon-reload
        systemctl start tomcat
        systemctl enable tomcat
    ```
2. CODE Build & Deploy application on Tomcat
    1. Download Source code
    ```bash
        git clone -b main https://github.com/myacov/vprofile-local.git
    ```
    2. Build code
    ```bash
        cd vprofile-local
        mvn install
    ```
    2. Deploy artifact
    ```bash
        systemctl stop tomcat
        rm -rf /usr/local/tomcat/webapps/ROOT*
        cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
        systemctl start tomcat
    ```

### Nginx Setup
  1. Login to the Nginx vm
    ```bash
        vagrant ssh web01
    ```
    2. Update OS & install Nginx
    ```bash
        sudo -i
        apt update
        apt upgrade -y
        apt install nginx -y
    ```
    3. Create Nginx conf file with below content
    ```bash
        vi /etc/nginx/sites-available/vproapp
    ```
    4. Remove default nginx conf
    ```bash
        rm -rf /etc/nginx/sites-enabled/default
    ```
    5. Create link to activate website
    ```bash
        ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
    ```
    6. Restart Nginx
    ```bash
        systemctl restart nginx
    ```



## Automating this process - Provisioning





