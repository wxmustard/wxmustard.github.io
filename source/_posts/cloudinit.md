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

## 使用`cloudinit`对虚拟机实现初始化

第一步：安装必要依赖

```bash
sudo apt install cloud-init cloud-utils
```

第二步：编写`meta-data`和`user-data`

```
# meta-data内容如下ssh
instance-id: vm03; local-hostname: vm03
```

```
# user-data内容如下
#cloud-config
# Hostname management
preserve_hostname: False
hostname: vm03
fqdn: vm03.example.local
# Remove cloud-init when finished with it
runcmd:
  - [ yum, -y, remove, cloud-init ]
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

```

第三步：准备虚拟机硬盘

```bash
cp ~/virt/images/xenial-server-cloudimg-amd64-disk.img vm03.qcow2
```

第四步：从`meta-data、user-data`中获得配置，生成`iso`文件,并将操作写入日志

```bash
genisoimage -output vm03-cidata.iso -volid cidata -joliet -r user-data meta-data &>> vm03.log
```

第五步：使用`virt-install`创建虚拟机

```bash
virt-install --import --name vm03 --ram 786 --vcpus 1 --disk vm03.qcow2,format=qcow2,bus=virtio --disk vm03-cidata.iso,device=cdrom --network bridge=virbr0,model=virtio --graphics vnc,listen=0.0.0.0,port=5903 --os-type=linux --os-variant=rhel7 --noautoconsole --mac=52:54:00:09:e6:03
```

第六步：Cleaning up cloud-init...

```bash
virsh change-media vm03 hda --eject --config >> vm03.log
```

第七步：虚拟机创建完成

```bash
# 添加指纹
ssh-keygen -f "/home/mustard/.ssh/known_hosts" -R vm03  
ssh ubuntu@vm03
```



## 一个使用`cloudinit`和`cloudimg`创建虚拟机的脚本文件

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



## 添加vnc ports listen和时区 

```sh
#!/bin/bash

## **Updates to this file are now at https://github.com/giovtorres/kvm-install-vm.**
## **This updated version has more options and less hardcoded variables.**

# Take one argument from the commandline: VM name
if ! [ $# -eq 3 ]; then
    echo "Usage: $0 <node-name>"
	echo "Usage: $1 <node-mac>"
	echo "Usage: $2 <node-vncport>"
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
        exit 1ssh-keygen -f "/home/mustard/.ssh/known_hosts" -R vm03

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
    bridge=virbr0,model=virtio --graphics vnc,listen=0.0.0.0,port=$3 --os-type=linux --os-variant=rhel7 --noautoconsole --mac=$2 
	
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

## 添加欢迎说明、硬盘扩充、ssh公钥传入的脚本

```bash
#!/bin/bash

## **Updates to this file are now at https://github.com/giovtorres/kvm-install-vm.**
## **This updated version has more options and less hardcoded variables.**

# Take one argument from the commandline: VM name
if ! [ $# -eq 7 ]; then
    echo "Usage: $0 <node-name>"
    echo "Usage: $1 <node-mac>"
    echo "Usage: $2 <node-vncport>"
    echo "Usage: $3 <node-mem>"
    echo "Usage: $4 <node-cpus>"
    echo "Usage: $5 <node-size>"
    echo "Usage: $6 <node-keys>"
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
DIR=/home/ubuntu/kvm/virt/images

# Location of cloud image
IMAGE=$DIR/xenial-server-cloudimg-amd64-disk.img


# Amount of RAM in MB
MEM=$4

# Number of virtual CPUs
CPUS=$5

# Start the vm afterwards?
DISK_SIZE=$6

# Cloud init files
USER_DATA=user-data
META_DATA=meta-data
CI_ISO=$1-cidata.iso
DISK=$1.qcow2

# Bridge for VMs (default on Fedora is virbr0)
BRIDGE=virbr0

# SSH keys
#PUBFILE="/home/ubuntu/pub_keys/$3.pub"
#if [ $# -gt 2 ]; then
#    if [ -e "$PUBFILE" ]; then
#       KEYS=$(<$PUBFILE)
#        echo "This pub key will be used."
#    else
#        echo "There is no this pub key. Exiting..."
#        exit 1
#    fi
#else
#    KEYS=$(<~/pub_keys/lab7.pub)
#    echo "The local pub key will be used."
#fi
KEYS=$7
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
preserve_hostname: false
hostname: $1
fqdn: $1
# Remove cloud-init when finished with it
runcmd:
  - [ locale-gen, zh_CN.UTF-8]
  - [ apt, remove, cloud-init ]
apt:
  primary:
    - arches: [default]
      uri: https://mirrors.shu.edu.cn/ubuntu/
package_update: true
write_files:
    - content: |
        #!/bin/sh
        printf "\n"
        printf " * Documentation:  https://help.ubuntu.com\n"
        printf " * Management:     https://landscape.canonical.com\n"
        printf " * Support:        https://ubuntu.com/advantage\n"
        printf "\n"
        printf " * Welcome to bdlab Compute Service!\n"
        printf " * If you have a problem in use, please contact @zhonger(zhonger@live.cn)!\n"
        printf "\n"
      path: /etc/update-motd.d/10-help-text
      permissions: "0755"
    - content: |
        #!/bin/sh
      path: /etc/update-motd.d/51-cloudguest
      permissions: "0755"
timezone: Asia/Shanghai
# Configure where output will go
output: 
  all: ">> /var/log/cloud-init.log"
# configure interaction with ssh server
ssh_svcname: ssh
ssh_deletekeys: True
ssh_genkeytypes: ['rsa', 'ecdsa']
# Install my public ssh key to the first user-defined user configured 
# in cloud.cfg in the template (which is centos for CentOS cloud images)
ssh_authorized_keys:
   -  $KEYS
_EOF_

    echo "instance-id: $1; local-hostname: $1" > $META_DATA
    cp $IMAGE $DISK
    qemu-img resize $DISK $DISK_SIZE
    # Create CD-ROM ISO with cloud-init config
    genisoimage -output $CI_ISO -volid cidata -joliet -r $USER_DATA $META_DATA &>> $1.log
    
    virt-install --import --name $1 --ram $MEM --vcpus $CPUS --disk \
    $DISK,format=qcow2,bus=virtio --disk $CI_ISO,device=cdrom --network \
    bridge=virbr0,model=virtio --os-type=linux --graphics vnc,listen=0.0.0.0,port=$3 --os-variant=rhel7 --noautoconsole --mac=$2

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

