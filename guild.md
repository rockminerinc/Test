## Raspbian guild

#### Download img

    http://downloads.raspberrypi.org/raspbian/images/

下载Win32DiskImager工具win32diskimager-v0.9-binary.zip<云盘：所有文件\文档\树莓派>,装载img文件到SD卡中.

#### 插入SD卡开机
    startx命令进入图形界面
    执行raspi-config配置工具
    扩展磁盘分区
    禁用overscan,使得屏幕最大化.

#### 安装WEB服务

````
$  sudo apt-get install apache2
$  sudo apt-get install php5
$  sudo chmod -R 777 /var/www
$  sudo chmod 777 /etc/network/interfaces
$  sudo chmod 777 /etc/resolv.conf 

````
下载最新webUI<所有文件\文档\树莓派\webui>,通过SSH上传webUI文件到树莓派，替换var目录下www文件夹

编辑/etc/sudoers 添加 www-data 为sudo用户

````
$  sudo nano /etc/sudoers
````

添加一行 Defaults visiblepw

````
#includedir /etc/sudoers.d
pi ALL=(ALL) NOPASSWD: ALL
www-data ALL=(ALL) NOPASSWD: ALL
````

#### 增加算力图形界面

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

#### 添加PHP服务

````
$  sudo chmod 777 /home/pi/cgminer.conf 
$  sudo chmod 777 /var/www/data
$  sudo chmod 777 /etc/network/interfaces
$  sudo chmod 777 /etc/resolv.conf 

````
#### 安装cgminer

```
$  sudo apt-get update
$  sudo apt-get install libusb-1.0-0-dev libusb-1.0-0 libcurl4-openssl-dev libncurses5-dev libudev-dev
$  wget http://ck.kolivas.org/apps/cgminer/
$  tar xvf cgminer-4.3.2.tar.bz2
$  cd cgminer
$  ./configure --enable-icarus
$  sudo  make
```

####制作一个服务自启动。
在 /etc/init.d/目录下新建cgminer文件：

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
    stop)
        killall cgminer
        ;;
*)
echo "Usage: $0 (start|stop)"
;;
esac
```

修改cgminer文件权限，并执行命令

```
sudo update-rc.d cgminer defaults
```

####  为了防止cgminer自己挂掉 增加监控脚本

新建在pi用户下新建shell目录，然后新建monitor_cgminer.sh文件
````
pi@raspberrypi ~ $ mkdir shell 
pi@raspberrypi ~/shell $ sudo nano monitor_cgminer.sh

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
*/1 * * * * /usr/bin/lynx  -source  http://localhost/index.php/home/SaveHashrate  > /dev/null 2>&2
*/5 * * * * sudo bash /home/pi/shell/monitor_cgminer.sh
0 */1  * * * sudo cp /dev/null /home/pi/nohup2.out


````

#### 备份树莓派镜像
