# My Arch Linux

## 写在前面

在现在日常使用archlinux的过程中目前遇到的痛点：

- `QQ`、`微信`、`钉钉` 等国产社交软件体验不尽人意。

- `Timeshift` 在`DWM` 桌面环境下暂时无法正常打开UI界面

## 一、Arch Linux安装

### 1，联网设置

由于`reflector`会自动更新mirrors，会删除中国源，因此提前禁用。

```bash
systemctl stop reflector.service
```

使用 `iwctl` 连接 `wifi`

```bash
# 进入iwctl模式
iwctl
# 列出网卡设备
device list
# 让网卡wlan0进行扫描
station wlan0 scan
# 查看扫描出的设备名称
station wlan0 get-networks
# 连接wifi并输入密码
station wlan0 connect XXXX
# 退出iwctl
exit
# 测试是否ping通
ping baidu.com
```

### 2，更新系统时钟

使用`timedatectl`命令同步`网络时钟`

```bash
timedatectl set-ntp true
```

使用`timedatectl`命令检查一下

```bash
timedatectl status
```

### 3，更换国内源

用`vim`打开编辑`mirrorlist`文件，找到国内的源，例如`清华源` 放在开头。

```bash
vim /etc/pacman.d/mirrorlist
```

用`pacman -Sy` 更新

### 4，分区和格式化

用`lsblk`命令查看分区

```bash
lsblk
```

用`cfdisk`命令进入图形化界面分区（以nvme1为例）

```bash
cfdisk /dev/nvmen1
```

建议的分区大小如下：

- EFI启动分区：1G

- swap交换分区：16G

- 根目录：100G

- /home 目录：剩余

用`mkfs.fat`命令格式化`EFI分区`所在的磁盘

```bash
mkfs.fat -F32 /dev/nvmen1p1
```

用`mkswap`和`swapon`命令格式化`swap分区`所在的磁盘

```bash
mkswap /dev/nvmen1p2
swapon /dev/nvmen1p2
```

用`mkfs`命令格式化`根目录`和`home`分区所在的磁盘

```bash
mkfs.ext4 /dev/nvmen1p3
mkfs.ext4 /dev/nvmen1p4
```

### 5，挂载磁盘

```bash
mount /dev/nvmen1p3 /mnt
mkdir /mnt/home
mkdir /mnt/boot
mount /dev/nvmen1p4 /mnt/home
mount /dev/nvmen1p1 /mnt/boot
df -h # 查看挂载情况
```

### 6，安装系统

用`pasctrap`命令安装`archlinux`的基本框架

```bash
pasctrap /mnt linux linux-firmware base base-devel sudo
```

用`genfstab`命令生成`fstab`，使得系统开机自动挂载磁盘

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

利用`arch-chroot`切换到安装好的系统中

```bash
arch-chroot /mnt
```

编辑 `/etc/hostname` 文件设置主机名（这里设置为Nono）

```bash
vim /etc/hostname
```

```vim
# /etc/hostname
Nono
```

编辑`/etc/hosts` 设置hosts

```vim
127.0.0.1    localhost
::1          localhost
127.0.0.1    Nono.localdomain    Nono
```

设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

同步硬件时钟

```bash
hwclock --systohc
```

编辑`/etc/locale.gen`文件设置locale

```bash
vim /etc/locale.gen
```

找到`en_US.UTF-8`和`zh_CN.UTF-8 UTF-8`行并删除前面的注释

`locale-gen`生成locale

```bash
locale-gen
```

下面这里如果设置成中文开机会出现方块，中文环境配置会在后面记录

```bash
echo 'LANG=en_US.UTF-8' > /etc/locale.conf 
```

设置`root`账户密码

```bash
passwd root
```

创建新用户`su`，并设置密码

```bash
useradd -m -G wheel -s /bin/bash su
passwd su
```

修改`/etc/sudoers` 文件赋予新用户`sudo`权限

```bash
vim /etc/sudoers
```

删除`%wheel ALL=(ALL:ALL)ALL`行注释

安装`微码`、`引导程序`、`网络`等工具

```bash
pacman -S efibootmgr amd-ucode os-prober neovim dhcpcd networkmanager zsh
```

安装`引导`到`EFI分区`

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

