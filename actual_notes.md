# NSM ENGINEERING NOTES
* to get containers started: 
    * `lxc list` to list the containers
    * `lxc start -all` to start all containers
> passwords for the container is `training`
* to disable ipv6, ***vi/vim*** into `/etc/sysctl.conf` and ensure that 1 is next to disable ipv6
* `ip a` to verify interfaces
* `sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`
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
<!--
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.30
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
* in vi,
    * `:%d` deletes the whole document
* to restart the network, type `sudo systemctl restart network`
>to ensure proper copy and paste format, right click for both, and make sure youre in insert mode

* into `~/.ssh/config` file, copy and paste this into the file:
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
* to recursively set ssh id key to each host in the container:`for host in sensor repo elastic{0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done`

1. `sudo yum install nginx` is a place that can store files through a web browser

1. `unzip all-class-files.zip -d /usr/share/nginx/` 

1. `sudo mv /usr/share/nginx/all-class-files /usr/share/nginx/fileshare`

1. `sudo mv emerging.rules.tar.gz /usr/share/nginx/fileshare`

* `sudo firewall-cmd --add-port=800/tcp --permanent`
* `sudo firewall-cmd --reload`