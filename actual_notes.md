# NSM Engineering Day 1
## Set up Containers
1.  ***on STUDENT machine*** to get containers started: 
    * `lxc list` to list the containers
    * `lxc start --all` to start all containers
> passwords for the container is `training`
3. to disable ipv6, ***vi/vim*** into `/etc/sysctl.conf` and ensure that 1 is next to disable ipv6
1. `ip a` to verify interfaces. Eth0 is management, eth1 is interfaces
* ***SSH INTO EACH CONTAINER*** and configure the following: `sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`
    * ```
        DEVICE=eth0
        BOOTPROTO=none
        ONBOOT=yes
        HOSTNAME=elastic0
        NM_CONTROLLED=no
        TYPE=Ethernet
        IPADDR=10.81.139.30
        GATEWAY=10.81.139.1
        PREFIX=24
        ```
    * ***be sure to change the hostname and the ipaddr for each machine.***
<!--
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.30
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic1
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.31
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic2
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.32
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=kibana
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.50
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=pipeline0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.40
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=pipeline1
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.41
GATEWAY=10.81.139.1
PREFIX=24

i

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=sensor
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.20
GATEWAY=10.81.139.1
PREFIX=24

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=repo
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.10
GATEWAY=10.81.139.1
PREFIX=24
-->
* `sudo vi /etc/hosts`
    * ```
        127.0.0.1 localhost
        10.81.139.10 repo
        10.81.139.20 sensor
        10.81.139.30 elastic0
        10.81.139.31 elastic1
        10.81.139.32 elastic2
        10.81.139.40 pipeline0
        10.81.139.41 pipeline1
        10.81.139.42 pipeline2
        10.81.139.50 kibana
<!--
127.0.0.1 localhost
10.81.139.10 repo
10.81.139.20 sensor
10.81.139.30 elastic0
10.81.139.31 elastic1
10.81.139.32 elastic2
10.81.139.40 pipeline0
10.81.139.41 pipeline1
10.81.139.42 pipeline2
10.81.139.50 kibana

-->
> in vi,`:%d` deletes the whole document
5. `sudo systemctl restart network` for each machine
>to ensure proper copy and paste format, right click for both, and make sure youre in insert mode

6. ***IN STUDENT*** ssh-keygen
    * if it asks to overwrite in /home/ubuntu/.ssh/id_rsa, hit yes
    * use training as key

7. `sudo vi ~/.ssh/config` file, copy and paste this into the file:
    * ```
        Host repo
            HostName repo
            User elastic
        Host sensor
            HostName sensor
            User elastic
        Host elastic0
            HostName elastic0
            User elastic
        Host elastic1
            HostName elastic1
            User elastic
        Host elastic2
            HostName elastic2
            User elastic
        Host pipeline0
            HostName pipeline0
            User elastic
        Host pipeline1
            HostName pipeline1
            User elastic
        Host pipeline2
            HostName pipeline2
            User elastic
        Host kibana
            HostName kibana
            User elastic 

<!--
Host repo
  HostName repo
  User elastic
Host sensor
  HostName sensor
  User elastic
Host elastic0
  HostName elastic0
  User elastic
Host elastic1
  HostName elastic1
  User elastic
Host elastic2
  HostName elastic2
  User elastic
Host pipeline0
  HostName pipeline0
  User elastic
Host pipeline1
  HostName pipeline1
  User elastic
Host pipeline2
  HostName pipeline2
  User elastic
Host kibana
  HostName kibana
  User elastic
  -->
8. to recursively set ssh id key to each host in the container:`for host in sensor repo elastic{0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done`

> if you cannot ssh into any of the containers, restart by doing `lxc restart --all`
## Configure Repo
9. ***on repo***
    1. `sudo yum install nginx` is a place that can store files through a web browser

    1. `unzip all-class-files.zip -d /usr/share/nginx/` 

    1. `sudo mv /usr/share/nginx/all-class-files /usr/share/nginx/fileshare`

    1. `sudo mv emerging.rules.tar.gz /usr/share/nginx/fileshare`

    * `sudo vi /etc/nginx/conf.d/fileshare.conf`
        * `server {
            listen 8000;
            location / {
                root /usr/share/nginx/fileshare;
                autoindex on;
                index index.html index.htm;
            }
        }`
    <!--
    server {
        listen 8000;
        location / {
            root /usr/share/nginx/fileshare;
            autoindex on;
            index index.html index.htm;
            }
        }
    -->
    * `sudo firewall-cmd --add-port=8000/tcp --permanent`
    * `sudo firewall-cmd --reload`
    * `sudo systemctl enable --now nginx`
