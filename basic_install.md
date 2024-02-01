# Arch Linux 安装

>[!IMPORTANT]
>[官方安装指南](https://wiki.archlinux.org/index.php/Installation_guide) ： 禁用`Secure Boot`
>
>[双启动](https://wiki.archlinux.org/title/Dual_boot_with_Windows) ： 建议禁用`Windows`的快速启动和休眠
>
>**更多细节安装前需仔细浏览，因为Guide会不断更新**


## 分区

- /efi EFI 分区：800MB
- Swap 分区：电脑实际运行内存(16G)
- / 根目录：剩下所有 (Btrfs 文件系统）

## 分完区格式化后创建子卷

```bash
# mount -o compress=zstd /dev/sdxn /mnt

# btrfs subvol create /mnt/@
# btrfs subvol create /mnt/@home
# btrfs subvol create /mnt/@var_tmp
# btrfs subvol create /mnt/@var_cache
# btrfs subvol create /mnt/@var_log
# btrfs subvol create /mnt/@docker

# chattr +C /mnt/@var_cache

# umount /mnt
```

如何决定需要创建什么子卷？

- 首先，推荐将 rootfs 和 /home 文件夹分别存放在不同子卷（@和 @home）中。

- /var/tmp 属于临时文件文件夹，推荐为其单独分一个子卷（@var_tmp）。不需要为 /tmp 创建子卷。

- /var/log 属于日志文件文件夹，可以为其单独分一个子卷（@var_log）。

- 建议为 /var/cache 单独创建一个子卷（@var_cache），并且根据需求，可考虑为该子卷关闭写时复制。

- 部分快照软件可能会用到 /.snapshots 目录。如果你打算使用的快照软件使用到了这个目录，则需要为此目录创建子卷 @snapshots。

- 假定你现在要对 rootfs 子卷执行快照操作，一些你觉得不应该纳入 rootfs 快照的目录，比如 Docker 数据文件夹（/var/lib/docker，@docker）和 libvirt 文件夹（/var/lib/libvirt，@libvirt）等。

- 不要为整个 /var 目录单独分一个子卷。

## 挂载

挂载是有顺序的，先挂载根目录，再挂载 EFI 分区，最后交换分区。

挂载分区，注意这里的compress和上面选择的压缩算法一致

```bash
# mount --mkdir /dev/sdax -o compress=zstd,subvol=@ /mnt
# mount --mkdir /dev/sdax -o subvol=@home /mnt/home
# mount --mkdir /dev/sdax -o subvol=@var_tmp /mnt/var/tmp
# mount --mkdir /dev/sdax -o subvol=@var_cache /mnt/var/cache
# mount --mkdir /dev/sdax -o subvol=@var_log /mnt/var/log
# mount --mkdir /dev/sdax -o subvol=@docker /mnt/var/lib/docker

# mount --mkdir /dev/sdax /mnt/efi

# swapon /dev/sdxn
```

## fstab

推荐删除 `ssd,discard=async,space_cache=v2,subvolid=xxx` 等这些由系统自动决定的挂载选项，保留 `rw,relatime,compress=zstd:3,subvol=@xxx` 

## 禁止蜂鸣 

`arch-chroot /mnt`后为了防止安装时出现蜂鸣声，简单粗暴的卸载 pcspkr 模块，加入黑名单

```bash
# rmmod pcspkr
# echo “blacklist pcspkr” > /etc/modprobe.d/nobeep.conf
```

## GRUB

编辑/etc/default/grub 文件，去掉`GRUB_CMDLINE_LINUX_DEFAULT`一行中最后的 quiet 参数，把 log level 的数值从 3 改成 5

同时在同一行加入 nowatchdog 参数，这可以显著提高开关机速度。

- - -
>[!IMPORTANT]
>[安装后的工作](https://wiki.archlinux.org/index.php/General_recommendations)


## 配置文件恢复

参考我的[dotfile](https://github.com/auryouth/archdot.git)