生成`grub引导文件`

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

打开网络服务

```bash
systemctl start dhcpcd
systemctl enable dhcpcd
systemctl start NetworkManager
systemctl enable NetworkManager
```

退出`archlinux`

```bash
exit
```

卸载`挂载`

```bash
umount -R
```

重启系统

```bash
reboot
```

archlinux的基础安装到此结束，下面进行一些进阶安装。

## 二、Arch Linux 进阶安装

### 1，添加archlinuxcn源和开启32位库支持

确保系统保持最新

```bash
sudo pacman -Syyu
```

编辑`/etc/pacman.conf`文件

```bash
vim /etc/pacman.conf
```

去掉`[multilib]`一节中两行的注释，来开启32位库支持,并在文档的结尾处加入下面的文字，来添加`archlinuxcn源`。

```vim
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

### 2，开启终端颜色显示及糖豆人配置

编辑`/etc/pacman.conf`文件

```bash
vim /etc/pacman.conf
```

取消`Color`行注释，并添加`ILoveCandy`行

更新源

```bash
sudo pacman -Syy
```

### 3，安装签名

```bash
sudo pacman -S archlinuxcn-keyring
```

### 4，安装yay包管理工具

```bash
sudo pacman -S yay
```

### 5，DWM的安装

#### 5.1，前置安装

安装`xorg`组件

```bash
sudo pacman -S xorg-server
sudo pacman -S xorg-xorg-apps
sudo pacman -S xorg-xinit
```

先安装一个多种语言的字体，避免不能正常显示

```bash
sudo pacman -S noto-fonts-cjk
```

安装`w3m`终端浏览器

```bash
sudo pacman -S w3m
```

#### 5.2，正式安装

用`w3m`浏览器访问`suckless`官网并拉取源码

```bash
w3m suckless
```

解压

```bash
tar -zxvf dwm-6.2
```

修改`config.mk`文件

```bash
X11INC = /usr/X11R6/include   ---修改为---> X11INC = /usr/include/X11
X11LIB = /usr/X11R6/lib       ---修改为---> X11INC = /usr/include/X11
```

编译安装

```bash
sudo make clean install
```

编辑`.xinitrc`文件

```bash
vim .xinitrc
```

插入

```bash
exec dwm
```

#### 5.3，安装Alacritty

##### 5.3.1，基础安装

下载alacritty

```bash
sudo pacman -S alacritty
```

编辑`config.h`文件

```vim
static const char *termcmd[] = { "alacritty",NULL }
```

编译安装

```bash
sudo make clean install
```

##### 5.3.2，美化

Import the scheme in `alacritty.yaml` file:

Clone the repository:

`git clone https://github.com/eendroroy/alacritty-theme.git ~/.alacritty-colorscheme`

And add the below line into `alacritty.yaml`:

```yaml
import:  - ~/.alacritty-colorscheme/themes/{scheme_name}.yaml
```

Here，I choose `nord`

#### 5.4，安装Rofi

##### 5.4.1，基础安装

下载`rofi`

```bash
sudo pacman -S rofi
```

编辑`config.h`文件

```vim
static const char *termcmd[] = { "rofi", "-show", "run", NULL }
```

 编译安装

```bash
sudo make clean install
```

##### 5.4.2，美化

**Everything here is created on rofi version : `1.7.4`**    

First,Make sure you have the same (stable) version of rofi installed.

```bash
sudo pacman -S rofi
```

Then，Clone this repository

```bash
git clone --depth=1 https://github.com/adi1090x/rofi.git
```

Change to cloned directory and make `setup.sh` executable

```bash
cd rofi
chmod +x setup.sh
```

Run `setup.sh` to install the configs

```bash
./setup.sh


[*] Installing fonts...
[*] Updating font cache...

[*] Creating a backup of your rofi configs...
[*] Installing rofi configs...
[*] Successfully Installed.
```

That's it,These themes are now installed on your system.

###### Launchers

**`Change Style` :** Edit `~/.config/rofi/launchers/type-X/launcher.sh` script and edit the following line to use the style you like.

```bash
theme='style-6'
```

**`Change Colors` :** Edit `~/.config/rofi/launchers/type-X/shared/colors.rasi` file and edit the following line to use the color-scheme you like.

