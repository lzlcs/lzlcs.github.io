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

## Ubuntu 配置

```bash
rm -rf .config/
git clone git@github.com:lzlcs/.config.git .config
sudo apt update && sudo apt upgrade -y
```

### 配置代理

```bash
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890"
export http_proxy="http://${hostip}:7890"
export all_proxy="socks5://${hostip}:7890"
```


### Git 配置

```bash
git config --global user.name lzl
git config --global user.email 3012386836@qq.com
ssh-keygen -t rsa -C "3012386836@qq.com"
```
复制 `~/.ssh/id_rsa.pub` 中内容, 到 github 上添加 ssh 密钥


### 安装 Hexo

* 安装 nvm: `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`, 重启终端
* 安装 node, npm: `nvm install node`
* 安装 hexo: `npm install -g hexo-cli`
    * 
    ```bash
    git clone -b source git@github.com:lzlcs/lzlcs.github.io.git
    mv lzlcs.github.io Blog
    cd Blog
    npm install
    ```

### 安装 nvim lvim

下载 nvim 安装包，解压
```bash
echo 'export PATH="~/Apps/nvim/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

安装 pip
```bash
sudo apt install python-is-python3 -y
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
echo 'export PATH="~/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
rm get-pip.py
```

安装 cargo
```bash
sudo apt install cargo -y
```

安装 lazygit

```bash
cd ~/Apps/
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*')
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
tar xf lazygit.tar.gz lazygit
sudo install lazygit /usr/local/bin
rm lazygit.tar.gz
```

安装 lvim

```bash
LV_BRANCH='release-1.3/neovim-0.9' bash <(curl -s https://raw.githubusercontent.com/LunarVim/LunarVim/release-1.3/neovim-0.9/utils/installer/install.sh)
echo 'export PATH="~/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Clone 各种项目
