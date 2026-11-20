---
title: "机载NUC配置"
description: "Notes on configuring an onboard NUC for development and deployment."
lang: "zh-CN"
permalink: "/blog/onboard-nuc-setup/"
---

# 关闭桌面服务

- 默认无桌面启动
```
sudo systemctl set-default multi-user.target
```
- 恢复默认图形桌面启动
```
sudo systemctl set-default graphical.target
```
- 临时启动图形桌面
```
sudo systemctl start gdm3
```
- 关闭图形桌面
```
sudo systemctl stop gdm3
```
- 查看桌面服务状态
```
systemctl status gdm3
```

# 电源关机键设置

```
sudo nano /etc/systemd/logind.conf.d/power-button.conf
```

```
[Login]
HandlePowerKey=poweroff
PowerKeyIgnoreInhibited=no
```

# SSH 共享开发机网络代理，不需要在NUC装Clash

- 宿主机执行，例如：

```
ssh -N -R 17897:127.0.0.1:7897 wbr-nuc
```

- 在 NUC 上设置代理：

```
export http_proxy=http://127.0.0.1:17897
export https_proxy=http://127.0.0.1:17897
export all_proxy=socks5h://127.0.0.1:17897
```

# USB网卡有线连接配置

替换实际网卡名称：

```
sudo nmcli connection add \
  type ethernet \
  ifname enx9c69d3e11c30 \
  con-name mac-direct \
  ipv4.method manual \
  ipv4.addresses 192.168.50.2/24 \
  ipv4.never-default yes \
  ipv6.method disabled \
  connection.autoconnect yes
```

# FastBoot和通电后自动开机

### Fast Boot 导致无法进入 BIOS

开启 Fast Boot 后，开机时检测 `F2` 的时间窗口可能很短，甚至无法识别键盘。

解决方法：

1. 完全关闭 NUC。
    
2. 按住电源键约 3 秒，在第 4 秒前松开。
    
3. 进入 Power Button Menu。
    
4. 按 `F2` 进入 BIOS，或按 `F3` 禁用 Fast Boot（如果菜单支持）。
    

进入 BIOS 后，可以尝试在以下位置关闭快速启动：

`Advanced → Boot → Boot Configuration → Fast Boot`

### 通电自动开机

BIOS 设置路径通常为：

`Advanced → Power → Secondary Power Settings`

找到 After Power Failure：

|选项|功能|
|---|---|
|`Stay Off`|恢复供电后保持关机|
|`Last State`|恢复断电前的状态|
|`Power On`|恢复供电后自动开机|

推荐机器人 NUC 设置为 `Power On`，方便上电后自动启动系统。