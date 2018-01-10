---
title: cloudinit
author: mustard
date: 2017-12-18 19:31:18
tags:
 - ubuntu
categories:
 - 技术
---

## 目标

- 设置hostman、hosts

- 添加ssh keys 到 .ssh/authorized_keys

## 一个使用cloudinit和cloudimg创建虚拟机的脚本文件

```sh
#!/bin/bash

## **Updates to this file are now at https://github.com/giovtorres/kvm-install-vm.**
## **This updated version has more options and less hardcoded variables.**

# Take one argument from the commandline: VM name
if ! [ $# -eq 2 ]; then
    echo "Usage: $0 <node-name>"
    echo "Usage: $1 <node-mac>"
    exit 1
fi

# Check if domain already exists
virsh dominfo $1 > /dev/null 2>&1
if [ "$?" -eq 0 ]; then
    echo -n "[WARNING] $1 already exists.  "
    read -p "Do you want to overwrite $1 (y/[N])? " -r
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        echo ""
        virsh destroy $1 > /dev/null
        virsh undefine $1 > /dev/null
    else
        echo -e "\nNot overwriting $1. Exiting..."
        exit 1
    fi
fi

# Directory to store images
DIR=~/virt/images

# Location of cloud image
IMAGE=$DIR/CenvnctOS-7-x86_64-GenericCloud.qcow2

# Amount of RAM in MB
MEM=768

# Number of virtual CPUs
CPUS=1

# Cloud init files
USER_DATA=user-data
META_DATA=meta-data
CI_ISO=$1-cidata.iso
DISK=$1.qcow2

# Bridge for VMs (default on Fedora is virbr0)
BRIDGE=virbr0

# Start clean
rm -rf $DIR/$1
mkdir -p $DIR/$1

pushd $DIR/$1 > /dev/null

    # Create log file
    touch $1.log

    echo "$(date -R) Destroying the $1 domain (if it exists)..."

    # Remove domain with the same name
    virsh destroy $1 >> $1.log 2>&1
    virsh undefine $1 >> $1.log 2>&1

    # cloud-init config: set hostname, remove cloud-init package,
    # and add ssh-key 
    cat > $USER_DATA << _EOF_
#cloud-config
# Hostname management
preserve_hostname: False
hostname: $1
fqdn: $1.example.local
# Remove cloud-init when finished with it
runcmd:
runcmd:
  - [ locale-gen, zh_CN.UTF-8 ]
  - [ apt, remove, cloud-init ]
timezone: Asia/Shanghai

# Configure where output will go
output: 
  all: ">> /var/log/cloud-init.log"
# change mirrors
apt:
  primary:
    - arches: [default]
      uri: https://mirrors.shu.edu.cn/ubuntu/
# configure interaction with ssh server
ssh_svcname: ssh
ssh_deletekeys: True
ssh_genkeytypes: ['rsa', 'ecdsa']
# Install my public ssh key to the first user-defined user configured 
# in cloud.cfg in the template (which is centos for CentOS cloud images)
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBil2QzORhDcnKiVVNpO5daOSYVp8nshcIc7aTEkdlqCRir2Oni8BEStK7x7bvh0jrp9KptlHPeos87fQs//VXEb1FEprL2c6fPWmVdtjmYw3yzSkaFKMksL7FdUoEiwF6t8pQAg2mU0Qj9emSHBKg5ttdGqNoSvXc92k7iOzgauda7jdNak+Dx9dPhR3FJwHMcZSlQHO4cweZcK63bZitxlFkJ/FJdry/TBirDhRcXslOJ3ECU2xiyRXJVPs3VNLjMdOTTAoMmZj+GraUBbQ9VIqe683xe02sM83th5hj2C4gW3qXUoFkNLfKAMRxXLRMEwI3ABFB/AAUhACxyTJp giovanni@throwaway
_EOF_

    echo "instance-id: $1; local-hostname: $1" > $META_DATA

    echo "$(date -R) Copying template image..."
    cp $IMAGE $DISK

    # Create CD-ROM ISO with cloud-init config
    echo "$(date -R) Generating ISO for cloud-init..."
    genisoimage -output $CI_ISO -volid cidata -joliet -r $USER_DATA $META_DATA &>> $1.log

    echo "$(date -R) Installing the domain and adjusting the configuration..."
    echo "[INFO] Installing with the following parameters:"
    echo "virt-install --import --name $1 --ram $MEM --vcpus $CPUS --disk
    $DISK,format=qcow2,bus=virtio --disk $CI_ISO,device=cdrom --network
    bridge=virbr0,model=virtio --os-type=linux --os-variant=rhel7 --noautoconsole"

    virt-install --import --name $1 --ram $MEM --vcpus $CPUS --disk \
    $DISK,format=qcow2,bus=virtio --disk $CI_ISO,device=cdrom --network \
    bridge=virbr0,model=virtio --vnc --vncport=--os-type=linux --os-variant=rhel7 --noautoconsole

    MAC=$(virsh dumpxml $1 | awk -F\' '/mac address/ {print $2}')
    while true
    do
        IP=$(grep -B1 $MAC /var/lib/libvirt/dnsmasq/$BRIDGE.status | head \
             -n 1 | awk '{print $2}' | sed -e s/\"//g -e s/,//)
        if [ "$IP" = "" ]
        then
            sleep 1
        else
            break
        fi
    done

    # Eject cdrom
    echo "$(date -R) Cleaning up cloud-init..."
    virsh change-media $1 hda --eject --config >> $1.log

    # Remove the unnecessary cloud init files
    rm $USER_DATA $CI_ISO

    echo "$(date -R) DONE. SSH to $1 using $IP with  username 'centos'."

popd > /dev/null
```