---
# NSM Engineer Day 2
1. `sudo yum install yum-utils -y`
1. `sudo reposync -l --repoid=extras --download_path=/repo/local-extras`
2. `sudo yum install createrepo`
    * convert rpms into packages repo can use
1. `sudo createrepo local-extras`
    * make sure youre in the repo directory
1. `sudo vi /etc/nginx/conf.d/packages.conf`
    * ```
        server {
            listen 8008;
            location / {
                root /repo;
                autoindex on;
                index index.html index.htm;
            }
        }`

<!--
server {
  listen 8008;
  location / {
    root /repo;
    autoindex on;
    index index.html index.htm;
  }
}
-->
> ss -lnt to verify listening ports

> "^word^ new word" to replace a word in the previous command

6. `sudo firewall-cmd --add-port=8008/tcp --permanent`
1. `sudo firewall-cmd --reload`
1. `sudo systemctl enable --now nginx`
---
### Sensor
* SSH in to Sensor and `mkdir ~/archive`
* `sudo mv /etc/yum.repos.d/* ~/archive`
* `cd` into /etc/yum.repos.d
* `sudo vi local.repo`
    * type in search bar to search and replace. `:%s/<hostname:port>/repo:8008/g` 
    ```
    [local-base]
    name=local-base
    baseurl=https://repo/packages/local-base/
    enabled=1
    gpgcheck=0

    [local-rocknsm-2.5]
    name=local-rocknsm-2.5
    baseurl=https://repo/packages/local-rocknsm-2.5/
    enabled=1
    gpgcheck=0

    [local-elasticsearch-7.x]
    name=local-elasticsearch-7.x
    baseurl=https://repo/packages/local-elastic-7.x/
    enabled=1
    gpgcheck=0

    [local-epel]
    name=local-epel
    baseurl=https://repo/packages/local-epel/
    enabled=1
    gpgcheck=0

    [local-extras]
    name=local-extras
    baseurl=https://repo/packages/local-extras/
    enabled=1
    gpgcheck=0

    [local-updates]
    name=local-updates
    baseurl=https://repo/packages/local-updates/
    enabled=1
    gpgcheck=0


in sensor after configuring local.repo, and creating everything in archive.   
```
scp /etc/yum.repos.d/local.repo ~/archive

