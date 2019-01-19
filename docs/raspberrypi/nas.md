# Raspberry Pi搭建NAS

## Raspberry Pi

树莓派官网为 https://www.raspberrypi.org/ 。  
通过官网可以跳转到指定的淘宝卖家店铺。按需选择即可。
个人建议附件只需要散热器、电源、多层外壳就好。  
如果没有microSD卡及配套的读卡器，建议同时购买（SD卡容量建议4G以上）。  
如果有单独的显示器，可考虑额外购置HDMI线。

0. Raspbian

树莓派支持许多系统，比如官方的Raspbian，专为NAS打造的OpenMediaVault等等。  
我们以官方推荐的Raspbian为例。(我下载的版本是2018-11-13-raspbian-stretch-lite)  
为了将镜像做成可启动U盘/SD卡，我们还需要下载Etcher或者Win32DiskImager。  
由于Etcher使用AWS服务，下载速度堪忧，建议Windows用户使用Win32DiskImager。  
安装Win32DiskImager，选取解压得到的img烧录进SD卡。

1. SSH

Raspbian默认没有启用SSH，需要在烧录后的SD卡boot磁盘下新建一个文件名为ssh的空文件（注意没有文件名后缀）。  
将SD卡插入树莓派，连接网线。过30秒左右，在路由器上查看新增的设备。  
使用SSH客户端（比如Windows上用XShell），连接树莓派。
```sh
#假设ip为10.110.97.115
ssh pi@10.110.97.115
#第一次连接需要接受证书，默认密码为raspberry
```

2. raspi-config

SSH连接成功后，会提示使用passwd命令修改密码。  
同时会提示因为地区代码没设置，导致WiFi未启用。  
Localisation Options --> Change WiFi Country --> CN


3. 文件格式选择

|     |优点                       |缺点                                 |
|-----|:-------------------------:|------------------------------------:|
|EXT4 |读写速度快                 |支持的最大卷为1EB，单个文件最大为16TB|
|BTRFS|支持快照、RAID、针对SSD优化|读写速度比EXT4慢，稳定性有待验证     |

毫无疑问，若要稳定使用，选EXT4，想自己折腾自己，考虑BTRFS吧。

我买了个2G的3.5寸SATA3硬盘，树莓派的USB2.0接口是提供不了需要的电源，而且接口不对，所以额外买了USB2.0和SATA接口互转的线和电源线（转接线上专门留出电源接口）。

格式化硬盘：

```sh
#需先确认哪块硬盘需要格式化
sudo mkfs.ext4 /dev/sda
sudo fdisk -l
sudo mount /dev/sda /home/pi/nas
#mount到用户目录下，避免权限问题
#如果mount到/mnt/nas则需执行命令，以便读写
#sudo chmod -R 777 /mnt/nas 
#如果需要开机启动自动mount，则
#sudo vi /etc/fstab
#/dev/sda    /mnt/nas    nfs    auto,noatime,rw,sync    0    2
#sudo reboot
```

4. 协议选择

|    |优点                    |
|----|-----------------------:|
|NFS |速度快                  |
|SMB |基于NetBIOS，Windows兼容|
|DLNA|速度快，适用于多媒体播放|


   * 安装NFS

     ```sh
     sudo apt-get update
     #nfs-common已安装
     #默认使用rpcbind而不是portmap
     sudo apt-get install nfs-kernel-server
     ```

   * 配置NFS

     ```sh
     #共享目录必须存在
     #/etc/exports第一列表示共享目录，第二列表示授权访问主机，可以是域名、IP，括号内的是选项
     #选项解释：
     #rw代表读写权限，ro代表只读权限。sync代表文件同步写入内存和磁盘，async代表文件先写入内存，必要时再写入磁盘。
     #no_root_squash代表客户端用root访问时，共享文件也拥有root权限。root_squash代表客户端用root访问时，对分享目录有匿名用户权限。all_squash代表客户端总是只有匿名用户权限。
     #anonuid代表匿名用户的uid，默认为nobody。anongid代表匿名用户的gid。
     sudo vi /etc/exports
     /home/pi/nas 192.168.1.0/24(rw,sync)
     sudo /etc/init.d/nfs-kernel-server restart
     #如果启动失败，根据提示查看原因(一般都是配置写错了)
     systemctl status nfs-server.service
     ```

5. Windows客户端配置

   + 启用NFS支持

    控制面板 --> 程序和功能 --> 启用或关闭Windows功能 --> NFS服务

   + 挂载网络驱动

     ```cmd
     mount 192.168.1.99:/home/pi/nas X:
     #如果想开机自动挂载，可在计算机选择“映射网络驱动器”，在弹出窗口选择驱动器盘符和远程目录。
     #注意不要使用中文目录或文件，否则会出现乱码
     ```

6. NAS系统介绍

|         |OpenMediaVault    |群晖DMS                       |威联通QTS                   |铁马威TOS                |
|--------|:-----------------:|:---------------------------:|:---------------------------:|------------------------:|
|基础系统|Debian             |Debian?                      |Linux                        |busybox?                 |
|文件格式|EXT4               |BTRFS		                   |BTRFS                        |BTRFS                    |
|文件协议|NFS, SMB, AFP      |SMB, AFP, NFS                |DLNA, NFS, SMB               |SMB, AFS, NFS, DLNA      |
|内置应用|                   |nginx, php, htppd, pg, python|qthttpd, MariaDB, php, python|nginx, php, mysql, python|
|WebUI   |![OMV](https://github.com/iMinusMinus/ex/blob/master/images/raspberry%20pi/OMV.PNG?raw=true)|![DMS](https://github.com/iMinusMinus/ex/blob/master/images/raspberry%20pi/DSM.PNG?raw=true)|![QTS](https://github.com/iMinusMinus/ex/blob/master/images/raspberry%20pi/QTS.PNG?raw=true)|![TOS](https://github.com/iMinusMinus/ex/blob/master/images/raspberry%20pi/TOS.PNG?raw=true)|

在安装OMV时，出现命令找不到的情况，重新安装OMV相关package解决：
 
```
apt-get install --reinstall openmediavault && omv-initsystem
omv-firstaid
#此时如果提示web管理控制台信息，则说明安装成功
```

附录：

1. [Raspbian下载地址](https://downloads.raspberrypi.org/raspbian_lite_latest)

2. [win32diskimager下载地址](https://sourceforge.net/projects/win32diskimager/files/latest/download)

3. [群晖在线体验demo](https://demo.synology.cn/zh-cn)

4. [威联通在线体验demo](https://www.qnap.com/zh-cn/live-demo)

5. [铁马威在线体验demo](https://www.terra-master.com/us/live-demo/)