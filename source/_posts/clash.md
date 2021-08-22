---
title: Ubuntu 20.04 clash for linux 配置
date: 2021-08-22 22:53:22
categories:
- Linux
tags:
- Linux
- 配置
---

参考：

[ClashForLinux自动更新订阅配置](https://swing1993.cn/clashzi-dong-geng-xin-ding-yue-pei-zhi/amp/)

[Clash For Linux自动更新订阅配置](https://ytlee.cn/2021/04/clash-for-linux-automatically-renews-the-subscription-link/)

# 配置

通过systemctl来管理Clash的进程，对应`clash.service`文件，通过两个脚本`start-clash.sh`和`stop-clash.sh`来管理Clash的启停，具体配置如下。

## clash.service

置于`/lib/systemd/system/clash.service`，注意修改用户名和订阅链接。

```shell
[Unit]
Description=clash daemon
After=network.target

[Service]
Type=simple
User=tangger
ExecStart=/home/<用户名>/.config/clash/start-clash.sh
ExecStop=/home/<用户名>/.config/clash/stop-clash.sh
Environment="HOME=/home/<用户名>/"
Environment="CLASH_URL=<填入您的订阅链接>"

[Install]
WantedBy=multi-user.target
```

## start-clash.sh

置于`${HOME}/.config/clash/start-clash.sh`，注意修改用户名。

```shell
#!/bin/bash
# save this file to ${HOME}/.config/clash/start-clash.sh

# save pid file
echo $$ > ${HOME}/.config/clash/clash.pid

diff ${HOME}/.config/clash/config.yaml <(curl -s ${CLASH_URL})
if [ "$?" == 0 ]
then
    /usr/local/bin/clash -d /home/<用户名>/.config/clash/
else
    curl -L -o ${HOME}/.config/clash/config.yaml ${CLASH_URL}
    /usr/local/bin/clash -d /home/<用户名>/.config/clash/
fi
```

## stop-clash.sh

置于`${HOME}/.config/clash/stop-clash.sh`。

```shell
#!/bin/bash
# save this file to ${HOME}/.config/clash/stop-clash.sh

# read pid file
PID=`cat ${HOME}/.config/clash/clash.pid`
kill -9 ${PID}
rm ${HOME}/.config/clash/clash.pid
```

## 赋予执行权限

运行：

```shell
chmod +x ${HOME}/.config/clash/start-clash.sh
chmod +x ${HOME}/.config/clash/stop-clash.sh
```

## 最后

运行：

```shell
systemctl enable clash
systemctl start clash
```

`ps -ef | grep clash`验证下clash是否已在运行，Setting - Network中配置Network Proxy，然后访问网页测试。没问题的话，clash将开机自动启动，并自动更新配置。

## Tips

- clash二进制文件放在`/usr/local/bin/`
- 注意文件路径
- .sh记得加执行权限
