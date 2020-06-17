---
title: JDK 安装
date: 2020-06-17 13:22:39
tags: Java
---

## JDK 安装

Java实际上分为三个版本：

- J2ME：Java 2 Micro Edition，适用于移动端的微型版本，曾广泛应用于机顶盒、车载系统、移动数字电话、个人数字助理（PDA）以及一系列嵌入式设备中，和大家接触最多的便是前几年的手机上搭载的J2SE环境，它几乎支撑了智能手机游戏的半边天。
- J2SE：Java 2 Standrad Edition，Java标准版本，只包含基础的Java类库，适用于在桌面端构建Java程序。它是J2ME和J2EE的基础。
- J2EE：Java 2 Enterprise Edition，Java企业版本，包含很多企业级特性，适用于简化企业开发的版本，包含一系列特性，如EJB、JAXB、JDBC API、CORBA、Servlet、JSP等等，实际上我们今天使用的JDK名义上是J2SE，但实际上他已经混入了J2EE的很多特性，算得上是J2EE版本了。

PS：Java在1.2版本之后统称为Java 2。当然，在当下“Java”这种叫法已经是熟路的叫法了。

**我们在开发中一般安装的是J2SE，若需要J2EE的一些特性我们一般是通过Maven去关联支持类库的。**
<escape><!-- more --></escape>

## JRE和JDK的区别

JRE是Java运行环境，只包含了Java程序运行时所需要的一系列类库（Binary），他可以被精简至更小。JDK则是Java开发包，除了包含一整套JRE还包含有一些列为开发者提供的工具（命令行工具和GUI工具）用于管理Java程序（jjs、jmc、jps、jvisualvm、jstatck、jhat、jdoc等等一系列工具），还包含大部分内置类库的源码（包括Native方法的源码）。

## 开始安装

### 1、下载安装包

- 在Oracle官网https://www.oracle.com/technetwork/java/javase/downloads/index.html找到你想要下载的安装包，由于2019年后java会开始收费，故我们只能使用java 8 固定版本。详细参见 [00.关于Java开始收费的说明](https://confluence.newegg.org/pages/viewpage.action?pageId=25157106)，因此我们只选择jdk1.8.0_181.rar之前的版本，并且只选择**小版本号为奇数**（如8u**191**）的安装包。
- 离线window x64的JDK压缩包

### **2、安装**

linux和windows都可以下载安装版和压缩包，安装过程略过。个人推荐压缩包，下载后解压即可。

### 3、配置环境变量

- Windows
  在Windows上需要配置的环境变量为JAVA_HOME、Path和Classpath（Classpath在Java1.5之后可以不设置）
  JAVA_HOME: 全路径，指向你的JDK目录，注意，**一定是jdkXXX这种目录**
  Path: .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\bin; (不要忘了前面的小点)

- Linux

  在Linux下配置环境变量需要注意，你需要根据你的实际情况选择是配置全局的环境变量还是该用户下的环境变量。全局环境变量请修改/etc/propfile，用户环境变量请修改~/.bash_profile

  在上述文件末尾加入：

  export JAVA_HOME=/usr/opt/java/jdkXXX
  export PATH=$JAVA_HOME/bin:$PATH
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

  保存后执行source <你刚刚修改的文件>

  

至此，配置完毕，请打开控制台输入javac命令验证。若出现响应则说明配置成功，若出现“不能识别的指令”则说明配置失败，请按照上文修改。