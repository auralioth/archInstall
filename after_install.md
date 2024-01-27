# This is a collection of daily problems and more settings.

<!--toc:start-->

- [This is a collection of daily problems and more settings.](#this-is-a-collection-of-daily-problems-and-more-settings)
  - [Pacman](#pacman)
  - [Power Management](#power-management)
    - [Hibernate](#hibernate)
      - [Configure the initramfs](#configure-the-initramfs)
      - [Required kernel parameters](#required-kernel-parameters)
  - [GRUB](#grub)
  - [Firefox](#firefox)
  <!--toc:end-->

## Pacman

To make paru more beautiful,recommend opening the `Color` option

```bash
$ sudo nvim /etc/pacman.conf
```

Then uncomment the `Color` option

## Power Management

### Hibernate

Refer to [Power management/Suspend and hibernate](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation) for latest settings.

#### Configure the initramfs

```bash
/etc/mkinitcpio.conf
# add the resumn after udev
HOOKS=(base udev autodetect keyboard modconf block filesystems resume fsck)
# regenerate the initramfs
mkinitcpio -P
```

#### Required kernel parameters

```bash
# add kernel parameters
/etc/default/grub
resume=UUID=swap_device_uuid

# regenerate grub.cfg
grub-mkconfig -o /boot/grub/grub.cfg

# reboot
reboot
```

## GRUB

To make grub more beautiful , I have installed `grub-theme-vimix`.

To let it take effect:

```bash
/etc/default/grub

GRUB_THEME="/usr/share/grub/themes/Vimix/theme.txt"
```

## Firefox

About firefox [Hardware video acceleration](https://wiki.archlinux.org/title/Firefox#Hardware_video_acceleration)

```bash
about:config

media.ffmpeg.vaapi.enabled = true
media.av1.enabled = false       # my graphics don't support av1.
```

## Use integrated graphics only

[See Hybrid graphics#Fully power down discrete GPU](https://wiki.archlinux.org/title/Hybrid_graphics#Fully_power_down_discrete_GPU)

**Using udev rules**

## NOPASSWD sudo

For `sudo dae reload` and `sudo dae suspend` to run without password, add the following with `visudo`

```bash
$ sudo EDITOR=nvim visudo

user ALL=(ALL:ALL) NOPASSWD: /usr/bin/dae reload, /usr/bin/suspend
```
