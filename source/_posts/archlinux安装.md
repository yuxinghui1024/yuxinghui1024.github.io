---
title: ArchLinux的安装
date: 2021-01-17 15:38:06
---
Arch Linux 是属于GNU/Linux的一个发行版，Linux只是一个内核，我们平常说的linux系统是指GNU/Linux。Arch的特点Keep It Simple, Stupid（对应中文为“保持简单，且一目了然”）。Arch还提供[Arch 用户仓库](https://wiki.archlinux.org/index.php/Arch_User_Repository_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，它包含了成千上万个由用户维护的[PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))脚本，配合[makepkg](https://wiki.archlinux.org/index.php/Makepkg_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))工具，从编译到打包一气呵成。

## 安装

### 安装介质的准备
1. [下载](https://archlinux.org/download/)ISO文件，准备8g以上的u盘
2. 推荐使用[Rufus](https://rufus.ie/)制作安装介质

### 启动到 Live 环境
1. 将制作好的u盘，插入电脑，并设置电脑从u盘启动，一般是开机按某个按键，具体情况请参考主板说明书。
2. 当引导加载程序菜单出现时，选择 Arch Linux install medium 并按 Enter 进入安装环境。
3. 您将会以 root 身份登录进一个[虚拟控制台](https://en.wikipedia.org/wiki/Virtual_console)，默认的 Shell 是 Zsh。

### 连接到因特网
* 有线以太网 —— 连接网线
* WiFi —— 使用 [iwctl](https://wiki.archlinux.org/index.php/Iwd#iwctl) 验证无线网络
    * 可以使用[iwd ](https://wiki.archlinux.org/index.php/Iwd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 工具连接

可以使用ping命令检查网络连接：
``` bash
# ping archlinux.org
```

### 更新系统时间
``` bash
# timedatectl set-ntp true
```

### 建立硬盘分区
建议使用UEFI+[GPT](https://wiki.archlinux.org/index.php/Partitioning_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#GUID_%E5%88%86%E5%8C%BA%E8%A1%A8)引导分区方案，并以此为例：

* [安装](https://wiki.archlinux.org/index.php/Help:Reading#Installation_of_packages)软件包[gptfdisk](https://archlinux.org/packages/?name=gptfdisk)
* 使用[gdisk](https://wiki.archlinux.org/index.php/GPT_fdisk_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))分区

|挂载点|分区|分区类型|分区大小|
|---|---|---|---|
|/mnt/boot 或 /mnt/efi|/dev/efi_system_partition（efi 系统分区）	|EFI 系统分区|至少 260 MiB|
|[SWAP]|/dev/swap_partition（交换空间分区）|	Linux swap (交换空间)|大于 512 MiB|
|/mnt|	/dev/root_partition（根分区）|Linux x86-64 根目录 (/)|剩余空间|
#### 格式化分区
``` bash
# mkfs.vfat /dev/efi_system_partition（efi系统分区）
# mkfs.ext4 /dev/root_partition（根分区）
# mkswap /dev/swap_partition（交换空间分区）
```
#### 挂载分区
``` bash
# mount /dev/root_partition /mnt
# swapon /dev/swap_partition（交换空间分区）
```

### 安装
使用 pacstrap 脚本，安装 base 软件包和 Linux 内核以及常规硬件的固件：
``` bash
# pacstrap /mnt base linux linux-firmware
```
### 配置系统

#### Fstab
用以下命令生成 fstab 文件 (用 -U 或 -L 选项设置UUID 或卷标)：
``` bash
# genfstab -U /mnt >> /mnt/etc/fstab
```

#### Chroot
Change root 到新安装的系统：
``` bash
# arch-chroot /mnt
```

#### 时区
设置时区：
``` bash
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
运行 hwclock(8) 以生成 /etc/adjtime：
``` bash
# hwclock --systohc
```
#### 本地化
编辑/etc/locale.gen 然后移除需要的 地区 前的注释符号 #。
接着执行 locale-gen 以生成 locale 信息：
``` bash
# locale-gen
```
然后创建 locale.conf(5) 文件，并 编辑设定 LANG 变量，比如：
``` bash
/etc/locale.conf
LANG=en_US.UTF-8
```
#### 网络配置
创建 hostname 文件:
``` bash
/etc/hostname
myhostname
```
添加对应的信息到 hosts(5):
``` bash
/etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

### 安装引导程序
``` bash
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

### 重启
```bash
# exit
# reboot
```



