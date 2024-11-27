**Bonding简介**
bonding(绑定)是一种linux系统下的网卡绑定技术，可以把服务器上n个物理网卡在系统内部抽象(绑定)成一个逻辑上的网卡，能够提升网络吞吐量、实现网络冗余、负载均衡等功能，有很多优势。

bonding技术是linux系统内核层面实现的，它是一个内核模块(驱动)。使用它需要系统有这个模块, 我们可以modinfo命令查看下这个模块的信息, 一般来说都支持。

```
modinfo bonding |more
```
![338db61f8dfd4cd3d3d7d5b02864a4f6](https://github.com/user-attachments/assets/4e67e93d-be25-415a-9767-086ddd4442dc)


Ubuntu从17.04开始，已经放弃在/etc/network/interfaces 里固定IP的配置，而是改成netplan方式，配置文件：/etc/netplan/00-installer-config.yaml


**1.修改配置文件**
备份文件
```
cp etc/netplan/00-installer-config.yaml etc/netplan/00-installer-config.yaml.old
```
修改文件
```
cat << EOF > /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  ethernets:
    ens29f0:
      dhcp4: false
      optional: true
    ens29f1:
      dhcp4: false
      optional: true
  bonds:
    bond0:
      dhcp4: false
      addresses: [192.168.1.11/24]
      optional: true
      routes:
        - to: default
          via: 192.168.1.254
      nameservers:
        addresses: [8.8.8.8,114.114.114.114]
      interfaces:
        - ens29f0
        - ens29f1
      parameters:
        mode: balance-alb #绑定模式
        mii-monitor-interval: 100 #心跳时间
        lacp-rate: fast #
        transmit-hash-policy: layer2
EOF
```

**2.验证网络**
重启netplan服务,或者重启服务器
```
netplan apply
ifconfig
```

