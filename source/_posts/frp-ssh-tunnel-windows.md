---
title: 用 frp + SSH 远程访问内网 Windows 主机
date: 2026-09-28 10:45:00
tags:
  - frp
  - ssh
  - windows
  - AI辅助写作
categories:
  - tutorials
---

内网 Windows 主机没有公网 IP，但可以主动访问一台有公网 IP 的 Linux 服务器？可以用 **frp 建立反向隧道，再通过 SSH 登录 Windows**。

这篇文章从网络路径、配置到排错，走一遍最小可用方案。文中的主机名、用户名和端口都是示例，请替换成自己的值；不要把真实 IP、密码或 frp token 提交到公开仓库。

## 网络路径

需要三台机器：

- **客户端**：发起 SSH 连接的电脑。
- **公网跳板机**：运行 `frps`，有公网 IP。
- **内网 Windows 主机**：运行 OpenSSH Server（`sshd`）和 `frpc`。

连接过程分两段：Windows 主机主动连到跳板机注册 frp 隧道；客户端再连到跳板机的映射端口。frp 把这条连接转发到 Windows 主机本机的 SSH 服务。

```text
客户端 --SSH--> 跳板机:6022 --frp 隧道--> Windows:22
                          ^                 |
                          +---- frpc 主动连接到跳板机:7000 ----+
```

这里用 `7000` 作为 frp 控制端口、`6022` 作为 SSH 映射端口、`22` 作为 Windows 本地 SSH 端口。可以按自己的网络和防火墙规则更换。

## 准备工作

1. 一台可从公网访问的 Linux 服务器，并能开放两个 TCP 端口。
2. 一台可以主动访问该服务器的 Windows 10/11 主机。
3. 三台机器上可用的 SSH 客户端；frps 和 frpc 使用**相同版本**的 frp。
4. 一个随机生成的长 token。不要复用账号密码，也不要把 token 写入 Git 仓库。

frp 配置语法会随版本演进。下面的 TOML 示例适用于支持该配置格式的版本；下载时选择匹配的版本，并以对应版本的[官方文档](https://gofrp.org/zh-cn/docs/)为准。

## 1. 配置公网 Linux 跳板机

从 [frp Releases](https://github.com/fatedier/frp/releases) 下载与服务器架构匹配的压缩包，将 `frps` 安装到系统路径。下面假设二进制文件位于 `/usr/local/bin/frps`。

创建配置目录和 `/etc/frp/frps.toml`：

```bash
sudo install -d -m 0755 /etc/frp
```

```toml
bindPort = 7000
auth.token = "替换为长随机字符串"
```

创建 systemd unit `/etc/systemd/system/frps.service`：

```ini
[Unit]
Description=frp server
After=network.target

[Service]
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.toml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

启动并检查服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now frps
sudo systemctl status frps
```

防火墙和云平台安全组都要允许需要的入站端口：`7000/tcp` 供 frpc 建立隧道，`6022/tcp` 供客户端连接。若网络条件允许，将 `6022/tcp` 限制为可信客户端的来源 IP。

## 2. 启用 Windows OpenSSH Server

在管理员 PowerShell 中查看服务：

```powershell
Get-Service sshd
```

如果尚未安装 OpenSSH Server，按 [Microsoft 的 Windows OpenSSH 安装说明](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse)安装。随后启动并设置开机自启：

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

先在 Windows 本机确认 SSH 服务正常，再继续配置 frp。不要为了调试而把 Windows 的 `22` 端口直接开放到公网。

## 3. 配置 Windows frpc

从 [frp Releases](https://github.com/fatedier/frp/releases) 下载与 `frps` **完全相同版本**的 Windows 压缩包，例如解压到 `C:\frp\`。创建 `C:\frp\frpc.toml`：

```toml
serverAddr = "跳板机公网 IP 或域名"
serverPort = 7000
auth.token = "与 frps 相同的长随机字符串"

[[proxies]]
name = "windows-ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6022
```

在 PowerShell 里手动启动一次，确认配置和连通性：

```powershell
Set-Location C:\frp
.\frpc.exe -c .\frpc.toml
```

连接成功后应能在客户端日志中看到代理启动成功的信息。确认后按 `Ctrl+C` 停止，再将 frpc 注册为 Windows 服务。可用 [NSSM](https://nssm.cc/usage) 管理服务：程序路径指向 `C:\frp\frpc.exe`，参数为 `-c C:\frp\frpc.toml`。安装并启动后检查：

```powershell
nssm status frpc
```

服务运行账户需要有读取配置文件和执行 frpc 的权限。token 仅供 frpc/frps 认证，不要放到博客、截图或公开仓库里。

## 4. 从客户端连接

直接连接映射端口：

```bash
ssh -p 6022 <Windows用户名>@<跳板机公网IP或域名>
```

SSH 使用的是 **Windows 上 OpenSSH Server 接受的账户和认证方式**，不是跳板机上的 Linux 用户。建议配置密钥认证；Windows OpenSSH 的公钥文件位置和权限有其平台规则，按 [Microsoft 文档](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_keymanagement)设置。

也可以在客户端的 `~/.ssh/config` 中添加别名：

```sshconfig
Host my-windows
    HostName <跳板机公网IP或域名>
    Port 6022
    User <Windows用户名>
    IdentityFile ~/.ssh/id_ed25519
```

之后运行 `ssh my-windows` 即可。

## 常见排查顺序

从内到外逐层确认，比一开始只看 SSH 报错更快：

| 检查项 | 命令 / 现象 |
|---|---|
| Windows SSH 服务 | `Get-Service sshd`，状态应为 `Running` |
| frpc 配置和日志 | 手动运行 `frpc.exe -c C:\frp\frpc.toml`，查看 token、地址、版本和连接错误 |
| frps 服务 | `sudo systemctl status frps`；日志用 `sudo journalctl -u frps -f` |
| 端口放行 | 云安全组和 Linux 防火墙都允许 `7000/tcp`、`6022/tcp` |
| 客户端到映射端口 | `nc -zv <跳板机地址> 6022`，或使用系统提供的 TCP 连通性测试工具 |
| SSH 认证 | 确认用户名是 Windows 账户名，检查公钥位置、权限及 sshd 日志 |

如果 frpc 已连接但 SSH 不通，优先检查 `remotePort`、跳板机入站规则和 Windows 本机 `sshd`。如果 frpc 无法连接，则先检查 `serverAddr`、`serverPort`、两端 token 与 frp 版本是否一致。

## 安全检查清单

- 使用随机生成的长 token；配置文件保持在私有位置，避免提交到版本控制。
- 优先使用 SSH 密钥认证。确认密钥登录可用后，再评估是否禁用密码登录；先保留一个已验证的管理会话，避免把自己锁在机器外。
- 对跳板机入站端口设置最小开放范围；`6022` 尽量限制可信来源 IP。
- Windows SSH 使用权限受限的账户；不要直接暴露日常高权限账户。
- 不使用 frps Dashboard 就不要启用；如果确有需要，必须限制访问并设置强认证。
- 给每台目标主机分配清楚的代理名和映射端口，并定期检查服务、日志和 frp 版本。

## 参考资料

- [frp 项目与 Releases](https://github.com/fatedier/frp)
- [frp 官方文档](https://gofrp.org/zh-cn/docs/)
- [Microsoft：安装 Windows OpenSSH](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse)
- [Microsoft：OpenSSH 密钥管理](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_keymanagement)
- [NSSM 使用说明](https://nssm.cc/usage)
