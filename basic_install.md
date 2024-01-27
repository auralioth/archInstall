# Arch Linux 基础安装

<!--toc:start-->

- [Arch Linux 基础安装](#arch-linux-基础安装)
  - [0.禁用 reflector](#0禁用-reflector)
  - [1.再次确保是否为 UEFI 模式](#1再次确保是否为-uefi-模式)
  - [2.连接网络](#2连接网络)
  - [3.更新系统时钟](#3更新系统时钟)
  - [4.分区](#4分区)
  - [5.格式化并创建子卷](#5格式化并创建子卷)
    - [5.1 格式化](#51-格式化)
    - [5.2 创建子卷](#52-创建子卷)
  - [6.挂载](#6挂载)
  - [7.镜像源的选择](#7镜像源的选择)
  - [8.安装系统](#8安装系统)
  - [9.生成 fstab 文件](#9生成-fstab-文件)
  - [10.change root](#10change-root)
  - [11.时区设置](#11时区设置)
  - [12.设置 Locale 进行本地化](#12设置-locale-进行本地化)
  - [13.设置主机名](#13设置主机名)
  - [14.为 root 用户设置密码](#14为-root-用户设置密码)
  - [15.安装微码](#15安装微码)
  - [16.安装引导程序](#16安装引导程序)
  - [17.完成安装](#17完成安装)
  - [18.进入系统](#18进入系统)
  <!--toc:end-->

先安装最基础的无图形化 ArchLinux 系统。详见[官方安装指南](https://wiki.archlinux.org/index.php/Installation_guide)

开始安装之前可以参考[安装前的准备](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/rookie/archlinux_pre_install)，做好准备工作

## 0.禁用 reflector

```bash
systemctl stop reflector.service
```

## 1.再次确保是否为 UEFI 模式

```bash
ls /sys/firmware/efi/efivars
```

若输出了一堆东西，即 efi 变量，则说明已在 UEFI 模式。否则需确认启动方式是否为 UEFI

## 2.连接网络

- **非校园网**：有线连接时，直接插入网线即可。

- **无线连接**：进行如下操作进行网络连接。

- **校园网认证**：建议手机热点或其他wifi

```bash
iwctl                                           # 执行iwctl命令，进入交互式命令行
device list                                     # 列出设备名，比如无线网卡看到叫 wlan0
station wlan0 scan                              # 扫描网络
station wlan0 get-networks                      # 列出网络 比如想连接YOUR-WIRELESS-NAME这个无线
station wlan0 connect YOUR-WIRELESS-NAME        # 进行连接 输入密码即可
exit                                            # 成功后exit退出

# 测试网络的操作
ping www.gnu.org
```

---

**如果**不能正常连接网络，首先确认系统已经启用网络接口[[1]](https://wiki.archlinux.org/index.php/Network_configuration/Wireless#Check_the_driver_status)。

```bash
ip link                 # 列出网络接口信息，如不能联网的设备叫wlan0
ip link set wlan0 up    # 比如无线网卡看到叫 wlan0
```

**如果**随后看到类似`Operation not possible due to RF-kill`的报错，继续尝试`rfkill`命令来解锁无线网卡。

```bash
rfkill unblock wifi
```

**如果**还是不能，则检查是否分配 ip 地址，如果没有，则使用 dhcpcd

```bash
dhcpcd
```

## 3.更新系统时钟

```bash
timedatectl set-ntp true    # 将系统时间与网络时间进行同步
timedatectl status          # 检查服务状态
```

## 4.分区

这是一个在我个人电脑上实行的方案。此步骤会清除磁盘中全部内容

- /boot EFI 分区：800MB（由电脑厂商或 Windows 决定，无需再次创建）
- Swap 分区：电脑实际运行内存(16G)（设置这个大小是为了配置休眠准备）
- / 根目录：剩下所有（和用户主目录在同一个 Btrfs 文件系统上）
- /home 用户主目录：剩下所有（和根目录在同一个 Btrfs 文件系统上）

将磁盘转换为 `gpt` 类型，假设磁盘名称为 `sda`。如果使用 `NVME` 的固态硬盘，你看到的磁盘名称可能为 `nvme0n1`。

```bash
lsblk                       # 显示分区情况 找到想安装的磁盘名称
parted /dev/sda             # 执行parted，进入交互式命令行，进行磁盘类型变更
(parted)mktable             # 输入mktable
New disk label type? gpt    # 输入gpt 将磁盘类型转换为gpt 如磁盘有数据会警告，输入yes即可
quit                        # 最后quit退出parted命令行交互

```

使用 cfdisk 命令对磁盘分区。

一般建议将 EFI 分区设置为磁盘的第一个分区，其中 EFI 分区选择`EFI System`类型，Swap分区选择`Linux Swap`类型，其余两个分区选择`Linux filesystem`类型。

**分区之后选择`write`写入磁盘再退出。**

```bash
cfdisk /dev/sda   # 来执行分区操作,分配各个分区大小，类型
fdisk -l          # 分区结束后， 复查磁盘情况
```

## 5.格式化并创建子卷

### 5.1 格式化

用`mkfs.btrfs`命令格式化根分区，用`mkfs.fat`命令格式化 EFI 分区，用`mkswap`格式化Swap分区。如下命令中的 sdax 中，x 代表分区的序号。
格式化命令要与上一步分区中生成的分区名字对应才可以。

磁盘若事先有数据，会提示: `proceed any way?` 按 `y` 回车继续即可。

```bash
mkfs.fat -F 32 /dev/sdax                # 格式化efi分区
mkswap /dev/sdax                        # 格式化Swap分区
mkfs.btrfs -L arch-btrfs /dev/sdax      # 格式化Btrfs文件系统
```

- `-L`选项后指定该分区的 LABLE，这里以 `arch-btrfs` 为例，也可以自定义，但不能使用特殊字符以及空格，且最好有意义

### 5.2 创建子卷

先将`Btrfs`分区挂载到`/mnt`下

```bash
mount -o compress=zstd /dev/sdxn /mnt
```

创建子卷

```bash
btrfs subvol create /mnt/@
btrfs subvol create /mnt/@home
btrfs subvol create /mnt/@var_tmp
btrfs subvol create /mnt/@var_cache
btrfs subvol create /mnt/@var_log
btrfs subvol create /mnt/@docker
```

为部分不需要写时复制的子卷设置相关属性

```bash
chattr +C /mnt/@var_cache
```

取消挂载

```bash
umount /mnt
```

如何决定需要创建什么子卷？

- 首先，推荐将 rootfs 和 /home 文件夹分别存放在不同子卷（@和 @home）中。

- /var/tmp 属于临时文件文件夹，推荐为其单独分一个子卷（@var_tmp）。不需要为 /tmp 创建子卷。

- /var/log 属于日志文件文件夹，可以为其单独分一个子卷（@var_log）。

- 建议为 /var/cache 单独创建一个子卷（@var_cache），并且根据需求，可考虑为该子卷关闭写时复制。

- 部分快照软件可能会用到 /.snapshots 目录。如果你打算使用的快照软件使用到了这个目录，则需要为此目录创建子卷 @snapshots。

- 假定你现在要对 rootfs 子卷执行快照操作，一些你觉得不应该纳入 rootfs 快照的目录，比如 Docker 数据文件夹（/var/lib/docker，@docker）和 libvirt 文件夹（/var/lib/libvirt，@libvirt）等。

- 不要为整个 /var 目录单独分一个子卷。

## 6.挂载

挂载是有顺序的，先挂载根目录，再挂载 EFI 分区，最后交换分区。

挂载分区，注意这里的compress和上面选择的压缩算法一致

```bash
mount --mkdir /dev/sdax -o compress=zstd,subvol=@ /mnt
mount --mkdir /dev/sdax -o subvol=@home /mnt/home
mount --mkdir /dev/sdax -o subvol=@var_tmp /mnt/var/tmp
mount --mkdir /dev/sdax -o subvol=@var_cache /mnt/var/cache
mount --mkdir /dev/sdax -o subvol=@var_log /mnt/var/log
mount --mkdir /dev/sdax -o subvol=@docker /mnt/var/lib/docker

mount --mkdir /dev/sdax /mnt/boot

swapon /dev/sdxn
```

## 7.镜像源的选择

使用如下命令编辑镜像列表：

```bash
vim /etc/pacman.d/mirrorlist
```

此时可以选择使用[Mirrorlist Generator](https://archlinux.org/mirrorlist/)去生成最新的镜像源，挑选几个非威权国家的镜像源加入到`mirrorlist`中，
可以在[Mirror Status](https://archlinux.org/mirrors/status/)上查看镜像源的状态。

例子如下:

```bash
Server = https://mirror.osbeck.com/archlinux/$repo/os/$arch
Server = https://ftp.lysator.liu.se/pub/archlinux/$repo/os/$arch
Server = https://ftp.acc.umu.se/mirror/archlinux/$repo/os/$arch
```

## 8.安装系统

必须的基础包

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware btrfs-progs    # base-devel对于AUR包的安装是必须的, btrfs-progs对于Btrfs系统必须
```

必须的功能性软件

```bash
pacstrap /mnt dhcpcd iwd neovim # 一个有线所需(iwd也需要dhcpcd) 一个无线所需 一个编辑器
```

## 9.生成 fstab 文件

fstab 用来定义磁盘分区

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

在执行 `genfstab` 之后，推荐立刻修改 `/mnt/etc/fstab`，将 `fstab` 中的 `btrfs` 挂载点的挂载选项稍作编辑，再执行 `arch-chroot` 进行后续配置操作。

推荐删除 `ssd,discard=async,space_cache=v2,subvolid=xxx` 等这些由系统自动决定的挂载选项，保留 `rw,relatime,compress=zstd:3,subvol=@xxx` 等以及其他你认为需要保留的挂载选项。

复查一下 `/mnt/etc/fstab` 确保没有错误

```bash
cat /mnt/etc/fstab
```

## 10.change root

把环境切换到新系统的/mnt 下

```bash
arch-chroot /mnt
```

为了防止安装时出现蜂鸣声，简单粗暴的卸载 pcspkr 模块，加入黑名单

```
rmmod pcspkr
echo “blacklist pcspkr” > /etc/modprobe.d/nobeep.conf
```

## 11.时区设置

设置时区，在/etc/localtime 下用/usr 中合适的时区创建符号连接。如下设置上海时区。

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

随后进行硬件时间设置，将当前的正确 UTC 时间写入硬件时间。

```bash
hwclock --systohc
```

## 12.设置 Locale 进行本地化

Locale 决定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

首先使用 neovim 编辑 /etc/locale.gen，去掉 en_US.UTF-8 所在行以及 zh_CN.UTF-8 所在行的注释符号（#）。

```bash
nvim /etc/locale.gen
```

然后使用如下命令生成 locale。

```bash
locale-gen
```

最后向 /etc/locale.conf 导入内容

```bash
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf
```

## 13.设置主机名

首先在`/etc/hostname`设置主机名

```bash
nvim /etc/hostname
```

加入想为主机取的主机名，这里比如叫 `arch`。

接下来在`/etc/hosts`设置与其匹配的条目。

```
nvim /etc/hosts
```

加入如下内容

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain    arch      # 注意这里与你的主机名对应
```

## 14.为 root 用户设置密码

```bash
passwd root
```

## 15.安装微码

```bash
pacman -S intel-ucode   # Intel
pacman -S amd-ucode     # AMD
```

## 16.安装引导程序

```bash
pacman -S grub efibootmgr            #grub是启动引导器，efibootmgr被 grub 脚本用来将启动项写入 NVRAM
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

**optional：** 接下来编辑/etc/default/grub 文件，去掉`GRUB_CMDLINE_LINUX_DEFAULT`一行中最后的 quiet 参数，同时把 log level 的数值从 3 改成 5。这样是为了后续如果出现系统错误，方便排错。

**recommend：** 同时在同一行加入 nowatchdog 参数，这可以显著提高开关机速度。

```bash
nvim /etc/default/grub
```

最后生成 GRUB 所需的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 17.完成安装

```bash
exit                # 退回安装环境
umount -R  /mnt     # 卸载新分区
reboot              # 重启
```

`注意：`重启前要先拔掉 U 盘，否则重启后还是进安装程序而不是安装好的系统。

## 18.进入系统

连接网络

```bash
systemctl start dhcpcd  # 立即启动dhcp
ping www.gnu.org        # 测试网络连接
```

若为无线连接，则还需要启动 `iwd` 才可以使用 `iwctl` 连接网络

```bash
systemctl start iwd     # 立即启动iwd
iwctl                   # 和之前的方式一样，连接无线网络
```

如果你是`archlinux`单系统，那么到此为止，一个基础的，无 UI 界面的 Arch Linux 已经安装完成了。

如果你是`arch`和`windows`双系统，那么还需要做一些工作来使得`grub`可以探测到`windows`磁盘

```bash
sudo pacman -S os-prober ntfs-3g  # os-prober用来探测，ntfs-3g识别windows的磁盘
```

启用 os-prober:默认禁用掉了，将 GRUB_DISABLE_OS_PROBER=false 前的注释符#去掉。

```bash
sudo nvim /etc/default/grub
```

生成 GRUB 所需的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

重新启动即可进入到接下来的[图形界面](<https://github.com/auryouth/archdoc/blob/master/WM(Hyprland)%26Setting.md>)安装配置。
