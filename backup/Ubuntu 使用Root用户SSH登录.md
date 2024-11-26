**1、安装openssh**
使用如下命令安装openssh

sudo apt install openssh-server

**2、修改配置文件sshd_config**
安装完成后修改配置文件/etc/ssh/sshd_config，命令如下
sudo nano /etc/ssh/sshd_config

将

#PermitRootLogin prohibit-password

改成

PermitRootLogin yes

**3、重启服务**
使用如下命令程序ssh服务

sudo systemctl restart ssh