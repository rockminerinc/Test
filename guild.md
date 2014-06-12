﻿## 烧制镜像至SD卡

#### download img

    http://www.raspberrypi.org/downloads

#### 插入SD卡，用df命令查看当前已挂载的卷


````
$ df -h
Filesystem      Size   Used  Avail Capacity  iused    ifree %iused  Mounted on
/dev/disk0s2    92Gi   33Gi   59Gi    36%  8609397 15410211   36%   /
devfs          187Ki  187Ki    0Bi   100%      648        0  100%   /dev
/dev/disk0s4   373Gi   71Gi  302Gi    19% 18594197 79281654   19%   /Volumes/d1
map -hosts       0Bi    0Bi    0Bi   100%        0        0  100%   /net
map auto_home    0Bi    0Bi    0Bi   100%        0        0  100%   /home
/dev/disk1s1   7.4Gi  3.0Mi  7.4Gi     1%        0        0  100%   /Volumes/NO NAME
````

#### 使用diskutil unmount将这些分区卸载

````
$ diskutil unmount /dev/disk1s1
Volume NO NAME on disk1s1 unmounted
````

#### 通过diskutil list来确认设备

````
$ diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            98.4 GB    disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
   4:                  Apple_HFS d1                      400.9 GB   disk0s4
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *7.9 GB     disk1
   1:                 DOS_FAT_32 NO NAME                 7.9 GB     disk1s1
````

#### 使用dd命令将系统镜像写入。

`/dev/disk1s1`是分区，`/dev/disk1`是块设备，`/dev/rdisk1`是原始字符设备。

````
$ sudo dd bs=4m if=2014-01-07-wheezy-raspbian.img of=/dev/rdisk1
706+1 records in
706+1 records out
2962227200 bytes transferred in 470.754370 secs (6292511 bytes/sec)
````

#### diskutil unmountDisk卸载设备

````
$ diskutil unmountDisk /dev/disk1
Unmount of all volumes on disk1 was successful
````

至此，树莓派SD烧录完毕。

### Raspberry Pi Source List

````
sudo vi  /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/   wheezy main contrib non-free rpi

sudo apt-get update
sudo apt-get install vim binutils-dev build-essential git zsh iptraf iftop screen
````

### 设置IP
`sudo vim /etc/network/interfaces`

```
# 动态IP
iface eth0 inet dhcp

# 静态IP
iface eth0 inet static
address 10.0.1.145
netmask 255.255.255.0
gateway 10.0.1.1
```

设置DNS: `vi /etc/resolv.conf`

```
nameserver 8.8.8.8
```

设置后保存，重启：`sudo /etc/init.d/networking restart`

### oh my zsh

```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s `which zsh`
```

### 设置系统时区
````
dpkg-reconfigure tzdata
````

### 安装nginx, php，用于查看监控数据

````
apt-get install nginx php5 php5-common php5-cli php5-fpm php5-gd php5-mcrypt php5-mysql php5-gmp

# 修改nginx配置，支持php-fpm
/etc/init.d/nginx restart

# 设置时区，修改php.ini
# date.timezone = Asia/Shanghai

/etc/init.d/php5-fpm restart
````



### Install cgminer

````
sudo apt-get install libusb-1.0-0-dev libusb-1.0-0 libcurl4-openssl-dev libncurses5-dev libudev-dev autoconf aptitude
sudo aptitude install libtool

mkdir -p ~/asic/cgminer
cd ~/asic/cgminer
git clone https://github.com/ckolivas/cgminer.git

cd cgminer
./autogen.sh
./configure --enable-hashratio
make
sudo make install
````

### Install cmd

```
# avalon
cgminer -o stratum+tcp://stratum.mining.eligius.st:3334 -u 1PDqnS6aC4xzUa9wPeJukbRMaJtttdm4Aj -p xxxx --avalon-options 115200:24:10:45:300 --avalon-temp 60 --avalon-cutoff 70 --api-allow "W:0/0" --api-listen --debug --syslog

# icarus
# cgminer -o stratum+tcp://stratum.mining.eligius.st:3334 -u 1PDqnS6aC4xzUa9wPeJukbRMaJtttdm4Aj -p xxxx --icarus-options 115200:1:1 --icarus-timing 3.0=100

```

## 备份树莓派镜像

````
sudo dd bs=4m if=/dev/rdisk1 of=rpi_hashratio_20140425.img
tar zcvf rpi_hashratio_20140425.img.tar.gz ./rpi_hashratio_20140425.img

# img文件与SD一样大小，这里是8G，压缩文件大约900MB
````