---
title: wls2 Ubuntu配置
date: 2021-04-15 22:18:57
categories:
- Linux
tags:
- Linux
- 配置
---


主要参考https://zhuanlan.zhihu.com/p/345559097

#  在Ubuntu安装Zsh

终端里面输入：

```shell
sudo apt-get update
sudo apt-get install zsh
```

# 安装插件和主题

插件：

- `zsh-autosuggestions`：这个是自动建议插件，能够自动提示你需要的命令。
- `zsh-syntax-highlighting`：这个是代码高亮插件，能够使你的命令行各个命令清晰明了。

终端里面输入命令：

```shell
sudo apt-get install zsh-autosuggestions zsh-syntax-highlighting
```

主题：

- `zsh-theme-powerlevel10k` 这个主题提供漂亮的提示符，可以显示当前路径、时间、命令执行成功与否，还能够支持 git 分支显示等等。安装过程参见：https://github.com/romkatv/powerlevel10k

# 更改默认 shell

终端输入命令：

```shell
chsh -s /usr/bin/zsh
```

# 在 Ubuntu 启用插件

打开 `~/.zshrc` 文件，将以下行代码添加到其中：

```text
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

# 启用历史输入搜索

在`~/.zshrc` 文件添加：

```shell
bindkey '^R' history-incremental-search-backward
```



# wsl2设置Windows代理

https://zhuanlan.zhihu.com/p/153124468

打开`/etc/zsh/zshenv`文件，将以下两行添加到末尾

```shell
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="http://$host_ip:7890"
```

# 自动将wsl2 Ubuntu ip写入Windows hosts文件

https://github.com/shayne/go-wsl2-host

# wsl2自动启动ssh server

https://gist.github.com/dentechy/de2be62b55cfd234681921d5a8b6be11

https://www.illuminiastudios.com/dev-diaries/ssh-on-windows-subsystem-for-linux/
