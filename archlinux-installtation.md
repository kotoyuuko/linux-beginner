# Arch Linux Installtation

### 准备工作

首先，准备一张你参加活动得到的 Ubuntu 或者 Debian，或者其他什么发行版的安装光盘，是的它有包装很漂亮，哦别急着找光驱，找个打火机把它烧掉，以示决心。

洗个手，沐浴更衣，斋戒三天，挑选良辰吉日，面向紫禁城方向摆好计算机，准备迎接挑战。

在开始之前，请在心中默念三遍:

    Arch Linux 是世界上最好的发行版，我一定能掌握它！

### 制作安装 U 盘

首先，你得有个 U 盘，备份好里面的数据。

随便去个镜像站，下载 Arch Linux 的安装 iso，然后把你 U 盘插到电脑上。

如果你之前已经装过其他的 Linux，先运行 `lsblk` 找到你的 U 盘的标识符。(这里用 `/dev/sdb` 举例)，然后运行：

    dd bs=4M if=/path/to/archlinux.iso of=/dev/sdb status=progress && sync

如果你之前是个 Windows 用户，来 [这里](https://sourceforge.net/p/usbwriter/wiki/Documentation/) 下载一个名叫 USBWriter 的软件，打开软件按照提示操作即可。

### 从 Arch Linux 安装 U 盘启动

这一步没啥好说的呢，从网上查查你的电脑怎么进 BIOS 或 UEFI 设置界面，去改一下启动顺序就好了。

成功启动后，你会看到一个以 root 用户运行的 zsh shell。

### 分区及格式化

既然你已经下定决心要用 Arch Linux 了，不妨把你电脑里原来装的系统删掉吧。还是用 `lsblk` 查看你需要安装到的硬盘的标识符(这里用 `/dev/sda` 举例)，运行：

UEFI:

    parted /dev/sda
    (parted) mklabel gpt
    (parted) mkpart ESP fat32 1M 513M
    (parted) set 1 boot on
    (parted) mkpart primary ext4 513M 40.5G
    (parted) mkpart primary ext4 40.5G 100%
    (parted) quit
    mkfs.fat -F32 /dev/sda1
    mkfs.ext4 /dev/sda2
    mkfs.ext4 /dev/sda3

BIOS:

    parted /dev/sda
    (parted) mklabel msdos
    (parted) mkpart primary ext4 1M 513M
    (parted) set 1 boot on
    (parted) mkpart primary ext4 513M 40.5G
    (parted) mkpart primary ext4 40.5G 100%
    (parted) quit
    mkfs.ext4 /dev/sda1
    mkfs.ext4 /dev/sda2
    mkfs.ext4 /dev/sda3

然后就没有然后了，分区已经成功并格式化完成。

### 连接到互联网

为了省事，这里还是推荐直接连接到一个有互联网访问权限的有线网。

无线网络直接运行 `wifi-menu` 选择无线网络即可

### 更新系统时间

    timedatectl set-ntp true

### 挂载硬盘分区

UEFI:

    mount /dev/sda2 /mnt
    mkdir -p /mnt/boot/efi
    mkdir -p /mnt/home
    mount /dev/sda1 /mnt/boot/efi
    mount /dev/sda3 /mnt/home

BIOS:

    mount /dev/sda2 /mnt
    mkdir -p /mnt/boot
    mkdir -p /mnt/home
    mount /dev/sda1 /mnt/boot
    mount /dev/sda3 /mnt/home

### 选择安装镜像

编辑 `/etc/pacman.d/mirrorlist` 文件，在文件头部写入你的首选镜像服务器，例如：

    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch

你可以选用你喜欢的编辑器，不过新手还是推荐用 `nano`，简单实用。

### 安装基本系统

    pacstrap /mnt base base-devel

### 生成 fstab

    genfstab -U /mnt >> /mnt/etc/fstab

### 进入 chroot 环境

    arch-chroot /mnt /bin/bash

### 设置时区

    rm /etc/localtime
    ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    hwclock --systohc --utc

### 选择 locale

编辑 `/etc/locale.gen` 文件，去掉你需要的 locale 前面的 `#` 符号，保存退出，执行：

    locale-gen
    echo LANG=en_US.UTF-8 > /etc/locale.conf

### 设置主机名

    echo myhostname > /etc/hostname

### 网络配置

同上，再次设置网络，有线网直接使用 `ip link` 查看当前使用的网络接口名称(这里用 `eth0` 举例)，然后执行：

    systemctl enable dhcpcd@eth0.service
    pacman -S networkmanager network-manager-applet
    pacman -S iw dialog net-tools

### 设置 root 密码

运行：

    passwd root

然后输入两次密码即可，注意这里密码是不显示的。

### 安装引导程序

UEFI:

    pacman -S grub efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
    grub-mkconfig -o /boot/grub/grub.cfg

BIOS:

    pacman -S grub
    grub-install --target=i386-pc /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg

### 退出 chroot 环境并重启

    exit
    umount -R /mnt
    reboot

到这里，你已经获得了一个可以启动的干净的 Arch Linux 基本系统！

### 建立普通用户

    useradd -m -g users -G wheel -s /bin/zsh username
    passwd username

### 授予普通用户 sudo 权限

编辑 `/etc/sudoers` 文件，找到下面这行：

    # %wheel ALL=(ALL) ALL

去掉前面的 `#` 注释即可。

### 安装桌面环境

如果你想装 GNOME：

    pacman -S xorg-xinit xorg-server xorg-twm xterm
    pacman -S gnome gnome-extra gnome-tweak-tool
    systemctl enable gdm.service

其他的这里就先不写了，以后再更新～

### 安装显卡驱动：

    pacman -S nvidia

### 配置 archlinuxcn 软件源

打开 `/etc/pacman.conf` 文件，在文件末尾加入：

    [archlinuxcn]
    SigLevel = Optional TrustedOnly
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

然后运行：

    pacman -S archlinuxcn-keyring yaourt

### 重启进入桌面环境

安装到这里，系统已经可以进入桌面环境啦～
重启后登录即可进入默认的桌面环境。

### 安装常用软件

Google Chrome:

    pacman -S google-chrome

Sogou Pinyin:

    pacman -S fcitx fcitx-configtool fcitx-im fcitx-sogoupinyin

在 `/etc/environment` 文件中写入下面的信息：

    XIM=fcitx
    XIM_PROGRAM=fcitx
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx

ss-qt5:

    pacman -S shadowsocks-qt5

Netease Cloud Music:

    pacman -S netease-cloud-music

VLC:

    pacman -S vlc

ProxyChains:

    pacman -S proxychains

### 常用字体

 - wqy-zenhei
 - wqy-microhei
 - ttf-dejavu
 - ttf-droid
 - cantarell-fonts
 - adobe-source-code-pro-fonts
 - adobe-source-han-sans-cn-fonts
 - adobe-source-han-serif-cn-fonts
 - noto-fonts
 - noto-fonts-emoji

### GNOME 扩展

通过 [gnome-shell-extensions](https://extensions.gnome.org/) 可以很轻松的安装各种 GNOME 扩展。

这里列出我常用的 GNOME 扩展：

 - Alternatetab —— 增强 Alt + Tab 的功能
 - Battery status —— 显示电池电量的百分比
 - Dash to Dock —— 将左侧的 Dash 变成一个 Dock 栏，我一般会把它放在底部
 - Hide workspace thumbnails —— 隐藏 Overview 视图右边的工作区栏
 - Media player indicator —— 显示音乐播放器的状态
 - Netspeed ——在顶栏上显示网速
 - Removable drive menu —— 显示连接到电脑的usb设备
 - User themes —— 用来启用自定义的shell主题
 - Workspace indicator —— 在顶栏显示当前示工作区的序号

### GNOME 主题

    pacman -S arc-gtk-theme

### 桌面美化

    pacman -S gnome-tweak-tool

然后找到 `Tweak Tool` 这个程序，就可以进行详细的设置了。

### 完成

没错，这就完了～

