## Raspbian guild

#### 1.Download img

    http://downloads.raspberrypi.org/raspbian/images/

下载Win32DiskImager工具，img安装可参考网址
````
    http://www.cnbeta.com/articles/204970.htm
````

#### 2.获取IP地址
方法1.插入SD卡开机,连接显示器，键盘。在开机信息最后显示当前IP地址,如下：
````
My IP Address is 192.168.1.128
````

也可通过输入命令查看当前ip信息：
````
$  sudo ifconfig
````

如果网关与本地路由不符合，修改网关：
    
````
$  sudo route add default gw 192.168.1.1
````

方法2.通过登录路由器WEB界面的客户端列表，找到Raspberry设备的IP地址。

#### 3.修改为静态IP地址 

获取树莓派的IP地址后，后续操作可通过远程登录修改。

远程登录账户为pi，初始密码为raspberry。

备份/etc/network/interfaces文件，并打开：
````
$  cd /etc/network
$  sudo cp interfaces interfaces1
$  sudo nano interfaces
````
修改如下：
````
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.168.1.254
gateway 192.168.1.1
netmask 255.255.0.0
network 192.168.1.0
````

#### 4.安装WEB服务

````
$  sudo apt-get update
$  sudo apt-get install apache2
$  sudo apt-get install php5
$  sudo chmod -R 777 /var/www
$  sudo chmod 777 /etc/network/interfaces
$  sudo chmod 777 /etc/resols
````
下载最新webUI<所有文件\文档\树莓派\webui>,通过SSH上传webUI文件到树莓派，替换var目录下www文件夹

编辑/etc/sudoers 添加 www-data 为sudo用户

````
$  sudo nano /etc/sudoers
````

在其中添加：
````
 Defaults visiblepw
````
再结尾添加：

````
#includedir /etc/sudoers.d
pi ALL=(ALL) NOPASSWD: ALL
www-data ALL=(ALL) NOPASSWD: ALL
````

#### 5.增加算力图形界面

安装lynx 

````
$  sudo apt-get install lynx
````

添加crontab，1分钟保存一次数据

````
$  crontab -e
   */1 * * * * /usr/bin/lynx  -source  http://localhost/index.php/home/SaveHashrate  > /dev/null 2>&2
$  sudo /etc/init.d/cron restart
````

#### 6.添加PHP服务

````
$  sudo chmod 777 /var/www/data
$  sudo chmod 777 /etc/network/interfaces
$  sudo chmod 777 /etc/resolv.conf 
````
#### 7.安装cgminer

```
$  sudo apt-get install libusb-1.0-0-dev libusb-1.0-0 libcurl4-openssl-dev libncurses5-dev libudev-dev
$  cd /home/pi
$  sudo  wget https://github.com/rockminerinc/Test/blob/master/cgminer-4.3.3-rockminer.zip
$  sudo  unzip cgminer-4.3.3-rockminer.zip
$  cd cgminer
$  chmod u+x configure
$  sudo ./configure --enable-icarus
$  sudo  make
```
新建cgminer.conf配置文件
```
$  sudo nano /home/pi/cgminer.conf 
```

修改如下
```
{"pools":[{"url":"uk1.ghash.io:3333","user":"rockminer.1","pass":"Password"},{"url":"stratum.f2pool.com:3333","user":"rockxie.test","pass":"Password"}],"api-listen":true,"api-port":"4028","expiry":"120","failover-only":true,"log":"5","no-pool-disable":true,"queue":"2","scan-time":"60","worktime":true,"shares":"0","kernel-path":"/usr/local/bin","api-allow":"W:0/0","icarus-options":"115200:1:1","rmu-auto":"350"}
```
修改权限
```
$  sudo chmod 777 /home/pi/cgminer.conf 
```
#### 8.制作一个服务自启动。
在 /etc/init.d/目录下新建cgminer文件：
```
$  sudo nano /etc/init.d/cgminer
```
修改如下：

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          cgminer
# Required-Start:    $remote_fs
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start or stop the HTTP Proxy.
### END INIT INFO
case $1 in
    start)
          sudo nohup  /home/pi/cgminer/cgminer --config /home/pi/cgminer.conf > /home/pi/nohup2.out 2>&1 &
        ;;
    stop).
        killall cgminer
        ;;
*)
echo "Usage: $0 (start|stop)"
;;
esac
```

修改cgminer文件权限，并执行命令

```
$  sudo chmod 777 /home/pi/cgminer.conf
$  sudo update-rc.d cgminer defaults
```

####  9.cgminer增加监控脚本

新建在pi用户下新建shell目录，然后新建monitor_cgminer.sh文件
````
cd /home/pi
$ mkdir shell 
$ sudo nano monitor_cgminer.sh
````
脚本文件如下：

````
#!/bin/sh
ps -fe|grep cgminer/cgminer |grep -v grep
if [ $? -ne 0 ]
then
sudo reboot
now=`date  +%Y-%m-%d[%H:%M:%S]` 
echo "at $now reboot sys" >> /home/pi/shell/check_cgminer.log 

fi
````
增加crontab（定时检查执行）,执行 crontab -e  添加：

````
*/1 * * * * /usr/bin/lynx  -source  http://localhost/index.php/home/SaveHashrate  > /dev/null 2>&2
*/5 * * * * sudo bash /home/pi/shell/monitor_cgminer.sh
0 */1  * * * sudo cp /dev/null /home/pi/nohup2.out
````

#### 10.备份树莓派镜像
