title: 用 virtual box 运行一个多节点的 k8s 集群
date: 2019-06-21 15:19:54
tags: [kubernetes,docker,虚拟化,virtual box,分布式]
categories: 技术

------

网络设置

```bash
# /etc/netplan/*.yaml

# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: no
            dhcp6: no
            addresses: [10.0.2.101/24, ]
            gateway4: 10.0.2.1
            nameservers:
                    addresses: [8.8.8.8, 8.8.4.4]
    version: 2
    
netplan apply
```

启动脚本

```bash
#!/bin/bash

VBoxManage startvm ubuntu00 --type headless
VBoxManage startvm ubuntu01 --type headless
VBoxManage startvm ubuntu02 --type headless
VBoxManage startvm ubuntu03 --type headless
```

关闭脚本

```bash
#!/bin/bash

VBoxManage controlvm ubuntu00 poweroff
VBoxManage controlvm ubuntu01 poweroff
VBoxManage controlvm ubuntu02 poweroff
VBoxManage controlvm ubuntu03 poweroff
```

设置hostname

```bash
hostnamectl set-hostname ubuntu00

vim /etc/cloud/cloud.cfg
preserve_hostname: true
```





```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

swapoff -a
vi /etc/fstab

kubeadm init --pod-network-cidr=172.11.0.0/16

kubeadm join 10.0.2.100:6443 --token dbst4w.beuy9i6hjiozxkx7 \
    --discovery-token-ca-cert-hash sha256:fd1bd6a64250cc5743c8140450723c57fb725bb5529be9c7f0c038eca9e6390d
   
```



翻墙设置

```bash
apt-get install tsocks
vi /etc/tsocks.conf

# This is the configuration for libtsocks (transparent socks)
# Lines beginning with # and blank lines are ignored
#
# The basic idea is to specify:
#	- Local subnets - Networks that can be accessed directly without
#			  assistance from a socks server
#	- Paths - Paths are basically lists of networks and a socks server
#		  which can be used to reach these networks
#	- Default server - A socks server which should be used to access
#			   networks for which no path is available
# Much more documentation than provided in these comments can be found in
# the man pages, tsocks(8) and tsocks.conf(8)

# Local networks
# For this example this machine can directly access 192.168.0.0/255.255.255.0
# (192.168.0.*) and 10.0.0.0/255.0.0.0 (10.*)

local = 192.168.2.0/255.255.255.0
local = 10.0.0.0/255.0.0.0

# Paths
# For this example this machine needs to access 150.0.0.0/255.255.0.0 as
# well as port 80 on the network 150.1.0.0/255.255.0.0 through
# the socks 5 server at 10.1.7.25 (if this machines hostname was
# "socks.hello.com" we could also specify that, unless --disable-hostnames
# was specified to ./configure).

path {
	reaches = 150.0.0.0/255.255.0.0
	reaches = 150.1.0.0:80/255.255.0.0
	server = 10.1.7.25
	server_type = 5
	default_user = delius
	default_pass = hello
}

# Default server
# For connections that aren't to the local subnets or to 150.0.0.0/255.255.0.0
# the server at 192.168.0.1 should be used (again, hostnames could be used
# too, see note above)

server = 192.168.2.7
# Server type defaults to 4 so we need to specify it as 5 for this one
server_type = 5
# The port defaults to 1080 but I've stated it here for clarity
server_port = 1086


tsocks apt-get update
tsocks aptitude upgrade
tsocks xxxx
```

