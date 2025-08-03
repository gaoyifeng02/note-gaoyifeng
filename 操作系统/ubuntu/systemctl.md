systemctl 是 Linux 系统中管理 systemd 服务（守护进程）的核心命令，用于 启动、停止、重启、查看状态、启用/禁用开机自启 等操作。以下是 systemctl 的 详细用法 和 常见场景示例。

1. 基本语法

systemctl [选项] <命令> <服务名>

• <服务名> 通常是 服务名.service（如 ssh.service），但可以省略 .service 直接写 ssh。

2. 常用命令

（1）服务状态管理

命令 作用 示例

systemctl start <服务> 启动服务 sudo systemctl start ssh

systemctl stop <服务> 停止服务 sudo systemctl stop ssh

systemctl restart <服务> 重启服务 sudo systemctl restart ssh

systemctl reload <服务> 重新加载配置（不重启） sudo systemctl reload nginx

systemctl status <服务> 查看服务状态（是否运行、日志等） systemctl status ssh
（2）开机自启管理
命令 作用 示例

systemctl enable <服务> 启用开机自启 sudo systemctl enable ssh

systemctl disable <服务> 禁用开机自启 sudo systemctl disable ssh

systemctl is-enabled <服务> 检查是否开机自启 systemctl is-enabled ssh
（3）服务日志查看
命令 作用 示例

journalctl -u <服务> 查看服务的日志 journalctl -u ssh

journalctl -u <服务> -f 实时跟踪日志（类似 tail -f） journalctl -u ssh -f

3. 常见使用场景

场景 1：管理 SSH 服务

# 启动 SSH
sudo systemctl start ssh

# 停止 SSH
sudo systemctl stop ssh

# 重启 SSH
sudo systemctl restart ssh

# 查看 SSH 状态
systemctl status ssh

# 设置 SSH 开机自启
sudo systemctl enable ssh

# 禁止 SSH 开机自启
sudo systemctl disable ssh


场景 2：管理 Nginx/Apache

# 启动 Nginx
sudo systemctl start nginx

# 重新加载 Nginx 配置（不中断服务）
sudo systemctl reload nginx

# 查看 Nginx 日志
journalctl -u nginx -f


场景 3：管理 Docker

# 启动 Docker
sudo systemctl start docker

# 设置 Docker 开机自启
sudo systemctl enable docker


场景 4：查看所有服务

# 列出所有已加载的服务
systemctl list-units --type=service

# 列出所有已启用的服务（开机自启）
systemctl list-unit-files --state=enabled


4. 高级用法

（1）强制杀死服务

如果 systemctl stop 无效，可以用 kill：
# 查看服务的 PID
systemctl show --property=MainPID <服务>

# 强制杀死进程
sudo kill -9 <PID>


（2）修改服务配置

# 编辑服务配置（如 ssh.service）
sudo systemctl edit --full ssh

# 重新加载 systemd 配置（修改后生效）
sudo systemctl daemon-reload


（3）依赖关系检查

# 查看服务的依赖关系
systemctl list-dependencies ssh


5. 常见问题

Q1: systemctl status 输出的关键信息

• Active: active (running) → 服务正在运行。

• Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled) → 服务已启用开机自启。

• Main PID: 1234 (sshd) → 服务的主进程 ID。

Q2: 服务启动失败怎么办？

# 查看详细错误日志
journalctl -u <服务> -xe


Q3: systemctl 和 service 命令的区别？

• systemctl 是 systemd 体系的命令（现代 Linux 发行版默认）。

• service 是旧版 SysVinit 的命令（兼容模式，底层仍调用 systemctl）。

总结

需求 命令

启动服务 sudo systemctl start <服务>

停止服务 sudo systemctl stop <服务>

重启服务 sudo systemctl restart <服务>

查看状态 systemctl status <服务>

启用开机自启 sudo systemctl enable <服务>

禁用开机自启 sudo systemctl disable <服务>

查看日志 journalctl -u <服务>

掌握 systemctl 后，你可以轻松管理 Linux 上的各种服务（如 SSH、Nginx、Docker 等）。🚀
