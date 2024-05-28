# 常用软件-Linux篇

# 常用软件-Linux篇
## 办公
* wps全家桶
> 对于arch而言，有两个源cn和国际版，国际版没有云同步功能，资源占用更低

## 音视频
* vlc mpv(视频播放器)
* ffmpeg(视频处理开源项目)
* deadbeef/spotify/(mpd + mpc/ncmpcpp) (音乐播放器/在线音乐/音乐播放器守护程序+接口/命令行GUI)
* osd-lyrics(歌词查找和展示器)
* kid3(歌曲tag编辑器)

## 文件类
* fzf(开源文件搜索工具)
* ripgrep(开源面向行的搜索器)
* pandoc(文件类型转换)
* thunar/dolphin(文件夹工具)
* ranger/yazi(命令行文件夹查看器)

## 下载
* Motrix(多线程下载工具)
* qBittorent(种子下载工具)
* aria2(命令行下载工具)

## PDF
* okular/zathura(pdf阅读软件)

## 绘图软件
* drawio

## 系统驱动
> 网络和蓝牙类驱动需要通过systemctl进行设置
### 网络
* iwd(无线网连接)
* systemd-networkd(有线网连接)
* systemd-resolved(dns服务器)
* dhcpcd(dhcp客户端)
### 蓝牙
* bluez(蓝牙驱动)
* bluez-utils(蓝牙工具)
### 触摸板
* libinput(wayland触摸板驱动)
* libinput+xf86-input-libinput(xorg触摸板驱动)
> libinput是一组接收和检测设备输入的函数库，xf86-input-libinput是其的一个包装，使其可以用于x
* libinput-gestures配置触摸板手势
* xdotool是模拟按键的小工具,主要用来配合libinput-gestures
### 打印
* cups(打印服务)
> wps打印不可用时要下载cups
### 音频
* ALSA是驱动级应用，早期并不支持多音频播放，之后集成了alsamixer工具之后可进行多音频播放
    * alsamixer是音量控制的命令行GUI程序,amixer是其命令行程序

* PulseAudio是运行在ALSA之上的多音频管理器,旨在作为应用程序和硬件设备(alsa)之间的中间件运行
    * pactl是控制其的命令行程序
    * pulsemixer是控制其的GUI和命令行程序

* PipeWire是旨在代替PulseAudio的新型音频管理器
    * 安装PipeWire-audio之后需要安装PipeWire-pulse和PipeWire-alsa来链接PulseAudio和alsa的功能
* 仍然可以使用alsa和PulseAudio的工具进行音量调节


### 显卡
* intel: mesa lib32-mesa vulkan-intel lib32-vulkan-intel(intel显卡驱动)
* nvidia: nvidia nvidia-settings lib32-nvidia-utils nvidia-prime(nvidia显卡驱动)
> nvidia-prime负责双显卡管理
>
> * prime-run %command% 使用nvida显卡运行命令
>
> * steam需要添加`__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME="nvidia" __VK_LAYER_NV_optimus="NVIDIA_only" %command%`
### 磁盘
* ntfs-3g(ntfs支持)

### 系统工具
* xprop查看窗口属性
* xev查看按键映射

## 输入法
* fcitx5/fcitx5-chinese-addons/fcitx5-pinyin-moegirl/fcitx5-material-color(输入法/中文输入/萌娘百科词库/输入法主题)

## 终端及终端工具
* fontconfig(字体管理)
* yazi/ranger(文件夹管理)
* zsh/ohmyzsh/powerlevel10k(zsh/优化/主题)
* p7zip/rar/unrar/zip/unzip/tar/Unarchiver(压缩和解压工具)
* rsync(拷贝和同步命令)
* rename(批量重命名)
* wget/curl(下载/模拟http请求)
* tldr(命令提示工具)
* playerctl(MPRIS的音频管理)
* 亮度调节
    * x11: xbacklight `xorg-xbacklight包`(使用mesa驱动的intel核显无法使用xbacklight)
    * 不依赖于平台：light/acpilight
    * 直接修改`/sys/class/backlight/intel_backlight/brightness`的值

## 图形显示(xorg)
* sddm(开机显示)
* xorg-server,xorg-setxkbmap,xorg-xsetroot,xorg-xrandr,xorg-xinit(x服务器)
* dunst/libnotify(通知服务器/通知显示)
* picom(合成器)
* Alacritty(支持GPU加速的终端)
* rofi(启动器)
* dwm(动态窗口管理器)
* slstatus(顶部bar显示)
* slock(锁屏)
* i3(平铺窗口管理器)
* feh(壁纸)
* xclip(剪贴板)
* flameshot(截图工具)

## 图形显示(wayland)
* dunst/libnotify(通知服务器/通知显示)
* Alacritty(支持GPU加速的终端)
* foot(轻量级终端)
* wofi(启动器)
* rofi-wayland(启动器)
* river(动态窗口管理器)
* sway(平铺窗口管理器)
* waybar(顶部bar显示)
* swaybg(壁纸)
* sway(平铺窗口管理器)
* wl-clipboard(剪贴板)
* flameshot/grim(截图工具)
* waylock(锁屏)
* lswt(列出当前的wayland窗口)
* kanshi(输出配置管理)

## 开发软件
### 环境管理
* docker(容器)
* nvm(nodejs和npm环境管理)
> nvm严重拖累zsh速度，可以使用fnm
* conda(python环境管理)
* Vmware/VirtualBox/EXSI/kvm+kvm-qemu-libvirt-virtmanager(虚拟机管理)
### 后端
* Navicat(数据库可视化)
* RDM(Redis可视化)
* Postman(接口测试)
* JetBrains全家桶(IDE)
### 通用
* Zeal(文档管理)
* FileZilla Client(ftp,sftp工具)
* vscode(编辑器)
* Node.js(前端管理工具)
* curl/wget(命令行下载工具)
* Copilot(人工智能代码补全)
### 代码管理
* git(代码版本管理)
* Sphinx(Python文档管理工具)
* reStructuredText(Python文档格式)


## 系统监控
* prometheus+Grafana(适合分布式，能够监控容器和云环境，支持k8s)
* zabbix(适合小规模部署，缺乏分布式扩展和可用性)
* nagios(相对更加轻量的监控软件)

## 日用软件
* clash/clash-meta/singbox(科学上网client)
* v2ray/x2ray/singbox(科学上网server)
* lattle(docker栏)
* p7zip(terminal)/ark(gui)(压缩与解压)
* osd(录屏)
* wine,winetricks(windows模拟层)
* mentohust(校园网连接)
* feh(图片查看和壁纸配置)
* flameshot(截图工具)
* syncthing(备份同步工具)
* screenkey/wshowkeys or showmethekey(xorg/wayland实时显示输入的按键)
* kvm，qemu,libvirt,virt-manager,virtio(虚拟机)
![虚拟机](/常用软件/kvm-qemu-libvirt-virtmanager.png "虚拟机架构")
* scrcpy/uxplay(安卓/IOS投屏)

## 硬件检测
* CPU-X
* GPU-Viewer
* smartmontools(硬盘信息)
* dmidecode(全面的硬件信息)

## 性能优化
* BIOS超频
* cpupower(cpu性能调整)
* intel-undervolt(cpu降压工具)
## 游戏
> 本质上是通过vulkan(一种多媒体图形接口)来运行DirectX游戏
* proton
* lutris
* dxvk(将dx转换为vulkan的转换层，使得wine可以在linux下运行3D游戏)
* mongohub(帧数显示)

