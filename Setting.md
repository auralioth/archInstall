# 安装后工作

<!--toc:start-->

- [安装后工作](#安装后工作)
  - [1.进入系统](#1进入系统)
  - [2.确保系统为最新](#2确保系统为最新)
  - [3.准备非 root 用户](#3准备非-root-用户)
  - [4.开启 32 位支持库](#4开启-32-位支持库)
  - [5.设置 DNS](#5设置-dns)
  - [6.网络管理](#6网络管理)
  - [7.软件源](#7软件源)
  - [8.配置文件恢复](#8配置文件恢复)
  <!--toc:end-->

官方文档: [安装后的工作](https://wiki.archlinux.org/index.php/General_recommendations)

## 1.进入系统

连接网络

```bash
systemctl start dhcpcd  # 立即启动dhcp
# 若为无线连接则还需要启动iwd
systemctl start iwd     # 立即启动iwd
```

如果是`arch`和`windows`双系统，那么还需要做一些工作来使得`grub`可以探测到`windows`磁盘

```bash
sudo pacman -Syu os-prober ntfs-3g  # os-prober用来探测，ntfs-3g识别windows的磁盘
```

启用 os-prober:默认禁用掉了，将 `GRUB_DISABLE_OS_PROBER=false` 前的注释符#去掉。

```bash
sudo nvim /etc/default/grub
```

生成 GRUB 所需的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 2.准备非 root 用户

添加用户，比如新增加的用户叫 ausosa

```bash
useradd -m -G wheel -s /bin/bash ausosa  # wheel附加组可sudo，以root用户执行命令 -m同时创建用户家目录
```

设置新用户 ausosa 的密码

```bash
passwd ausosa
```

编辑 sudoers 配置文件

```bash
EDITOR=nvim visudo  # 需要以 root 用户运行 visudo 命令
```

找到下面这样的一行，把前面的注释符号 `#` 去掉，`:wq` 保存并退出即可。

```sudoers
#%wheel ALL=(ALL:ALL) ALL
```

## 3.开启 32 位支持库

```bash
nvim /etc/pacman.conf
```

去掉[multilib]一节中两行的注释，来开启 32 位库支持。

最后:wq 保存退出，刷新 pacman 数据库

```bash
pacman -Syu
```

## 4.设置 DNS

nvim 编辑/etc/resolv.conf，删除已有条目，并将如下内容加入其中

```bash
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
nameserver 8.8.4.4
nameserver 2001:4860:4860::8844
```

加入不可变标志防止被覆盖

```bash
sudo chattr +i /etc/resolv.conf
```

## 7.软件源

添加[archlinuxcn 源](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)

```bash

# 在 /etc/pacman.conf 文件末尾添加以下两行
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch

# 安装keyring
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```

注意添加后安装 archlinuxcn-keyring 报错的话执行下列命令再安装

```bash
sudo rm -rf /etc/pacman.d/gnupg
sudo pacman-key --init
sudo pacman-key --populate
```

## 8.配置文件恢复

参考我的[dotfile](https://github.com/auryouth/archdot/tree/Hyprland)
