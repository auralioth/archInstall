# WM(Hyprland)的安装配置

<!--toc:start-->

- [WM(Hyprland)的安装配置](#wmhyprland的安装配置)
  - [1.确保系统为最新](#1确保系统为最新)
  - [2.准备非 root 用户](#2准备非-root-用户)
  - [3.开启 32 位支持库](#3开启-32-位支持库)
  - [4.设置 DNS](#4设置-dns)
  - [5.Hyprland 前置](#5hyprland-前置)
  - [6.配置文件恢复](#6配置文件恢复)
  <!--toc:end-->

官方文档: [安装后的工作](https://wiki.archlinux.org/index.php/General_recommendations)

## 1.确保系统为最新

确认网络连接，然后更新系统。

```bash
pacman -Syu    # 升级系统中全部包
```

## 2.准备非 root 用户

添加用户，比如新增加的用户叫 wx

```bash
useradd -m -G wheel -s /bin/bash wx  # wheel附加组可sudo，以root用户执行命令 -m同时创建用户家目录
```

设置新用户 wx 的密码

```bash
passwd wx
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

如果路由器可以自动处理 DNS,resolvconf 会在每次网络连接时用路由器的设置覆盖本机/etc/resolv.conf 中的设置，执行如下命令加入不可变标志，使其不能覆盖如上加入的配置。

```bash
sudo chattr +i /etc/resolv.conf
```

## 5.Hyprland 前置

首先进行网络配置

```bash
sudo pacman -S networkmanager

sudo systemctl disable --now iwd                                            # 立即关闭iwd,其无线连接会与NetworkManager冲突
sudo systemctl enable --now NetworkManager
nmtui                                                                       # 图形化网络配置
```

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

## 6.配置文件恢复

参考我的[dotfile](https://github.com/auryouth/archdot)