```bash
@import "~/.config/rofi/colors/nord.rasi"
```

*Colors in `type-5`, `type-6` and `type-7` are hard-coded (based on image colors) and can be changed by editing the respective **`style-X.rasi`** file.*

Here,I choose `style 6`,and color `Nord` .

###### Another:[GitHub - adi1090x/rofi: A huge collection of Rofi based custom Applets, Launchers &amp; Powermenus.](https://github.com/adi1090x/rofi)

#### 5.5，dwm补丁

拉取补丁

```bash
wget https://dwm.suckless.org/patches/alpha/dwm-alpha-6.4.diff
wget https://dwm.suckless.org/patches/autostart/dwm-autostart-20210120-cb3f58a.diff
wget https://dwm.suckless.org/patches/noborder/dwm-noborderfloatingfix-6.2.diff
wget https://dwm.suckless.org/patches/scratchpad/dwm-scratchpad-20221102-ba56fe9.diff
wget https://dwm.suckless.org/patches/vanitygaps/dwm-vanitygaps-20200610-f09418b.diff
```

打补丁

```bash
patch < <补丁文件>
```

#### 5.6，配置autostart.sh

[dwm 美化 - masimaro - 博客园](https://www.cnblogs.com/lanuage/p/15970577.html)

### 6，系统软件安装

使得系统可以识别`NTFS`格式的硬盘

```bash
sudo pacman -S ntfs-3g
```

一些声音固件

```bash
sudo pacman -S alsa-utils alsa-firmware alsa-ucm-conf pulseaudio pulseaudio-bluetooth pulseaudio-alsa cups sof-firmware
```

浏览器安装

```bash
yay -S google-chrome
```

*校园网如没有自动弹出登录页，手动输入网址:`https://portal2.cjlu.edu.cn`*

终端文件管理器

```bash
sudo pacman -S ranger
```

### 7，输入法及中文环境安装配置

安装`fcitx5`输入法

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```

编辑`/etc/environment`

```bash
sudo vim /etc/environment
```

添加下列`vk`，不需要`export`

```vim
GTK_IM_MODULE=fcitx5
QT_IM_MODULE=fcitx5
XMODIFIERS=@im=fcitx5
INPUT_METHOD=fcitx5
SDL_IM_MODULE=fcitx5
GLFW_IM_MODULE=ibus
```

编辑`.xinitrc`文件，插入

```vim
# 加载中文环境
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
export LC_CTYPE=en_US.UTF-8

# 启动fcitx5
fcitx5 &
```

### 8，安装更多字体

安装图标字体（查询网站：[Nerd Fonts - Iconic font aggregator, glyphs/icons collection, &amp; fonts patcher](https://www.nerdfonts.com/cheat-sheet)）

```bash
pacman -S nerd-fonts-complete
```

安装更多字体

```bash
pacman -S ttf-symbola ttf-nerd-fonts-symbols-2048-em noto-fonts-emoji nerd-fonts-complete adobe-source-code-pro-fonts nerd-fonts-fira nerd-fonts-fira-code ttf-fira-code wqy-microhei wqy-zenhei
```

安装完成后执行

```bash
fc-cache -vf
```

### 9，壁纸软件

安装`feh`

```bash
sudo pacman -S feh
```

编辑`.xinitrc`

```bash
vim .xinitrc
```

插入

```vim
feh -g 640x480 -d -S filename /path/to/directory
```

### 10，Picom

```bash
sudo pacman -S picom 
```

### 11，Neovim

https://zhuanlan.zhihu.com/p/434731678

https://zhuanlan.zhihu.com/p/438380547

https://zhuanlan.zhihu.com/p/438382701

https://zhuanlan.zhihu.com/p/439574087

https://zhuanlan.zhihu.com/p/440349051

https://www.bilibili.com/video/BV17A411D7AN/?spm_id_from=333.999.0.0

### 12，StatusBar

### 13，Timeshift

### 14，KVM

## 疑难

安装`wmname`

```bash
pacman -S wmname
```

编辑`.xinitrc`文件

```bash
export _JAVA_AWT_WM_NONREPARENTING=1
export AWT_TOOLKIT=MToolkit
wmname LG3D
```
