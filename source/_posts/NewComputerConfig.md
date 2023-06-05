---
title: NewComputerConfig
date: 2023-06-04 21:34:51
tags:
---

# 验机

检测屏幕, 硬盘, 配置单, 注意不要联网
联网激活Office, 安全下车

# 卸载及安装

* 卸载内置联想套系, 除了 `Legion Zone`, 这是调节电脑性能的工具
* 安装 QQ, Wechat 分在 `D://Software/ChatTools` 中
    * 更改 QQ, Wechat 默认文件下载位置为 `D://Files/QQFiles` 和 `D://Files/WechatFiles` 

* 安装网易云, 更改默认缓存目录 `D://Files/CloudMusicCache` 

* 安装 Steam, `D://Games/Steam`
* 安装 Bandzip, `D://Software/CompressionTools`
* 安装 Clash, `D://Software/Clash`, 然后改代理
    * 开启 Allow Lan, 开启 IPV6, 开启 Start with Windows, 开启 System Proxy
    * 打开 UWP Loopback 全选后保存

* 根据当前硬件安装各种驱动在 `D://Drivers` 中

# 安装 WSL

管理员模式打开 cmd

`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`
`dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`
`Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart`
重启
在 Microsoft Store 中安装 Ubuntu, 打开后设置用户名和密码

`wsl -l -v` 确认安装 Ubuntu 版本及 WSL 版本

## Windows Terminal 配置

启动: 
1. 默认配置文件 Ubuntu 22.04.2 LTS
2. 默认终端应用程序: Windows 终端

进入 Ubuntu 子页面中的外观
[下载](https://link.zhihu.com/?target=https%3A//www.nerdfonts.com/font-downloads) NerdFonts, 并在 Ubuntu 中选择该字体
[检测](https://link.zhihu.com/?target=https%3A//www.nerdfonts.com/cheat-sheet) 字体是否安装成功