## 添加vnc ports listen 

```sh
#!/bin/bash

## **Updates to this file are now at https://github.com/giovtorres/kvm-install-vm.**
## **This updated version has more options and less hardcoded variables.**

# Take one argument from the commandline: VM name
if ! [ $# -eq 3 ]; then
    echo "Usage: $0 <node-name>"
	echo "Usage: $1 <node-mac>"
	echo "Usage: $3 <node-vncport>"
    exit 1
fi

# Check if domain already exists
virsh dominfo $1 > /dev/null 2>&1
if [ "$?" -eq 0 ]; then
    echo -n "[WARNING] $1 already exists.  "
    read -p "Do you want to overwrite $1 (y/[N])? " -r
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        echo ""
        virsh destroy $1 > /dev/null
        virsh undefine $1 > /dev/null
    else
        echo -e "\nNot overwriting $1. Exiting..."
        exit 1
    fi
fi

# Directory to store images
DIR=/home/mustard/virt/images

# Location of cloud image
IMAGE=$DIR/xenial-server-cloudimg-amd64-disk.img

# Amount of RAM in MB
MEM=768

# Number of virtual CPUs
CPUS=1

# Cloud init files
USER_DATA=user-data
META_DATA=meta-data
CI_ISO=$1-cidata.iso
DISK=$1.qcow2

# Bridge for VMs (default on Fedora is virbr0)
BRIDGE=virbr0

# Start clean
rm -rf $DIR/$1
mkdir -p $DIR/$1

pushd $DIR/$1 > /dev/null

    # Create log file
    touch $1.log

    echo "$(date -R) Destroying the $1 domain (if it exists)..."

    # Remove domain with the same name
    virsh destroy $1 >> $1.log 2>&1
    virsh undefine $1 >> $1.log 2>&1

    # cloud-init config: set hostname, remove cloud-init package,
    # and add ssh-key 
    cat > $USER_DATA << _EOF_
#cloud-config
# Hostname management
preserve_hostname: False
hostname: $1
fqdn: $1.example.local
# Remove cloud-init when finished with it
runcmd:
  - [ locale-gen, zh_CN.UTF-8 ]
  - [ apt, remove, cloud-init ]
timezone: Asia/Shanghai
# Configure where output will go
output: 
  all: ">> /var/log/cloud-init.log"
apt:
  primary:
    - arches: [default]
      uri: https://mirrors.shu.edu.cn/ubuntu/
# configure interaction with ssh server
ssh_svcname: ssh
ssh_deletekeys: True
ssh_genkeytypes: ['rsa', 'ecdsa']
# Install my public ssh key to the first user-defined user configured 
# in cloud.cfg in the template (which is centos for CentOS cloud images)
ssh_authorized_keys:
 - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIMWzWLDpQJgNdzXKcEjMHdTrZTDVQknB9C3kE55oqi82WJh1HRueaZxvlZMH9T93URe1jmt9FAX+Pcd1hvGLzv3OPd03Oq5r8hVpQ7bitXtTngrdhvJ0zBW/E6fDo1uAnAeudW9FowMu+YBv5Yjd7Y8Wrjn1qzRjHKJyhpS09ieb5Q/cBnmNDm45oyVp4IqC5wc8xjjb6B2y9pMf8bEWfzYkneSeYYCFhoRm7sAM5Lt93tVNeRoo4VqRqrogSLwLR+Ls3mVYDAq6QkEHpvibw8TkrOkLmSKYAC4PWaiW6ZL4nuJDN+bIDcEnGtb8r6vRJhj+jlZIUCv7TZ6vLQ3vJ mustard@mustard
_EOF_

    echo "instance-id: $1; local-hostname: $1" > $META_DATA

    echo "$(date -R) Copying template image..."
    cp $IMAGE $DISK

    # Create CD-ROM ISO with cloud-init config
    echo "$(date -R) Generating ISO for cloud-init..."
    genisoimage -output $CI_ISO -volid cidata -joliet -r $USER_DATA $META_DATA &>> $1.log

    echo "$(date -R) Installing the domain and adjusting the configuration..."
    echo "[INFO] Installing with the following parameters:"
    echo "virt-install --import --name $1 --ram $MEM --vcpus $CPUS --disk
    $DISK,format=qcow2,bus=virtio --disk $CI_ISO,device=cdrom  --network --vnclient=0.0.0.0 --vncport=$3
    bridge=virbr0,model=virtio --os-type=linux --os-variant=rhel7 --noautoconsole"

    virt-install --import --name $1 --ram $MEM --vcpus $CPUS --disk \
    $DISK,format=qcow2,bus=virtio --disk $CI_ISO,device=cdrom --network \
    bridge=virbr0,model=virtio --graphics vnc,listen=0.0.0.0,port=$3 --noautoconsole --os-type=linux --os-variant=rhel7 --noautoconsole --mac=$2 
	
	MAC=$(virsh dumpxml $1 | awk -F\' '/mac address/ {print $2}')
    while true
    do
        IP=$(grep -B1 $MAC /var/lib/libvirt/dnsmasq/$BRIDGE.status | head \
             -n 1 | awk '{print $2}' | sed -e s/\"//g -e s/,//)
        if [ "$IP" = "" ]
        then
            sleep 1
        else
            break
        fi
    done

    # Eject cdrom
    echo "$(date -R) Cleaning up cloud-init..."
    virsh change-media $1 hda --eject --config >> $1.log

    # Remove the unnecessary cloud init files
    rm -rf $USER_DATA $CI_ISO

    echo "$(date -R) DONE. SSH to $1 using $IP with  username 'ubuntu'."

popd > /dev/null
```

