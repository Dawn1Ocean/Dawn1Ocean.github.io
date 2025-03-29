---
title: KDE Tricks (1) —— 快捷键切换下一张壁纸
date: 2023-12-17 18:13:30
categories:
- Linux 配置
- Shell
tags:
- KDE
- Shell
---

# 需求
KDE 自带的环境可以实现桌面定时切换下一张壁纸，但是想要手动切换，就只能右击桌面 -> 点击 `Next Wallpaper Image`。
![自带的 SlideShow 模式](KDE-Tricks-1/wall.png "自带的 SlideShow 模式")

经过查询，并无命令负责此动作，因此打算通过脚本实现，并绑定快捷键。

在 [SuperUser](https://superuser.com/questions/488232/how-to-set-kde-desktop-wallpaper-from-command-line) 上找到一个办法，可以通过 `QDBus` 直接写入需要更改的图片，将随机选择图片部分和定时部分均交由脚本处理。

# 操作
由于直接采用写入图片的方式，KDE 的壁纸模式会被切换为 `Image`，因此同时需要接管 `SlideShow` 模式：定时自动切换下一张壁纸。
## 接管定时功能
新建一个 `.sh` 脚本，写入如下内容：
```Bash
while true
do

# //////////// 配置变量 ////////////

# 两次切换时间间隔，单位为秒
sleeptime=900
# 壁纸文件夹路径
location="/home/myusername/Pictures"

# ////////////////////////////////////////

array=($(ls $location*)) # 列出文件内容
( IFS=$'\n'; echo "${array[*]}" ) # echo 在终端里，后台运行可注释掉

size=${#array[@]}
index=$(($RANDOM % $size)) 
wallpaper=${array[$index]} # 随机选择

echo # 显示所有文件信息，后台运行可注释掉
echo " || SELLECTED WALLPAPER = "$wallpaper # 显示选择图片信息，后台运行可注释掉

dbus-send --session --dest=org.kde.plasmashell --type=method_call /PlasmaShell org.kde.PlasmaShell.evaluateScript 'string:
var Desktops = desktops();                                                                                                                       
for (i=0;i<Desktops.length;i++) {
        d = Desktops[i];
        d.wallpaperPlugin = "org.kde.image";
        d.currentConfigGroup = Array("Wallpaper",
                                    "org.kde.image",
                                    "General");
        d.writeConfig("Image", "file://'''$location'''/'${wallpaper}'");
}' # 核心语句，使用 dbus-send
sleep $sleeptime # 定时

done
```
`chmod +x` 后，可以将脚本添加到 KDE 的 `Autostart`，实现无感切换。


## 手动切换脚本
只需要略微修改定时脚本即可。
```Bash
# //////////// 配置变量 ////////////

# 壁纸文件夹路径
location="/home/myusername/Pictures"

# ////////////////////////////////////////

array=($(ls $location*)) # 列出文件内容
( IFS=$'\n'; echo "${array[*]}" ) # echo 在终端里，后台运行可注释掉

size=${#array[@]}
index=$(($RANDOM % $size)) 
wallpaper=${array[$index]} # 随机选择

echo # 显示所有文件信息，后台运行可注释掉
echo " || SELLECTED WALLPAPER = "$wallpaper # 显示选择图片信息，后台运行可注释掉

dbus-send --session --dest=org.kde.plasmashell --type=method_call /PlasmaShell org.kde.PlasmaShell.evaluateScript 'string:
var Desktops = desktops();                                                                                                                       
for (i=0;i<Desktops.length;i++) {
        d = Desktops[i];
        d.wallpaperPlugin = "org.kde.image";
        d.currentConfigGroup = Array("Wallpaper",
                                    "org.kde.image",
                                    "General");
        d.writeConfig("Image", "file://'''$location'''/'${wallpaper}'");
}' # 核心语句，使用 dbus-send
```
可以使用 `alias` 将脚本变为便于输入的命令。

再进入 KDE 设置中的 `Custom Shortcuts` 部分，设置自己喜欢的快捷键即可。
![设置快捷键](KDE-Tricks-1/shortcut.png "设置快捷键")
