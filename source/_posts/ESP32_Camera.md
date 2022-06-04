---
layout: ESP32
title: ESP32_CAM使用RTSP推流图像到本地服务器，附Arduino IDE+VsCode 环境搭建
date: 2022-06-04 16:32:17
categories: 
- ESP32
tags: 
- ESP32
---
# ESP32_CAM使用RTSP推流图像到本地服务器，附Arduino IDE+VsCode 环境搭建

## 效果展示

![先给图再BB](https://img-blog.csdnimg.cn/efbac25c014e409eb993e9f5b7522b2e.png)

## 一、前言

首先，感谢龙哥的板板，没有龙哥的板子支持就没有这篇文章。

本片文章参考国外进行实践[参考链接](https://randomnerdtutorials.com/esp32-cam-video-streaming-face-recognition-arduino-ide/), 使用ESP32-S CAM 开发板进行开发，通过驱动摄像头，将采集的图像通过RTSP推流到本地服务器，访问WEB后台即可查看实时画面，包含人脸检测、人脸识别功能，使用Arduino IDE进行实现, 因为个人习惯，顺便在VsCode上配置了Arduino平台的开发环境。

## 二、环境搭建

### 1.Arduino IDE

[Arduino 官方下载地址](https://www.arduino.cc/en/software)

![图1](https://img-blog.csdnimg.cn/ac82615ca7664eb0af79fc5d2247542c.png)

点击右侧下载压缩包，解压可以直接用，如果不想配置VsCode的话也可以在微软商店里面下载，要配置而且你电脑的用户名还是中文的话就别了，设置完路径一样会报错;

### 2.Arduino配置ESP32库、字体

**双击打开下载解压好的安装包里面的arduino.exe文件，进入首页，这就是Arduino IDE啦**

![图2](https://img-blog.csdnimg.cn/281c64fe90e44119ae719ebd6e8bfc58.png)

**点击"文件"、选择"首选项"**

在下面的附加开发板管理器网址中**填入以下链接**:

```code
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json,http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

![图3](https://img-blog.csdnimg.cn/70f3da49dfe448cab6077dda0dc87432.png)

点击**好**

如果想要修改字体的话，点击修改下图那个文件中的属性editor.font即可，我始终觉得，一个如果常用的IDE，就像你的女朋友（或者未来的女朋友），有时候你面对他的时间比面对你真实的女朋友的时间多得多，肯定要好看噶，字体是首要；

![图4](https://img-blog.csdnimg.cn/e7fd2509ca354aeeabbdf0de36ae4939.png)

**回到首页，点击"工具", 点击开发板-->开发板管理器**

![图5](https://img-blog.csdnimg.cn/f1a45578daa349f18d2e9779895dc8a7.png)

**在搜索框输入ESP32, 点击"安装"**

![图6](https://img-blog.csdnimg.cn/c02a9e1320d74c898522a20cabb7c550.png)

等待安装完成，如果没有工具的话，就要耐性等待哇

安装完成后，点击关闭，工具-->开发板-->选择目标开发板

![图7](https://img-blog.csdnimg.cn/12b8f62d4b7b4a0283d6ca88141f048e.png)

**到这里，对应我们的板板的环境已经待见起来了哈**

## 三、项目配置及执行

### 1.项目配置

**1.打开文件，选择示例程序，如图（有图莫多BB）**

![图8](https://img-blog.csdnimg.cn/3d45480f1b8d470d955be52383186973.png)
**2.记得配置板板分区大小，不然会编译失败哦**
![配置应用大小](https://img-blog.csdnimg.cn/d283371baa1c4cd9819c4f9cd6ad26e9.png)

**3.修改WIFI SSID和密码为你当前可用的WiFi**
实在不行你拿手机或笔记本开个热点也行，注意：保证待会你的访问设备（就你要看视频的设备）要和板板在同一个局域网内;

![图9](https://img-blog.csdnimg.cn/8dba974b11fa4c7b92d1fe8c5509ca40.png)

### 2.编译

**4.好噶，点击验证（我也迷糊这IDE的编译不叫编译，叫验证，英文），点它就完事了，等待编译完成；**

![图10](https://img-blog.csdnimg.cn/e82c2bdb12db449b875fc07ed9219b11.png)
![编译](https://img-blog.csdnimg.cn/5e72f983352a45449fc6b58f954be042.png)

**5.好啦，现在将你的板板插上PC，点击工具选择你板板所在的端口**

![图11](https://img-blog.csdnimg.cn/8567ff49863946bfb217cfbc166e217d.png)

### 3.烧录

**6.按住IO0别松手，然后按一下Reset键，点击upload（烧录不叫烧录，叫上传，我真是服了这个老6），等待烧录完成（其实你每次烧录前默认它都会帮你再编译一遍）；**
![图12](https://img-blog.csdnimg.cn/a7c470b260204cdf80b971e15f77a470.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5236a95507bc4437a74f29b2a1c31ef6.png)

>> 注意，有的同学可能在玩ESP32板子的收碰到这么一个问题，明明俺的是重启，但是却提示等待烧录：如下
>
> ```ssh
> rst:0x1 (POWERON_RESET),boot:0x3 (DOWNLOAD_BOOT(UART0/UART1/SDIO_REI_REO_V2)) waiting for download
> ```
>
遇事不要慌，let's thinks, 然后google一下，[参考链接](https://github.com/espressif/arduino-esp32/issues/577)，发现出现这个问题是因为IO0一直是低电平，可能你会说没有啊，但是电压不稳定或者存在其他干扰都可能导致IO0没办法拉高，插拔或者将IO0接3.3V试下，真帮你解决了回来帮我点个赞；

### 4.运行

**7.给你的板板重新上电，Arduino自带有串口工具，点击工具-->串口监视器；**

![图13](https://img-blog.csdnimg.cn/d90943181eee46e6865a7ea79d4f147b.png)

看到有打印IP信息说明你的程序运行成功啦

![图14](https://img-blog.csdnimg.cn/5a34891d7144499ebbb419effe25eb77.png)

使用设备访问（同一个局域网内的手机电脑都可以哦），点击Get Stream，小功告成；

![图15](https://img-blog.csdnimg.cn/cf7b303fe47e428087f6cab147015a44.jpeg)
可能你会说，咋这么糊捏，嚯，整起，如下图，点击Resolution，所示你即可随心所欲
![图16](https://img-blog.csdnimg.cn/704100d8843c4abe9e95567633879711.jpeg)
![最高分辨率](https://img-blog.csdnimg.cn/9d46d38814284ea99860f486d6d93751.png)

最高分辨率支持1600x1200，但是分辨率高了，帧率就会变低，各位根据自己的需求作夺舍吧；

## 四、Arduino IDE+VsCode 环境搭建

~~每个人都有自己真心所爱的办公挚爱，相处的时间也最多，能看到这里的小伙伴们，我想我们都有一个相同的挚爱--VsCode，~~ 好啦好啦，现在开始在vscode想配置Arduino的开发环境；

### 1.安装插件

扩展商店，输入Arduino搜索，点击安装，完视；
![安装插件](https://img-blog.csdnimg.cn/fa5c30bb73dc4d11b6b367393e0ca772.png)

### 2.扩展配置

![扩展设置](https://img-blog.csdnimg.cn/176d2ac328824331b686b032c9b67e8a.png)
**将Arduino Path设置为你Arduino的解压或者或者是下载路径；**
![设置路径](https://img-blog.csdnimg.cn/22cdc93248844ee09a93896025baa164.png)

### 3.常用功能

![常用功能](https://img-blog.csdnimg.cn/b2c4e5de98394e62b58ce49c4d3e354b.png)
>> 如果你也是个PC用户名为中文，而且从微软商店下载Arduino IDE的大怨种的话，如果出现编译工具的路径问题的话，请去官网重新压缩包进行操作，而如果都设置好了还有问题，报错output 路径没有设置的话，修改项目根目录下的.vsoce文件夹下载arduino.json文件，添加以下配置
>
>```json
>    "output": "../Arduino/build",
>    ```
>
End.
