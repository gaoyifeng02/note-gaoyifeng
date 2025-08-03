# ip地址

ip addr show

# 配置管理员账号

配置账号
sudo passwd root

登录账号
su root

# 配置SSH

配置ssh后方可使用finalshell连接

sudo apt update
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh

systemctl可以服务自启动


# 永久关闭防火墙

sudo ufw disable  # 立即关闭防火墙
sudo systemctl disable ufw  # 禁止防火墙开机自启




