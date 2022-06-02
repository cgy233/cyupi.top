---
layout: autohotkey
title: Windows一键修改IP、DNS等网络配置
date: 2022-01-25 17:27:50
categories: 
- Tools
tags:
- AutoHotkey
---
# 前言

寒假在家，我姐的电脑闲置了，就拿个U盘在上面装了个 [OpenWRT](https://baike.baidu.com/item/openWRT/3528947) 作为旁路由，有些时候需要频繁切换IP等配置，觉得太麻烦了， 网上的方法...CMD命令+桌面快捷方式，emmm...我喜欢干干净净的桌面，一个图标也没有的那种，然后就发现了个神器：[AutoHotkey](https://www.autohotkey.com/)

# 环境

- Windows11
- [AutoHotkey](https://www.autohotkey.com/)

# 安装AutoHotkey

## 1.[官方下载链接](https://www.autohotkey.com/download/ahk-install.exe)

## 2.[安装教程](https://www.jianshu.com/p/4ec83b75c696)

# 创建并运行脚本

在桌面新建个文本，将下列内容复制进去，然后保存退出，将文件的后缀改成.ahk

```bash
; ---------------------------------------------------------
; 网络配置，直连或Openwrt
; 直连 Ctrl + Shift + Alt + d
!+^d::
{
    Run, *RunAs %ComSpec% /c netsh interface ip set address name="以太网" source=static addr=192.168.31.3 mask=255.255.255.0 gateway=192.168.31.1 1,,hide
    Run, *RunAs %ComSpec% /c netsh interface ip set dns name="以太网" source=static addr=192.168.31.1,,hide
        return
}
; Openwrt Ctrl + Shift + Alt + s
!+^s::
{
    Run, *RunAs %ComSpec% /c netsh interface ip set address name="以太网" source=static addr=192.168.31.3 mask=255.255.255.0 gateway=192.168.31.11 1,,hide
    Run, *RunAs %ComSpec% /c netsh interface ip set dns name="以太网" source=static addr=192.168.31.11,,hide
        return
}
```

> **Tip：** IP、Mask、DNS、Gateway和设备名修改成自己的，例如我修改的是以太网，win+R 输入**ncpa.cpl**快速打开系统网络连接界面。
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/0e436c29c5804ec7b665ccff9f6ac3db.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iqx6Zuq6ZqP6aOO5LiN5Y6M55yL,size_20,color_FFFFFF,t_70,g_se,x_16)

双击运行：
![wallpaper:哔哩哔哩](https://img-blog.csdnimg.cn/c03ada0a0c0347ccaa02b8516efd1fdd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iqx6Zuq6ZqP6aOO5LiN5Y6M55yL,size_16,color_FFFFFF,t_70,g_se,x_16)
状态栏显示图标证明运行成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/8547e91755134cbe877293761eb9a059.png)
然后就可以实现一键切换IP啦

![网络IP](https://img-blog.csdnimg.cn/d8ce2ed6a6894a61a06bfb70537581f5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iqx6Zuq6ZqP6aOO5LiN5Y6M55yL,size_18,color_FFFFFF,t_70,g_se,x_16)
End.
> 参考：
> [Windows下用脚本快速修改IP地址](https://blog.csdn.net/violet_echo_0908/article/details/50375946)
> [如何下载和安装autohotkey](https://www.jianshu.com/p/4ec83b75c696)
> [修改网络需要管理员权限解决方法](https://www.autohotkey.com/boards/viewtopic.php?t=36639)