for host in elastic{0..2} pipeline{0..2} kibana; do scp ~/archive/* elastic@$host:/home/elastic/ ; done
```
inside each machine: double check with `ll` to ensure all docs to include the local.repo is there

```
mkdir ~/archive; sudo mv /etc/yum.repos.d/* ~/archive; sudo mv local.repo /etc/yum.repos.d; ll ~/archive; ll /etc/yum.repos.d
```

* `for host in elastic{0..2} pipeline{0..2} kibana; do ssh -t elastic@$host 'sudo mkdir ~/archive ; sudo mv /etc/yum.repos.d/* ~/archive/' ; sudo scp /etc/yum.repos.d/local.repo elastic@$host:~/local.repo ; ssh -t elastic@$host 'sudo mv ~/local.repo /etc/yum.repos.d/local.repo ; sudo yum makecache fast' ; ll elastic@$host:~/archive ; ll elastic@$host:/etc/yum.repos.d ; done`

---
### Securing Repo and FileShare
* `mkdir` and `cd` into ~/certs
* `openssl genrsa -des3 -out localCA.key 2048`
    * for the passphrase, use `training`
* `openssl req -x509 -new -nodes -key localCA.key -sha256 -days 1095 -out localCA.crt`
    * for the passphrase, use `training`
    * Country Name: `US`
    * State: `CA`
    * City: `Mountain View`
    * Company: `Elastic`
    * Unit: `Security`
    * Name: `Instructor`
    * Email: `<blank>`
* `openssl genrsa -out repo.key 2048`
* `openssl req -new -key repo.key -out repo.csr`
    * Country Name: `US`
    * State: `CA`
    * City: `Mountain View`
    * Company: `Elastic`
    * Unit: `Security`
    * Name: `Repo`
    * Email: `<blank>`
    * Challenge Key: `<blank>`
    * Opt Co Name: `<blank>`
* `sudo vi ~/certs/repo.ext`
    * ```
        authorityKeyIdentifier=keyid,issuer
        basicConstraints=CA:FALSE
        keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
        subjectAltName = @alt_names

        [alt_names]
        DNS.1 = repo
        IP.1 = 10.81.139.10

<!--
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = repo
IP.1 = 10.81.139.10
-->
* `openssl x509 -req -in repo.csr -CA localCA.crt -CAkey localCA.key -CAcreateserial -out repo.crt -days 365 -sha256 -extfile repo.ext`
    * use the `training` password
* `sudo mv repo.crt /etc/nginx`
* `sudo mv repo.key /etc/nginx`
* `cd /etc/nginx/conf.d` and `ll`
* `sudo curl -LO http://repo:8000/nginx/proxy.conf`
* `sudo vi /etc/nginx/nginx.conf`
    * add "#" to lines 39-41
* `sudo vi /etc/nginx/conf.d/packages.conf`
    * add "127.0.0.1:" in front of "listen" and the port number 
    * do the same for `fileshare.conf`
* `sudo firewall-cmd --add-port={80,443}/tcp --permanent`
* `sudo firewall-cmd --remove-port={8000,8008}/tcp --permanent`
* `sudo firewall-cmd --reload`
* `sudo systemctl restart nginx`
* `^restart^status`
* in vi, edit the following in `/etc/yum.repos.d/local.repo`:
    * `baseurl=https://repo/packages/...`

<!--1.  from sensor, 
    * `for host in pipeline{0..2} kibana; do scp ~/archive/* $host:~ ; done`
        * adds local.repo to home page
    * `for host in pipeline{0..2} kibana; do scp ~/archive/yum.repos.d/* $host:~ ; done`
        * adds all contents of yum.repos.d
2. for each host, `ssh` into each host and perform the following:
    * `sudo mv /etc/yum.repos.d/* ~/archive; sudo mv ~/local.repo /etc/yum.repos.d; ll; ll ~/archive; ll /etc/yum.repos.d ; done`
    * you should have all contents from yum directory in both the `home` and `archive` file. nothing in the yum directory besides the `local.repo` file
---



<!--* `for host in elastic{0..2} pipeline{0..2} kibana; do scp ~/archive elastic@host:/home/elastic ; done`

```
mkdir ~/archive
sudo mv /etc/yum.repos.d/* ~/archive
sudo vi /etc/yum.repos.d/local.repo
    *copy and paste local.repo document from sensor and paste*
```


for host in pipeline{0..2} kibana; do scp ~/archive/* $host:~ ; done
adds local.repo to home page

for host in pipeline{0..2} kibana; do scp ~/archive/yum.repos.d/* $host:~ ; done
adds all contents of yum.repos.d

mkdir ~/archive; sudo mv /etc/yum.repos.d/* ~/archive; sudo mv local.repo /etc/yum.repos.d; ll ~/archive; ll /etc/yum.repos.d
in the host:

for host in <hosts>; do 

on my host computer, i want to copy files in my `~/archive` and `/etc/yum.repos.d` directories to each host. Move the `~/archive` directory and everything in it to each host's `home` directory, and copy the file from my `/etc/yum.repos.d` to the other host's `/etc/yum.repos.d`. it should be 10 files in `archive` and one file in `yum.repos.d`. 

note: each host does not have `~/archive` directory. I'm trying to form a for loop that creates the directory in each host.

what i tried to do below, since each host is the same, is basically redo what i did on the original host, onto the others. Create the archive file, move files from my yum directory into the archive file, copy the local.repo file from the OG computer onto each host, and then display each directory in the host.

```
for host in pipeline{1..2} kibana;
    do mkdir ~/archive ;
    sudo mv $host:/etc/yum.repos.d/* $host:~/archive ; 
    scp /etc/yum.repos.d/local.repo $host:/etc/yum.repos.d ; 
    ll $host ; 
    ll $host:~/archive ; 
    ll $host:/etc/yum.repos.d ; 
done
```

if this worked the way i wanted because i had my files all the way wrong on the sensor. if all of my documents were just in ~/archive instead of ~/archive/yum.repos.d, it wouldve just been

for host in <hosts> ; do scp ~/archive/* elastic@$host:/home/elastic/ ; scp /etc/yum.repos.d/local.repo elastic@$host:/etc/yum.repos.d ; done

the second command, `scp /etc/yum.repos.d/local.repo elastic@$host:/etc/yum.repos.d ` gives back a permission denied.
-->

> sudo systemctl restart nginx

---
* from repo: `for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt ; done`

* in each host:
    * `sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt ; sudo update-ca-trust ; sudo yum makecache fast`
    
* or use in repo `for host in elastic{0..2} pipeline{0..2} kibana sensor; do sudo scp ~/certs/localCA.crt elastic@$host:~/localCA.crt && ssh -t elastic@$host 'sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/ && sudo update-ca-trust'; done`


---
### Configure Capture Interfaces
> eth0 is management and eth1 is going to be our capture interface
-sensor
* sudo yum install ethtool
* sudo ethtool -k eth1
    * -k is the flag to show features
> sudo curl -LO `https://address`
* sudo chmod +x interface.sh
* sudo ./interface.sh 
* sudo vi /sbin/ifup-local
    * 
    ```
    #!/bin/bash
    if [[ "$1" == "eth1" ]]
    then
    for i in rx tx sg tso ufo gso gro lro rxvlan txvlan
    do
    /usr/sbin/ethtool -K $1 $i off
    done
    /usr/sbin/ethtool -N $1 rx-flow-hash udp4 sdfn
    /usr/sbin/ethtool -N $1 rx-flow-hash udp6 sdfn
    /usr/sbin/ethtool -n $1 rx-flow-hash udp6
    /usr/sbin/ethtool -n $1 rx-flow-hash udp4
    /usr/sbin/ethtool -C $1 rx-usecs 10
    /usr/sbin/ethtool -C $1 adaptive-rx off
    /usr/sbin/ethtool -G $1 rx 4096

    /usr/sbin/ip link set dev $1 promisc on

    fi
    ```
* sudo chmod +x /sbin/ifup-local
* sudo vi /etc/sysconfig/network-scripts/ifup
    * type the following: 
    ``` 
    if [ -x /sbin/ifup-local ]; then 
    /sbin/ifup-local ${DEVICE}
    fi
    ```
<!--
if [ -x /sbin/ifup-local ]; then 
/sbin/ifup-local ${DEVICE}
fi
-->
* sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1
    * type the following: 
        ```
        DEVICE=eth1
        BOOTPROTO=none
        ONBOOT=yes
        NM CONTROLLED=no
        TYPE=Ethernet
        ```
<!--
DEVICE=eth1
BOOTPROTO=none
ONBOOT=yes
NM CONTROLLED=no
TYPE=Ethernet
--->
* sudo tcpdump -nn -i eth1 '!port 22'
* sudo yum install stenographer
* cd /etc/stenographer or sudo vi /etc/stenographer/config
    * ```
        {
             "Threads": [
                { "PacketsDirectory": "/data/stenographer/packets"
                , "IndexDirectory": "/data/stenographer/index"
                , "MaxDirectoryFiles": 30000
                , "DiskFreePercentage": 30
                }
            ]
            , "StenotypePath": "/usr/bin/stenotype"
            , "Interface": "eth1"
            , "Port": 1234
            , "Host": "127.0.0.1"
            , "Flags": []
            , "CertPath": "/etc/stenographer/certs"
        }  
         ```
<!--
{
  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 30
    }
  ]
  , "StenotypePath": "/usr/bin/stenotype"
  , "Interface": "eth1"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}
-->
* sudo mkdir -p /data/stenographer/{packets,index} ; sudo chown -R stenographer:stenographer /data/stenographer ; ll /data/stenographer/
* sudo stenokeys.sh stenographer stenographer ; ll /etc/stenographer/certs/
* sudo systemctl enable stenographer --now ; sudo systemctl status stenographer
* ping 8.8.8.8 -c 5
* sudo stenoread 'host 8.8.8.8' -nn
* cd /data/stenographer/packets
* watch ls -al /data/stenographer/packets/
* sudo yum install suricata
sudo -s
* in /etc/suricata/suricata.yaml

/etc/stenographer/config
```
line 56, data/suricata; 
60, no; 
76, no; 
404 no, 
557, no; 
580, eth1; 
582, remove # and change threads to 3; 
981-3, remove # and change user and group to suricata; 
1500; enabled:no; 
1516 = no; 
1521 enabvbled: no; 
1527 = no; 
1536 = = no
1434: set cpu affinity = yes
1452 : worker cpu set cpu: 0-2
1459-61: priority medium = 1 (no quotes) and high = 2, default = "high"
```
in etc/sysconfig/suricata 
```
# The following parameters are the most commonly needed to configure
# suricata. A full list can be seen by running /sbin/suricata --help
# -i <network interface device>
# --user <acct name>
# --group <group name>

# Add options to be passed to the daemon
OPTIONS="--af-packet=eth1 --user suricata --group suricata "
```
sudo suricata-update add-source emergingthreats https://repo/fileshare/emerging.rules.tar.gz

sudo mkdir -p /data/suricata ; sudo chown -R suricata:suricata /data/suricata ; sudo systemctl enable suricata --now


---
# SET UP GUIDE FROM INSTRUCTORS
## Configure environment
2. On the student laptop, view the status of the containers:
1. lxc list
1. If the containers are “stopped”, then start the containers:
1. `lxc start --all`
1. Configure eth0
1. ssh <node>
1. sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
1. Make sure you are changing the host name and IP address
1. Edit the hosts files
1. `sudo vi /etc/hosts`
1. Restart the network `sudo systemctl restart network`
1. `sudo systemctl restart network`
1. On the student laptop, list the containers to see the static IP
1. `lxc list`
1. REPEAT FOR EACH CONTAINER EXCEPT repo
1. SSH to each container without the password
1. FROM THE STUDENT LAPTOP- we already generated the key!
1. `for host in sensor repo elastic{0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done`
1. LOCAL REPO
1. The repo and packages are already secure. Each container needs to use
1. local.repo to grab our local repositories
1. ssh <container>
1. Make an archive directory and move the default repos to this directory
1. Create local.repo in /etc/yum.repos.d
1. *Make sure this is the version that has a base url of https
1. FROM THE REPO
1. Copy the CA root certificate to all the containers
1. for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt ; done
1. BACK ON THE CONTAINER
1. Move localCA.crt to the trust anchors directory
1. sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt
1. Update the CA trust
1. sudo update-ca-trust
1. The package repository should now properly respond to yum from all kit boxes because they can all validate the repo certificate against the CA. Verify this with:
1. sudo yum makecache fast
1. NOW YOU CAN CONFIGURE THE CAPTURE INTERFACE
---
in sensor
1. sudo yum install zeek
75. sudo yum install zeek-plugin-af packet
76. sudo yum install zeek-plugin-af_packet
77. sudo yum install zeek-plugin-kafka
78. ll /etc/zeek
79. cat /etc/zeek/networks.cfg 
80. sudo vi /etc/zeek/zeekctl.cfg 
    * line 67 /data/zeek
    * add: lb_custom.InterfacePrefix=af_packet:: (be easier to do right under 67)
1. sudo vi /etc/zeek/neode.cfg
    * add a # to lines 8-11
    * 16-31 uncomment by deleting #
    * under manager, add the line "pin_cpus=1"
    * change interface under worker 1 to eth1
    * under worker 1: add "lb_method=custom", "lb_procs=2", "pin_cpus=2,3", env_vars=fanout_id=77"
<!--
lb_method=custom
lb_procs=2
pin_cpus=2,3
env_vars=fanout_id=77
-->

* sudo vi /etc/zeek
* sudo vi /etc/zeek/node.cfg 
* sudo mkdir /usr/share/zeek/site/scripts
* cd /usr/share/zeek/site/scripts/
* url_base="https://repo/fileshare/zeek/"; files=("afpacket.zeek" "extension.zeek" "extract-files.zeek" "fsf.zeek" "json.zeek" "kafka.zeek"); for file in "${files[@]}"; do sudo curl -LO "${url_base}${file}"; done
* cd /usr/share/zeek/site

* add to local.zeek in /usr/zeek/site
    ```
    @load ./scripts/afpacket.zeek
    @load ./scripts/extension.zeek
    redef ignore_checksums = T;


* sudo mkdir -p /data/zeek
* sudo chown -R zeek: /data/zeek
* sudo chown -R zeek: /etc/zeek
* sudo chown -R zeek: /usr/share/zeek
* sudo chown -R zeek: /usr/bin/zeek
* sudo chown -R zeek: /usr/bin/capstats
* sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek
* sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats
* sudo chown -R zeek: /var/spool/zeek
* sudo getcap /usr/bin/zeek
* sudo getcap /usr/bin/capstats
* sudo -u zeek zeekctl deploy
* sudo -u zeek zeekctl status
* cd /data/zeek/current/ ; ll
sudo -u zeek zeekctl start
^start^stop
^stop^status
sudo yum install fsf
* sudo vi /opt/fsf/fsf-server/conf/config.py
    ```
    #!/usr/bin/env python
    #
    # Basic configuration attributes for scanner. Used as default
    # unless the user overrides them.
    #

    import socket

    SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf',
                    'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
                    'PID_PATH' : '/run/fsf/fsf.pid',
                    'EXPORT_PATH' : '/data/fsf/archive',
                    'TIMEOUT' : 60,
                    'MAX_DEPTH' : 10,
                    'ACTIVE_LOGGING_MODULES' : ['rockout'],
                    }

    SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
                    'PORT' : 5800 }
<!--
#!/usr/bin/env python
#
# Basic configuration attributes for scanner. Used as default
# unless the user overrides them.
#

import socket

SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf',
                   'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
                   'PID_PATH' : '/run/fsf/fsf.pid',
                   'EXPORT_PATH' : '/data/fsf/archive',
                   'TIMEOUT' : 60,
                   'MAX_DEPTH' : 10,
                   'ACTIVE_LOGGING_MODULES' : ['rockout'],
                   }

SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
                  'PORT' : 5800 }
-->
> if you need to change the server config, make sure to change the ownership again and restart by doing `sudo chown -R fsf: /data/fsf ; sudo systemctl restart fsf`
* sudo mkdir -p /data/fsf/archive ; sudo chown -R fsf: /data/fsf
sudo systemctl enable fsf --now
sudo vi into 
@load ./scripts/json.zeek
@load ./scripts/extract-files.zeek
@load ./scripts/fsf.zeek
@load ./scripts/kafka.zeek
---
* in pipeline0, 1, 2
- sudo yum install kafka zookeeper
- sudo mkdir -p /data/zookeeper ; sudo chown -R zookeeper: /data/zookeeper
- sudo vi /etc/zookeeper/zoo.cfg
    - ```
        # where zookeeper will store its data
        dataDir=/data/zookeeper

        # what port should clients like kafka connect on
        clientPort=2181

        # how many clients should be allowed to connect, 0 = unlimited
        maxClientCnxns=0

        # list of zookeeper nodes to make up the cluster
        # First port is how followers and leaders communicate
        # Second port is used during the election process to determine a leader
        server.1=pipeline0:2888:3888
        server.2=pipeline1:2888:3888
        server.3=pipeline2:2888:3888

        # more than one zookeeper node will have a unique server id.
        # Ex.) server.1, server.2, etc..

        # milliseconds in which zookeeper should consider a single tick
        tickTime=2000

        # amount of ticks a follow has to connect and sync with the leader
        initLimit=5

        # amount of ticks a follower has to sync with a leader before being dropped
        syncLimit=2
        ```
<!--
# where zookeeper will store its data
 dataDir=/data/zookeeper

 # what port should clients like kafka connect on
 clientPort=2181

 # how many clients should be allowed to connect, 0 = unlimited
 maxClientCnxns=0

 # list of zookeeper nodes to make up the cluster
 # First port is how followers and leaders communicate
 # Second port is used during the election process to determine a leader
 server.1=pipeline0:2888:3888
 server.2=pipeline1:2888:3888
 server.3=pipeline2:2888:3888

 # more than one zookeeper node will have a unique server id.
 # Ex.) server.1, server.2, etc..

 # milliseconds in which zookeeper should consider a single tick
 tickTime=2000

 # amount of ticks a follow has to connect and sync with the leader
 initLimit=5

 # amount of ticks a follower has to sync with a leader before being dropped
 syncLimit=2
--->
- sudo touch /data/zookeeper/myid ; sudo chown zookeeper: /data/zookeeper/myid
- echo "#" | sudo tee /data/zookeeper/myid ; cat /data/zookeeper/myid
    | pipeline0 | 1 |
    |---|---|
    | pipeline1 | 2 |
    | pipeline2 | 3 |
- sudo firewall-cmd --add-port={2181,2888,3888}/tcp --permanent
- sudo firewall-cmd --reload

***do not start zookeeper yet. must repeat the above on  the other machines***
- sudo systemctl enable zookeeper --now
- sudo systemctl status zookeeper
on student box
- for host in pipeline{0..2}; do (echo "stats" | nc $host 2181 -q 2); done
- sudo mkdir -p /data/kafka ;sudo chown kafka: /data/kafka
- sudo cp /etc/kafka/server{.properties,.properties.bk}    
- sudo vi /etc/kafka/server.properties 
    - ```
        # The unique id of this broker should be different for each kafka node. Good practice is to match the kafka broker id to the zookeeper server id.
        broker.id=0

        # the port in wich kafka should use to communicate with other kafka clients
        port=9092
        # the hostname or IP address in which the server listens on
        listeners=PLAINTEXT://pipeline0:9092

        # hostname that will be advertised to producers and consumers
        advertised.listeners=PLAINTEXT://pipeline0:9092

        # number of threads used to send network responses
        num.network.threads=3

        # number of threads used to make I/O requests
        num.io.threads=8
        socket.send.buffer.bytes=102400
        socket.receive.buffer.bytes=102400
        socket.request.max.bytes=104857600

        # where kafka should write its data to
        log.dirs=/data/kafka

        # how many partitions and replicas should be generated for topics that are created by other software
        num.partitions=3
        offsets.topic.replication.factor=3
        transaction.state.log.replication.factor=3
        transaction.state.log.min.isr=2
        default.replication.factor = 3
        min.insync.replicas = 2

        # how many threads should be used for shutdown and start up
        num.recovery.threads.per.data.dir=3

        # how long should we retain logs in kafka
        log.retention.hours=12
        log.retention.bytes=90000000000

        # max size of a single log file
        log.segment.bytes=1073741824

        # frequency in miliseconds to check if a log needs to be deleted
        log.retention.check.interval.ms=300000
        log.cleaner.enable=false

        # will not allow a node to be elected leader if it is not in sync with other nodes. Prevents possible missing messages
        unclean.leader.election.enable=false

        # automatically create topics from external software
        auto.create.topics.enable=false


        # how to connect kafka to zookeeper
        zookeeper.connect=pipeline0:2181,pipeline1:2181,pipeline2:2181
        zookeeper.connection.timeout.ms=30000
    - ***be sure to change the following for the other pipelines:***
        | pipeline0 | 0 |
        |---|---|
        | pipeline1 | 1 |
        | pipeline2 | 2 |

        
- sudo firewall-cmd --add-port=9092/tcp --permanent ; 
- sudo firewall-cmd --reload
- sudo systemctl enable kafka --now
- sudo systemctl status kafka
- sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --create --topic test --partitions 3 --replication-factor 3
- sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --list
- sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --describe --topic  test
- to mark topic for deletion
    - sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --delete --topic test
- sudo /usr/share/kafka/bin/kafka-topics.sh --zookeeper pipeline0:2181 --create --topic zeek-raw --partitions 3 --replication-factor 3
- sudo /usr/share/kafka/bin/kafka-topics.sh --zookeeper pipeline0:2181 --create --topic zeek-raw --partitions 3 --replication-factor 3

- sudo /usr/share/kafka/bin/kafka-topics.sh --zookeeper pipeline0:2181 --create --topic zeek-raw --partitions 3 --replication-factor 3
- ***on sensor*** sudo vi /usr/share/zeek/site/local.zeek
- sudo vi kafka.zeek
    - ```
        @load Apache/Kafka/logs-to-kafka

        redef Kafka::topic_name = "zeek-raw";
        redef Kafka::json_timestamps = JSON::TS_ISO8601;
        redef Kafka::tag_json = F;
        redef Kafka::kafka_conf = table(
            ["metadata.broker.list"] = "pipeline0:9092,pipeline1:9092,pipeline2:9092");

        event zeek_init() &priority=-5
        {
            for (stream_id in Log::active_streams)
            {
                if (|Kafka::logs_to_send| == 0 || stream_id in Kafka::logs_to_send)
                {
                    local filter: Log::Filter = [
                        $name = fmt("kafka-%s", stream_id),
                        $writer = Log::WRITER_KAFKAWRITER,
                        $config = table(["stream_id"] = fmt("%s", stream_id))
                    ];

                    Log::add_filter(stream_id, filter);
                }
            }

<!-- 
@load Apache/Kafka/logs-to-kafka

redef Kafka::topic_name = "zeek-raw";
redef Kafka::json_timestamps = JSON::TS_ISO8601;
redef Kafka::tag_json = F;
redef Kafka::kafka_conf = table(
    ["metadata.broker.list"] = "pipeline0:9092,pipeline1:9092,pipeline2:9092");

event zeek_init() &priority=-5
{
    for (stream_id in Log::active_streams)
    {
        if (|Kafka::logs_to_send| == 0 || stream_id in Kafka::logs_to_send)
        {
            local filter: Log::Filter = [
                $name = fmt("kafka-%s", stream_id),
                $writer = Log::WRITER_KAFKAWRITER,
                $config = table(["stream_id"] = fmt("%s", stream_id))
            ];

            Log::add_filter(stream_id, filter);
        }
    }
-->
---
> to test for traffic: curl google.com 
## FileBeat
***ON SENSOR*** 
1. sudo yum install filebeat
2. sudo mv /etc/filebeat/filebeat{.yml,.yml.bk}
1. cd /etc/filebeat
1. sudo curl -LO https://repo/fileshare/filebeat/filebeat.yml
1. sudo vi filebeat.yml

***in pipeline0*** 
1. sudo /usr/share/kafka/bin/kafka-topics.sh --list --zookeeper pipeline0:2181
    * lists topics
1. sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic fsf-raw
1. sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic suricata-raw

***ON SENSOR*** 

1. sudo systemctl start filebeat
1. Verify messages are added to suricata-raw
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic suricata-raw --from-beginning`

1. Verify messages are added to fsf-raw
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic fsf-raw --from-beginning`

***on elastic0***

1. sudo yum install elasticsearch -y
2. sudo mv /etc/elasticsearch/elasticsearch{.yml,.yml.bk}
1. sudo curl -LO https://repo/fileshare/elasticsearch/elasticsearch.yml

*for a single node*

2. make sure you're in the home directory and sudo vi elasticsearch.yml
    * ```
      cluster.name:  nsm-cluster
      node.name:  es-node-0
      path.data: /data/elasticsearch
      path.logs: /var/log/elasticsearch
      bootstrap.memory_lock: true
      network.host: _local:ipv4_
      http.port:9200
      discovery.type: single-node

*for multinode*

2. make sure you're in the home directory and sudo vi elasticsearch.yml
    * ```
        cluster.name:  nsm-cluster
        node.name:  es-node-0
        path.data: /data/elasticsearch
        path.logs: /var/log/elasticsearch
        bootstrap.memory_lock: true
        network.host: _site:ipv4_
        http.port:9200
        discovery.seed_hosts: ["elastic0","elastic1","elastic2"]
        cluster.initial_master_nodes: ["es-node-0","es-node-1","es-node-2"]
    * be syure to change the node name!!
<!--
cluster.name: nsm-cluster
node.name: es-node-0
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: _site:ipv4_
http.port:9200
discovery.seed_hosts: ["elastic0","elastic1","elastic2"]
cluster.initial_master_nodes: ["es-node-0","es-node-1","es-node-2"]

cluster.name: nsm-cluster
node.name: es-node-1
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: _site:ipv4_
http.port: 9200
discovery.type: single-node

-->
> if you created the single node first, be sure to cd into the new folder it was moved to before, and change the ownership again

1. sudo mv ~/elasticsearch.yml /etc/elasticsearch ; sudo chmod 640 /etc/elasticsearch/elasticsearch.yml
1. sudo mkdir /usr/lib/systemd/system/elasticsearch.service.d ; sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d
1. sudo vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf
    * ```
        [Service]
        LimitMEMLOCK=infinity
<!-- 
[Service]
LimitMEMLOCK=infinity
-->
1. sudo chmod 644 /usr/lib/systemd/system/elasticsearch.service.d/override.conf
1. sudo vi /etc/elasticsearch/jvm.options.d/jvm_override.conf
    * ```
        -Xms2g
        -Xmx2g
<!-- 
-Xms2g
-Xmx2g
-->
1. sudo mkdir -p /data/elasticsearch ; sudo chown elasticsearch: /data/elasticsearch ; sudo chmod 755 /data/elasticsearch/
1. sudo firewall-cmd --add-port={9200,9300}/tcp --permanent ; sudo firewall-cmd --reload
---
## Kibana
***on kibana machine***
1. sudo yum install kibana -y
2. sudo mv /etc/kibana/kibana{.yml,.yml.bk}
3. sudo vi /etc/kibana/kibana.yml
    * ```
        server.port: 5601
        server.host: localhost
        server.name: kibana
        elasticsearch.hosts: ["http://elastic0:9200","http://elastic1:9200","http://elastic2:9200"]

<!-- 
server.port: 5601
server.host: localhost
server.name: kibana
elasticsearch.hosts: ["https://elastic0:9200","https://elastic1:9200","https://elastic2:9200"]
-->
4. sudo yum install nginx -y
5. sudo vi /etc/nginx/conf.d/kibana.conf
    * ```
        server {
        listen 80;
        server_name kibana;
        proxy_max_temp_file_size 0;

        location / {
            proxy_pass http://127.0.0.1:5601/;

            proxy_redirect off;
            proxy_buffering off;

            proxy_http_version 1.1;
            proxy_set_header Connection "Keep-Alive";
            proxy_set_header Proxy-Connection "Keep-Alive";

        }

        }
<!-- 
server {
  listen 80;
  server_name kibana;
  proxy_max_temp_file_size 0;

  location / {
    proxy_pass http://127.0.0.1:5601/;

    proxy_redirect off;
    proxy_buffering off;

    proxy_http_version 1.1;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";

  }

}
-->
6. sudo vi /etc/nginx/nginx.conf
    * comment out lines 39-41

7. sudo systemctl enable nginx --now ; sudo systemctl enable kibana --now
1. sudo systemctl status nginx ; sudo systemctl status kibana
1. sudo firewall-cmd --add-port=80/tcp --permanent ; sudo firewall-cmd --reload
1. sudo curl -LO https://repo/fileshare/kibana/ecskibana.tar.gz
1. sudo mv ecskibana.tar.gz ~
1. cd ~ ; tar -zxvf ecskibana.tar.gz 
1. sudo yum install jq -y
1. cd ecskibana
1. sudo ./import-index-templates.sh http://elastic0:9200

