---
title: "服务计算第一周作业：安装配置你的私有云（VMware+CentOS7）"
date: 2019-12-18T11:11:38+08:00
draft: false
categories: ["服务计算"]             
author: "Tifinity"  
---

# 服务计算实验一：安装配置你的私有云（VMware+CentOS7）
## 实验内容
### 使用VMware创建虚拟机
1. 下载Linux镜像：从老师给的地址或者其他网址下载Centos操作系统。

2. 创建虚拟机：基本操作此处不赘述。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171423128.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

3. 再添加一块虚拟网卡

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171453633.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
   第一块网卡为NAT模式，用于访问外网，第二块网卡为仅主机模式，用于与主机相连。

4. 开机

### 安装CentOS系统并配置

1. 按照提示一步步安装

   **【注意】创建用户时有两个用户名，以后使用的都是全小写的那个**

2. 登陆root账户

   `su` 并输入密码

   **【注意】在虚拟机中小键盘默认关闭，输入密码时请不要使用小键盘，以免误认为密码输错**

3. 开启虚拟网卡

   若配置完网卡无法ping通主机或者升级内核时出现错误，出现以下信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171512388.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

   **【注意】可能是因为CentOS的网卡默认是不开启的，需要手动开启**

   以下为操作过程：

   - 进入网卡的配置文件夹

     `cd /etc/sysconfig/network-scripts` 

   - `ls -a` 查看所有文件，找到 **ifcfg-en33** (或其他网卡名称)

     `vi ifcfg-en33` ，按 **i** 进入插入模式，将文件中的 onboot = no 改成 onboot = yes ，按 **ESC**，再按 **:wq** 保存并退出。

   - 对 **ifcfg-en37**（或其他名称，即另一个网卡）也进行同样操作。
   - 完成操作后需要重启虚拟机 `reboot` 或 重启网络 `service network restart`

3. 升级CentOS系统内核

   ` yum install wget` 先安装wget，一个从网络上自动下载文件的工具

   `yum update` 升级内核

   因为网络没有问题所以没有换源。

4. 配置虚拟网卡

   *本人此处截图是在安装完图形界面之后所截，只需中英文对照即可*

   首先`nmtui`配置网络

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171543898.jpg)

   先选择编辑连接，将NAT的网卡ipv4改成手动设置，随便设置一个ip，”/24“表示子网掩码“255.255.255.0”，网关改为“192.168.100.1”。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171553264.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
   在启用连接中查看连接状态，确保两个网卡（ens33，ens37）都启用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083117162086.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

   使用 `nmcli` 查看网络连接，类似于windows的 `ipconfig`。

   若安装 net-tools 即` yum install net-tools`  之后，还可以使用 `ip addr` 查看网卡和ip。

   ![\[外链图片转存失败(img-eek1otdm-1567242807176)(T:\TH\大三上\服务计算\1\1567187691(3)\].jpg)](https://img-blog.csdnimg.cn/20190831171642398.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

   这里的ens33为NAT模式的网卡，ens37为仅主机模式的网卡，此处仅为本人的网卡信息，不同虚拟机网卡名称可能不同，此时的设置即为正确设置。

   然后ping：

   ping主机网关`ping 192.168.100.1` 

   ping外网网关`ping 192.168.78.1` 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171711635.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083117172373.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
   主机使用ssh登陆虚拟机：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171736643.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
   如果无法ping通，可能需要关闭防火墙。

   关于防火墙的操作：

   ```
   systemctl start firewalld   /*启动*/
   systemctl stop firewalld    /*关闭*/
   systemctl status firewalld  /*查看状态*/
   systemctl disable firewalld /*开机禁用*/
   systemctl enable firewalld  /*开机启用*/
   ```

   然后关闭主机的防火墙，以Windows10为例：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083117180981.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171817493.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171825776.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
   此时主机和虚拟机应该可以互相ping通。

   到此Base虚拟机配置完毕，需要用到多个虚拟机时，新建比较麻烦，这时候就需要克隆来快速地创建多个虚拟机。

   虚拟机->管理->克隆（在此之前需要关闭虚拟机），记得选择重新初始化网卡MAC地址，并且需要修改手动设置地址的网卡ip地址，自动获取则不需要

   ### 设置端口映射

   要想通过局域网中其他主机直接ping通虚拟机，需要设置端口映射。

   VMware -> 编辑 -> 虚拟网络编辑器 -> NAT设置 -> 添加

   输入主机端口（建议5000以上），类型为TCP，虚拟机IP为NAT网卡的IP，端口为3389（3389是提供远程桌面服务的端口），描述可不写，然后确认。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171912822.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171933414.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
此时使用其他主机对本主机的5005端口的访问就会被转发给虚拟机的3389端口，可以直接ping通虚拟机，也可以使用ssh直接连接虚拟机，连接的目标为**主机ip：主机端口**。

### 安装图形界面

`yum groupinstall "GNOME Desktop"` 下载图形界面

若此时出现黑屏，等待几分钟，不要轻易重启，会发生图形界面安装不完全的情况，等待compelete出现再进行操作。

`ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target`  设置开机启动GUI

`reboot` 重启

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831171945583.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
终端在 应用程序->收藏->终端，图形界面默认不是root用户，记得`su`登陆root用户。

### 远程连接虚拟机

想要远程连接到虚拟机，最后需要安装远程连接的安全证书xrdp。

首先，xrdp不在的yum默认库中，需要安装补充开源库epel。

`yum install epel-release`

然后安装xrdp。

`yum install xrdp`

如果安装过程中出现错误：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831172000113.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

原因是xrdp的版本太高导致的bug，需要自行下载较低版本的xrdp手动安装。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083117201065.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831172026714.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

`rpm -ivh /*文件名*/` 

然后再 `yum install -y xrdp`, 加上-y默认所有选项选yes。

`yum install tigervnc-server`

开启xrdp服务

`systemctl start xrdp`
`systemctl enable xrdp`

到这里，就已经可以用局域网（即校园网）里的任意一台电脑远程直接连接虚拟机了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831172041417.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831172049966.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083117205814.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20190831172107329.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
成了·~·