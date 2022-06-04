---
layout: ESP32蓝牙网关
title: 网关开发注意事项
date: 2022-03-03 15:18:27
tags: 
---

# 网关开发注意事项

**经过和深圳老伍合作之后，将踩过的坑记录下来，为后续我们自主开发网关少走弯路。**

## 1 网关与平台通讯

### 1.1 下发命令

- 远程开锁
**网关收到指令后，尝试和目标蓝牙锁建立连接，发送开锁指令；**
- 绑定蓝牙锁
**下发到网关，网关本地要有一个列表用于存储绑定的蓝牙锁;**
- 下发密码
**下发临时密码到蓝牙锁，具有时限，开始和结束的时间可以自定义;**
- 下发卡号
**下发门卡卡号到蓝牙锁，存储到蓝牙锁本地;**

### 1.2 上报信息

- 开锁成功上报
- 门磁状态变化上报
- 密码开锁上报
- 门卡开锁上报
- 指纹开锁上报
- 定时发送锁的状态和网关的状态

> 建议添加:1.远程重启功能，或在凌晨重启，保持网关正常的运行状态;2.锁的电量信息上报;

## 2 网关与蓝牙锁通讯

### 2.1 蓝牙扫描

网关分为WiFi模块和蓝牙模块，蓝牙服务进程，不停扫描附近的蓝牙设备，外部中断停止进行连接；

### 2.2 蓝牙连接、重连

接收到开锁指令，停止扫描，与目标蓝牙设备建立连接，连接失败重试，超时上报开锁失败信息；

### 2.3 蓝牙锁上报状态信息

若门磁状态发生变化、门卡触发、密码开锁、主动请求连接网关，上报信息

> 注意：1.门卡触发或者密码开锁都会触发门磁，需要在锁端进行判断是否发送门磁信息;2.密码判断是否正确；

## 3 硬件问题

### 3.1 蓝牙锁触摸屏

电路板和锁面板模具的适配问题；

### 3.2 网关的天线

网关的Wifi连接信号、蓝牙信号都与射频相关，目前出现的无法开锁，网关和蓝牙锁无法连接问题都指向网关的天线，Wifi和蓝牙都依靠一根天线，不能再做成板载天线；