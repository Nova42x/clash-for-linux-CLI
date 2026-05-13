# clash-for-linux-CLI

![GitHub License](https://img.shields.io/github/license/Nova42x/clash-for-linux-CLI)
![GitHub top language](https://img.shields.io/github/languages/top/Nova42x/clash-for-linux-CLI)
![GitHub Repo stars](https://img.shields.io/github/stars/Nova42x/clash-for-linux-CLI)

![preview](resources/preview.png)

在 Linux 终端中一键安装、管理和使用 Clash / mihomo 代理内核。支持 systemd、SysVinit、OpenRC、runit 等主流 init 系统，兼容 root 与普通用户环境。

## 快速开始

### 安装

```bash
git clone --depth 1 https://gh-proxy.org/https://github.com/Nova42x/clash-for-linux-CLI.git \
  && cd clash-for-linux-CLI \
  && bash install.sh
```

> 安装完成后，`clashctl` 命令和所有快捷别名（`clashon`、`clashoff` 等）会自动加载到你的 shell 中。

### 第一次使用

```bash
# 启动代理
$ clashon
😼 已开启代理环境

# 关闭代理
$ clashoff
😼 已关闭代理环境
```

搞定！这两条命令就是最常用的日常操作。

---

## clashctl 使用指南

`clashctl` 是这个项目的核心命令，有两种使用方式：**交互式菜单** 和 **直接命令**。

### 方式一：交互式菜单（推荐新手）

直接输入 `clashctl` 不带任何参数，会进入一个可视化控制面板：

```bash
$ clashctl
```

进入后你会看到这样的界面：

```
╔═══════════════════════════════════════════════╗
║              clashctl 控制面板                ║
╚═══════════════════════════════════════════════╝

  内核: mihomo    状态: [ON]
  订阅: [1] https://example.com/sub
  Tun:  [OFF]    系统代理: [ON]
  端口: mixed=7890  http=无  socks=无

  [1] 开启代理         [2] 关闭代理
  [3] 订阅管理         [4] Tun 模式
  [5] 系统代理         [6] Web 密钥
  [7] Web 控制台       [8] 查看日志
  [9] 升级内核         [10] Mixin 配置
  [0] 退出
```

输入数字编号即可操作，比如输入 `3` 进入订阅管理子菜单，输入 `0` 返回上一级。**无需记忆任何命令参数**。

### 方式二：直接命令（推荐老手）

如果你习惯在终端里敲命令，`clashctl` 也支持直接传参：

```bash
clashctl <命令> [选项]
```

日常高频操作还有更短的快捷别名：

| 操作 | clashctl 命令 | 快捷别名 |
|------|--------------|---------|
| 开启代理 | `clashctl on` | `clashon` |
| 关闭代理 | `clashctl off` | `clashoff` |
| 查看状态 | `clashctl status` | `clashstatus` |
| 打开 Web 面板 | `clashctl ui` | `clashui` |
| 查看日志 | `clashctl log` | `clashlog` |
| 管理订阅 | `clashctl sub` | `clashsub` |
| 升级内核 | `clashctl upgrade` | `clashupgrade` |
| Tun 模式 | `clashctl tun` | `clashtun` |
| Mixin 配置 | `clashctl mixin` | `clashmixin` |
| Web 密钥 | `clashctl secret` | `clashsecret` |
| 系统代理 | `clashctl proxy` | `clashproxy` |

> 两种方式完全等价，快捷别名就是对应的函数名，Tab 补全更方便。

---

## 各功能详解

### 开启 / 关闭代理

```bash
$ clashon
😼 已开启代理环境

$ clashoff
😼 已关闭代理环境
```

执行 `clashon` 时，工具会自动：
1. 检测端口占用，冲突时随机分配可用端口
2. 启动内核进程
3. 同步设置系统代理环境变量

执行 `clashoff` 时，会自动关闭内核并清除代理环境变量。

### 订阅管理

```bash
# 添加订阅
$ clashsub add "https://example.com/sub"
🎉 订阅已添加：[1] https://example.com/sub

# 查看所有订阅
$ clashsub ls

# 切换使用某个订阅
$ clashsub use 1
🔥 订阅已生效

# 更新订阅（拉取最新节点）
$ clashsub update

# 设置自动更新（每 2 天通过 crontab 自动执行）
$ clashsub update --auto

# 更新时使用订阅转换
$ clashsub update --convert

# 删除订阅
$ clashsub del 1

# 查看订阅操作日志
$ clashsub log
```

- 支持本地订阅文件：`file:///root/clashctl/resources/config.yaml`
- 链接含特殊字符时请用引号包裹

### Tun 模式

```bash
# 查看 Tun 状态
$ clashtun
😾 Tun 状态：关闭

# 开启 Tun 模式
$ clashtun on
😼 Tun 模式已开启

# 关闭 Tun 模式
$ clashtun off
😼 Tun 模式已关闭
```

Tun 模式的作用是将本机及 Docker 容器的所有流量路由到 Clash 代理，实现全局代理和 DNS 劫持。

### 系统代理

```bash
# 查看系统代理状态
$ clashproxy
系统代理：开启
http_proxy=http://127.0.0.1:7890
...

# 单独开启系统代理（不影响内核）
$ clashproxy on

# 单独关闭系统代理
$ clashproxy off
```

> `clashon` 默认会同步开启系统代理，`clashproxy` 可以独立控制。

### Web 控制台

```bash
$ clashui
╔═══════════════════════════════════════════════╗
║                Web 控制台                     ║
║═══════════════════════════════════════════════║
║     注意放行端口：9090                         ║
║     内网：http://192.168.0.1:9090/ui          ║
║     公网：http://x.x.x.x:9090/ui             ║
║     公共：http://board.zash.run.place         ║
╚═══════════════════════════════════════════════╝
```

- 默认使用 [zashboard](https://github.com/Zephyruso/zashboard) 作为控制台前端
- 若需暴露到公网，建议定期更换密钥（`clashsecret`）

### Web 密钥

```bash
# 查看当前密钥
$ clashsecret
当前密钥：mysecret

# 修改密钥
$ clashsecret newsecret
密钥更新成功，已重启生效
```

### Mixin 配置

```bash
# 查看 Mixin 配置
$ clashmixin

# 编辑 Mixin 配置（会自动重启内核生效）
$ clashmixin -e

# 查看原始订阅配置（安装时下载的）
$ clashmixin -c

# 查看合并后的运行时配置
$ clashmixin -r
```

Mixin 是自定义配置的入口。你在 `mixin.yaml` 中的修改会与原始订阅深度合并，具有最高优先级。支持对规则、节点和策略组进行 prefix（前置）、suffix（后置）、override（覆盖）和 inject（注入）操作。

### 升级内核

```bash
$ clashupgrade
⏳ 请求内核升级...
{"status":"ok"}
内核升级成功

# 查看详细升级日志
$ clashupgrade -v

# 升级到稳定版
$ clashupgrade -r

# 升级到测试版
$ clashupgrade -a
```

建议通过 `clashmixin` 为 GitHub 配置代理规则，避免网络问题导致升级失败。

### 查看日志

```bash
# 查看内核运行日志
$ clashlog

# 查看最近 30 行日志
$ clashlog | tail -30
```

---

## 卸载

```bash
bash uninstall.sh
```

## 常见问题

👉 [Wiki · FAQ](https://github.com/Nova42x/clash-for-linux-CLI/wiki/FAQ)

## 引用

- [clash](https://clash.wiki/)
- [mihomo](https://github.com/MetaCubeX/mihomo)
- [subconverter](https://github.com/tindy2013/subconverter)
- [yq](https://github.com/mikefarah/yq)
- [zashboard](https://github.com/Zephyruso/zashboard)

## Star History

<a href="https://www.star-history.com/#Nova42x/clash-for-linux-CLI&Date">

 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=Nova42x/clash-for-linux-CLI&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=Nova42x/clash-for-linux-CLI&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=Nova42x/clash-for-linux-CLI&type=Date" />
 </picture>
</a>

## Thanks

[@鑫哥](https://github.com/TrackRay)

## 特别声明

1. 编写本项目主要目的为学习和研究 `Shell` 编程，不得将本项目中任何内容用于违反国家/地区/组织等的法律法规或相关规定的其他用途。
2. 本项目保留随时对免责声明进行补充或更改的权利，直接或间接使用本项目内容的个人或组织，视为接受本项目的特别声明。
