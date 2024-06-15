## linux上部署项目

要解决的问题

1. 没有raft-java-core.jar包，要从mvn仓库里下载到本地，再手动加载进项目里

   jar包从https://mvnrepository.com/里面搜

   添加方法：[linux下maven项目中添加本地jar包_linux环境下maven引用本地jar包-CSDN博客](https://blog.csdn.net/u014763678/article/details/83513521)

2. 报错信息：

![](D:\godblessing\forwork\操作记录\figure\linux部署项目-1.png)

[解决Maven编译项目报错：Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile_[error\] failed to execute goal org.apache.maven.pl-CSDN博客](https://blog.csdn.net/qq_53483101/article/details/136298392)

可能是jdk版本不一样，需要换成跟pom.xml里面的1.7版本，试一下

[Linux上安装多个JDK，并随意切换版本_linux安装两个jdk-CSDN博客](https://blog.csdn.net/weixin_41753664/article/details/122452178)



重新安装了jdk1.7.0和maven3.6.1版本（并修改了环境变量）之后，这个报错解决



3. 报错信息

![](D:\godblessing\forwork\操作记录\figure\linux部署项目-2.png)

问题：package com.baidu.brpc.client does not exist

4. 直接导入已经配置好的虚拟机

[Vmware添加已配置好的虚拟机_原先有vmware虚拟机怎么加载-CSDN博客](https://blog.csdn.net/u012453843/article/details/70045136)

注意要修改网关，不然ping不上

