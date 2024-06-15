## Linux虚拟机-配置

##### 一、 配置JDK

1. 首先虚拟机自带了一个jdk，需要先卸载掉。参考：[Linux上卸载JDK-CSDN博客](https://blog.csdn.net/weixin_44990104/article/details/117589372)

![](D:\godblessing\forwork\操作记录\figure\linux-1.png)

2. 安装jdk，参考链接：[Linux安装JDK及配置环境变量保姆级教程_linux配置jdk17环境变量-CSDN博客](https://blog.csdn.net/weixin_44904239/article/details/137240064)

   安装目录：/usr/local/jdk，版本：17.0.11

   ```
   java version "17.0.11" 2024-04-16 LTS
   Java(TM) SE Runtime Environment (build 17.0.11+7-LTS-207)
   Java HotSpot(TM) 64-Bit Server VM (build 17.0.11+7-LTS-207, mixed mode, sharing)
   ```

   安装好之后，存个快照

##### 二、安装mysql

[虚拟机中安装mysql 完整教程-简单实用-亲测有效（ CentOS7 版本）_虚拟机安装mysql-CSDN博客](https://blog.csdn.net/weixin_43884466/article/details/109741382)

账号：root  密码：123456

```
//启动
systemctl start mysqld.service
//查看状态
systemctl status mysqld.service
//重启服务
systemctl restart mysqld

//登录数据库
mysql -uroot -p
然后输入密码：123456  --linux输入密码时不会显示，输入完后enter就行
```

可能远程连接的适合会有问题，到时候再设置防火墙

存个快照

##### 三、安装maven

[Linux环境安装Maven（详细图文）_linux安装maven-CSDN博客](https://blog.csdn.net/m0_52985087/article/details/136155283#:~:text=1.检查当前环境是否安装maven 2.下载maven 3.上传maven压缩包 4.解压maven包,5.移动到%2Fusr%2Flocal目录下方便管理 6.配置maven环境变量 7.刷新配置文件 8.配置maven镜像仓库)

/usr/local/software/

存个快照

##### 四、安装iptables

本来是没有iptables的

安装[Unit iptables.service could not be found.(防火墙问题)-CSDN博客](https://blog.csdn.net/y368769/article/details/104490697)

[【操作系统----Linux】没有iptables时的解决办法-CSDN博客](https://blog.csdn.net/ningjiebing/article/details/89411005)

##### 五、安装tomcat

[Linux 虚拟机环境 Tomcat 的下载与安装 详细操作步骤_linux tomcat下载-CSDN博客](https://blog.csdn.net/CKS_GRASS/article/details/131149926)

第四部分已经设置好了防火墙，8080端口开放

linux虚拟机的ip地址是192.168.204.128

在浏览器输入http://192.168.204.128:8080

![](D:\godblessing\forwork\操作记录\figure\linux-2.png)

成功访问

```
进入/usr/local/tomcat/tomcat01/bin之后

//启动tomcat
sh startup.sh
//关闭tomcat
sh shutdown.sh
```

存个快照