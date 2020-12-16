# 实验室服务器配置指北

------

## Contents

- [BIOS设置](#bios)
- [系统安装与配置](#system)

------

## BIOS
([↑up to contents](#contents))
在完成硬件的安装，插入刻录好的U盘，开机之后，由于我们安装了多张显卡，需要在BIOS设置中打开"4G Decodeing"选项。这样可正常开机，系统不会报错。
之后将选择U盘启动，进入系统安装。

## System
([↑up to contents](#contents))
### 1. 账户管理
开机后进入Ubuntu18.04系统安装步骤，配置带有root权限的账户密码，完成安装之后重启系统。
进入系统后根据实验需求，在root权限账户的终端输入
```
sudo adduser username
```
并在接下来的步骤设置密码，完成无root权限账户的创建，并分发给相应同学。
### 2.显驱安装
在root账号下，终端输入
```
ubuntu-drivers devices
sudo ubunutu-drivers autoinstall
```
自动安装Nvidia驱动，之后重启，在命令行输入
```
nvidia-smi
```
若显示显卡信息则安装成功。

### 3. 图形界面
为确保训练进程不被内核关闭，我们将服务器图形界面关闭，在root权限的终端输入
```
sudo systemctl set-default multi-user.target
sudo reboot
```
关闭图形界面，之后采用远程链接的方式进行配置。

### 4.固定IP配置
首先在命令行输入
```
ifconfig
```
查看当前服务器使用的网卡(例：enp4s0)。
修改在"/etc/netplan/"文件夹下的./yaml配置文件如下（该文件原有的内容保留）：
```
network:
    ethernets:
        enp4s0:
            dhcp4: no
            addresses: [192.168.1.xxx/24]
            optional: true
            gateway4: 192.168.1.1
            nameservers:
                    addresses: [192.168.1.1]
```
最后在命令行输入
```
sudo netplan apply 
```
完成静态ip配置。

### 5.ssh 服务安装

在拥有root权限的账号下使用
```
sudo apt-get install openssh-server
sudo /etc/init.d/ssh start
```
安装并启动ssh服务。

### 6. FTP服务安装

在拥有root权限的账号下使用
```
sudo apt-get install vsftpd
```
安装FTP服务，修改"/etc/init.d/vsftpd.conf"配置文件中
```
sudo vim /etc/init.d/vsftpd.conf
```
将"write_enable=YES"注释取消。
最后使用
```
sudo /etc/init.d/vsftpd start
```
