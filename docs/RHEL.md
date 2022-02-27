RHEL的全称为Red Hat Enterprise Linux，它是为追求稳定的企业用户打造的企业级Linux系统。普通用户可以选择试验性质的Fedora，来获取更新的内核和更新的软件，当然代价就是更容易出问题。
企业用户如果不想掏钱，那么可以使用CentOS。由于RHEL的代码必须开源，所以实质上CentOS就是去除Redhat商标的RHEL。不过正因为没有购买，出现问题只能自己解决或者求助社区。
需要补充的是，由于CentOS被Redhat收购，CentOS也变成了RHEL的上游，即Fedora发布后，用户使用一段时间，基本没啥问题后再发布CentOS Stream，CentOS被使用一段时间后，再发布RHEL。

不过值得庆幸的是，RHEL并不会针对个人学习使用收取费用。


# 下载

根据CPU类型，从[RHEL官网](https://developers.redhat.com/products/rhel/download)下载光盘镜像。
比如64位的Intel/AMD CPU选择x86_64；如果是ARMv8的CPU，选aarch64（某些场合也称为arm64，如Debian）。
光盘镜像除CPU架构不同外，根据操作系统是否含有附带软件又区分为DVD和Boot镜像。

下载镜像需要红帽账号，登录后选择对应的镜像，网站会自动开始下载。
需要注意的是，整个下载链接只有240分钟的有效期。所以，请务必保持较好的网速！（8G大小时，平均下载速度需不低于570K/s）

当然，国内某些网站提供了镜像，比如搜索rhel镜像出来的[山东女子学院镜像](https://mirrors.sdwu.edu.cn/redhat/)。

# 安装

## 分区
禁用swap分区，在内存较大时，禁用swap来提升性能。

将 "/home"和"/var"分区放到其他机械硬盘上，避免频繁读写影响固态寿命，避免文件过大，单盘容量不足。

## dnf

1. media.repo
设置软件包地址（默认指向安装光驱）
在RHEL8，yum只是dnf的一个软链接。

```sh
which yum
# /usr/bin/yum
ls -alh /usr/bin/yum
# dnf-3
ls -alh /usr/bin/dnf
# dnf-3
```

在"/etc/yum.repos.d"目录下新建文件media.repo，用于指向光盘镜像的附带软件，即rhel-{m}.{n}-{arch}-dvd.iso。
其中该文件分两个部分，InstallMedia-BaseOS的baseurl指向BaseOS目录，我们可以从"/mnt/cdrom/BaseOS"里复制到磁盘目录"/home/administrator/BaseOS"。
InstallMedia-AppStream指向Apptream目录，我们可以从"/mnt/cdrom/AppStream"复制到磁盘目录"/home/administrator/AppStream"。
一个可能的内容如下：

```txt
[InstallMedia-BaseOS]
name = Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
metadata_expire=-1
gpgcheck=1
enabled=1
baseurl=file:///home/administrator/BaseOS/
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
cost=500
[InstallMedia-AppStream]
name = Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
baseurl = file:///home/administrator/AppStream/
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
metadata_expire = 86400
enabled_metadata = 1
```
2. redhat.repo

在较高版本的RHEL，安装时就要求联网，输入账号信息进行激活，可以跳过此步骤。
较低版本的redhat.repo文件由rhsm自动生成，没有配置任何信息。可以按照如下步骤操作：

```sh
# 注册，填写在红帽官网注册的用户名和密码 
subscription-manager register
# 注册成功后,redhat.repo内容就会被修改，配置完成
# 查看所有repo,会发现现在仓库标识多了rhel-8-for-x86_64-appstream-rpms和rhel-8-for-x86_64-baseos-rpms
yum repolist
# 查看redhat.repo也能看到文件内容多了很多信息
cat /etc/yum.repos.d/redhat.repo
# dnf clean all
# dnf makecache
# 如果不想注册到红帽，或者因网络问题，注册不了，可以下载Centos-8.repo，将内容复制到redhat.repo
curl -o Centos-8.repo http://mirrors.aliyun.com/repo/Centos-8.repo
cat Centos-8.repo >> /etc/yum.repos.d/redhat.repo
# 除aliyun外，清华\中科大\华为等大学/公司也提供了镜像地址
```

3. epel.repo

EPEL(Extra Packages for Enterprise Linux)是社区志愿者为RHEL系统(及其衍生系统)提供的高质量附加软件包仓库。

```sh
# 安装软件包
# yum install epel-release
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
# 安装完毕后"/etc/yum.repos.d/"下有epel相关的存储库了
ls -alh /etc/yum.repos.d/
# 启用 "codeready-builder-for-rhel-8-$(arch)-rpms" 存储库(EPEL包可能依赖)，EPEL支持x86_64，aarch64等架构
subscription-manager repos --enable "codeready-builder-for-rhel-8-$(arch)-rpms" 
# 启用epel-testing
# dnf config-manager --set-enabled epel-testing
# 禁用epel-testing
# dnf config-manager --set-disable epel-testing 
# 临时使用epel-testing
# dnf upgrade --enablerepo=epel-testing
# dnf install <foo> --enablerepo=epel-testing

# 更换镜像
sudo sed -e 's|^metalink=|#metalink=|g' \
         -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -i.bak \
         /etc/yum.repos.d/epel.repo
```

4. elrepo.repo

ELRepo项目作为企业级Linux的软件包仓库，由志愿者维护，主要聚焦于硬件驱动相关的软件包，如文件系统驱动、图像驱动、网络驱动、声卡驱动、摄像头和视频驱动等。

```sh
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
##### RHEL7/CentOS7: sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
sudo yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
sudo yum --enablerepo=elrepo search kmod-nvidia
# sudo yum --enablerepo=elrepo-extras search
# sudo yum --enablerepo=elrepo-testing install
# sudo yum --enablerepo=elrepo-kernel install
```

5. WIFI

使用有线网卡，而且没有或不使用无线网卡的可以跳过此步骤。
安装时可以联网，最小安装后WIFI可能无法使用！

```sh
# 查看网络配置：可以看到显示的wl开头的网卡适配器没有IP地址，或者有IPv6地址，但是ping不通外网
ifconfig -a
# 查看配置文件，SSID名称为你连接无线路由的名称
ls -alh /etc/sysconfig/network-scripts
cat /etc/sysconfig/network-scripts/ifcfg-{SSID}
# 由于目前IPv6的可用性处于薛定谔的猫状态，先禁用IPv6，可以启动图形界面禁用
nmtui
# 再次查看，应该看到已禁用IPv6
cat /etc/sysconfig/network-scripts/ifcfg-{SSID} | grep IPv6_DISABLED
# 尝试启用无线网络连接"wlo1"，具体名字根据上一个命令结果
nmcli c up wlo1
# 命令提示失败原因
# Error: Connection activation failed: No suitable device found for this connection(device lo not available because device is strictly unmanaged)
# 查看服务状态
sudo systemctl status NetworkManager
# 可以看到提示
# 'wifi' plugin not available; creating generic device
```

如果是完整DVD安装，可以尝试使用yum/dnf命令安装：

```sh
sudo yum install NetworkManager-wifi
# 如果提示当前用户不在sudoers中时，执行visudo，以“username”用户名只在本机可运行yum/dnf为例添加一行如下（不含“#”）：
# username localhost=/usr/bin/yum,/usr/bin/dnf,/usr/bin/dnf-3
# 更改完后，重新执行安装命令
sudo dnf install NetworkManager-wifi
```

如果只有无线网络，但安装了双系统，且Windows系统与Linux在不同的硬盘上时，可以考虑在Windows系统使用WSL来解决：

```cmd
wsl --update
wsl --shutdown
# 显示支持的linux发行版，显示结果分两列，一列名字（下一个安装命令会用到），另一列为还是名字（全名，可能包含版本等其他信息）
wsl --list --online
# 选取其中一个发行版，进行安装
wsl --install -d Debian

# 查看磁盘信息
diskpart
list disk
# WSL不能挂载启动分区所在磁盘，找到要挂载的磁盘序号（假设为0）
select disk 0
# 找到要挂载的分区号
list partition
## 挂载整个磁盘时也可以使用“wmic diskdrive list brief”来查看磁盘信息

# 装载指定分区，命令格式：wsl --mount <DiskPath> --partition <PartitionNumber> --type <Filesystem>
wsl --mount \\.\PHYSICALDRIVE0 --partition 1 --type ext4
# 卸载
wsl --unmount \\.\PHYSICALDRIVE0
# 如果卸载失败，则退出wsl来卸载
wsl --shutdown
```

6. GUI

如果使用启动镜像最小化安装，启动后是没有图形界面的。

安装图形界面方式如下：

```sh
# 查看程序组
dnf group list
# 安装
dnf groupinstall "Server with GUI"
# 设置启动后默认界面
systemctl set-default graphical.target
# 使用GUI界面
systemctl isolate graphical.target
# 重启
# reboot
```

## 与Windows系统互通

1. Windows文件格式读写

双系统没有虚拟机方便的一点是，切换系统需要重启。但好处是性能会比虚拟机高很多。
但有时候，Windows用的比较顺手，有的时候需要Redhat来试验些东西，而且有些东西也只能用Linux来实验。
这时我们需要打通两个系统的文件，当然通过网盘，或者自建NAS也是可行，但毕竟又多了些条件。
我们需要通过U盘，甚至直接读写NTFS文件系统。
如果是U盘，一般建议格式化为exFAT。

```sh
# fuse-exfat可以用于读写exfat格式，在“https://access.redhat.com/downloads/content/package-browser”网页搜索，可以发现它不在正式软件包
# 通过搜索exfat，比如网站“https://pkgs.org/search/?q=exfat”，我们找到下载地址，同时发现一个可选的工具exfatprogs在EPEL
sudo dnf install https://download1.rpmfusion.org/free/el/updates/8/x86_64/f/fuse-exfat-1.3.0-3.el8.x86_64.rpm
sudo dnf install exfatprogs
# ntfs-3g和ntfsprogs都在EPEL，ntfs-3g用于挂载及读写，ntfsprogs提供了额外的工具（如格式化成NTFS、解密、列出目录、输出文件内容）
sudo dnf install ntfsprogs

# 查看磁盘
sudo fdisk -l
# 挂载U盘（假定为sdb1，让系统自动识别文件格式）
mount /dev/sdb1 /media
# 挂载Windwos文件系统
mount.ntfs-3g /dev/sda0 /mnt/c
```

2. UEFI & GRUB2

安装完Linux后，如果Windows启动项丢失，在grub启动界面按"c"健，使用如下命令到Windows系统：

```
# 查看硬盘
ls
# 根据硬盘和分区设置
set root=(hd0,gpt2)
# 如果是BIOS引导
# chainloader +1
# 如果是UEFI引导
chainloader /EFI/Microsoft/bootmgfw.efi
# 启动
boot
```

如果直接格式化原来的EFI分区，通过上述方法是无法启动的。
因为UEFI要求磁盘必须以GPT方式分区，分区后多个系统的启动文件会在同一个分区的不同文件夹下！当我们格式化分区，安装其他系统时，相当于丢失原有的启动文件，自然无法多系统启动。
此时，我们可以利用多系统启动文件在同一个分区这个特性，再安装原来的Windows操作系统到另一个分区上。
安装成功后，分区信息在Linux系统信息如下：

```sh
[root@localhost]# fdisk -l
Disk /dev/nvme0n1：238.5 GiB，256060514304 字节，500118192 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
磁盘标识符：27EE6043-B2C3-42B1-B1A6-2C3F85DF9A68

设备                起点      末尾      扇区  大小 类型
/dev/nvme0n1p1      2048    206847    204800  100M EFI 系统
/dev/nvme0n1p2    206848    239615     32768   16M Microsoft 保留
/dev/nvme0n1p3    239616 208613375 208373760 99.4G Microsoft 基本数据
/dev/nvme0n1p4 208613376 209952767   1339392  654M Windows 恢复环境
/dev/nvme0n1p5 209952768 212049919   2097152    1G Linux 文件系统
/dev/nvme0n1p6 212049920 316923903 104873984   50G Linux LVM
/dev/nvme0n1p7 316923904 500117503 183193600 87.4G Microsoft 基本数据

[root@localhost]# mount /dev/nvme0n1p1 /media

[root@localhost]# ls -alh /media/EFI
总用量 24K
drwx------. 5 root root 2.0K 12月 15 15:42 .
drwx------. 3 root root  16K 1月   1 1970 ..
drwx------. 2 root root 2.0K 12月 15 15:44 BOOT
drwx------. 4 root root 2.0K 12月 17 19:37 Microsoft
drwx------. 3 root root 2.0K 12月 17 19:42 redhat

[root@localhost]# umount /dev/nvme0n1p1
```

由于Linux缺乏编辑BCD的工具，回到新安装的Windows系统，下载bootice，运行，然后选择加载当前系统BCD，在智能模式下，将Windows启动分区设置到原来的Windows分区，最后保存系统设置退出程序，重启便可以回到原来的Windows系统。
![bootice BCD](/images/BCD.png?raw=true)
虽然将原来的Windows系统找回变得可启动，但是Linux的GRUB2启动选项并没有Windows菜单。
每次都通过设置UEFI启动顺序来控制使用哪个系统没有GRUB2菜单选择方便。
此时需要再登入Linux系统

```sh
# 不同的Linux发行版，EFI目录不一样，比如centos可能是“/EFI/centos”，
# redhat将EFI分区挂载在“/boot/efi”目录下，使用如下命令查看
cat /boot/efi/EFI/redhat/grub.cfg
# 终端输出30_os-prober信息如下：
### BEGIN /etc/grub.d/30_os-prober ###
### END /etc/grub.d/30_os-prober ###

sudo grub2-mkconfig
```

正常情况下，在命令运行完后，终端会看到如下信息：

```
### BEGIN /etc/grub.d/30_os-prober ###
Found Windows Boot Manager on /dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi
menuentry 'Windows Boot Manager (on /dev/nvme0n1p1)' --class windows --class os $menuentry_id_option 'osprober-efi-A25C-CF85' {
	insmod part_gpt
	insmod fat
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root  A25C-CF85
	else
	  search --no-floppy --fs-uuid --set=root A25C-CF85
	fi
	chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
# Other OS found, undo autohiding of menu unless menu_auto_hide=2
if [ "${orig_timeout_style}" -a "${menu_auto_hide}" != "2" ]; then
  set timeout_style=${orig_timeout_style}
  set timeout=${orig_timeout}
fi
### END /etc/grub.d/30_os-prober ###
```

此时准备重新生成grub.cfg。

```sh
sudo cat /etc/default/grub | grep GRUB_DISABLE_OS_PROBER
# 如果没有此配置，则追加配置：GRUB_DISABLE_OS_PROBER=false
sudo cp /boot/efi/EFI/redhat/grub.cfg /boot/efi/EFI/redhat/grub.cfg.bak
sudo rm -rf /boot/efi/EFI/redhat/grub.cfg
sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
# sudo rm -rf /boot/efi/EFI/redhat/grub.cfg.bak
```
再查看grub.cfg可以看到原先的30_os-prober之间有了一个menuentry，内容如下：

```
### BEGIN /etc/grub.d/30_os-prober ###
menuentry 'Windows Boot Manager (on /dev/nvme0n1p1)' --class windows --class os $menuentry_id_option 'osprober-efi-A25C-CF85' {
	insmod part_gpt
	insmod fat
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root  A25C-CF85
	else
	  search --no-floppy --fs-uuid --set=root A25C-CF85
	fi
	chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
# Other OS found, undo autohiding of menu unless menu_auto_hide=2
if [ "${orig_timeout_style}" -a "${menu_auto_hide}" != "2" ]; then
  set timeout_style=${orig_timeout_style}
  set timeout=${orig_timeout}
fi
### END /etc/grub.d/30_os-prober ###

```

重启查看GRUB2启动菜单，如果出现Windows菜单，并可正常进入Windows系统便说明修复成功。
如果需要将Windows系统设置为默认启动系统，可以采取如下方式：

```sh
# 根据原启动界面的顺序，比如Linux一般又2个启动项，那么Windows是第三个启动项
sudo grub2-set-default 2
# 上述方法在新安装其他操作系统时可能需要重新调整，为避免调整，可以采用下面一种方式（但注意menuentry修改时要重新运行设置默认启动系统命令
sudo grub2-set-default 'Windows Boot Manager (on /dev/nvme0n1p1)'
sudo reboot
```

## 更新

操作系统内核和软件包都需要定期更新来获取更新功能，同时降低安全风险。

```sh
sudo yum update
### 重启后的GRUB界面出现多个操作系统启动项，可以考虑删除旧的 ###
rpm -q kernel
# kernel-4.18.0-348.2.1.el8_5.x86_64
# kernel-4.18.0-348.7.1.el8_5.x86_64
sudo yum remove kernel-4.18.0-348.2.1.el8_5.x86_64
### 可选删除/boot下旧文件
# sudo rm -rf /boot/config-4.18.0-348.2.1.el8_5.x86_64
# sudo rm -rf /boot/initramfs-4.18.0-348.2.1.el8_5.x86_64.img
# sudo rm -rf /boot/initramfs-4.18.0-348.2.1.el8_5.x86_64kdump.img
# sudo rm -rf /boot/System.map-4.18.0-348.2.1.el8_5.x86_64
# sudo rm -rf /boot/vmlinuz-4.18.0-348.2.1.el8_5.x86_64
# sudo rm -rf /boot/.vmlinuz-4.18.0-348.2.1.el8_5.x86_64.hmac
# sudo rm -rf /boot/symvers-4.18.0-348.2.1.el8_5.x86_64.gz && rm -rf /lib/modules/4.18.0-348.2.1.el8_5.x86_64/symvers.gz

### 删除grub搜索文件
sudo ls -alh /boot/loader/entries/
# 91ee24112dd64dfd9910405658fd0dfe-0-rescue.conf
# 91ee24112dd64dfd9910405658fd0dfe-4.18.0-348.2.1.el8_5.x86_64.conf
# 91ee24112dd64dfd9910405658fd0dfe-4.18.0-348.7.1.el8_5.x86_64.conf
sudo rm -rf /boot/loader/entries/91ee24112dd64dfd9910405658fd0dfe-4.18.0-348.2.1.el8_5.x86_64.conf

### 重启
sudo reboot
```


0. [redhat package](https://access.redhat.com/downloads/content/package-browser)
1. [EPEL](https://fedoraproject.org/wiki/EPEL/zh-cn)
2. [EPEL mirros](https://admin.fedoraproject.org/mirrormanager/mirrors/EPEL)
3. [EPEL ustc mirrors](http://mirrors.ustc.edu.cn/help/epel.html)
4. [ELRepo](http://elrepo.org/tiki/HomePage)
5. [ELRepo tuna](https://mirrors.tuna.tsinghua.edu.cn/help/elrepo/)
6. [wsl2-mount-disk](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-mount-disk)
