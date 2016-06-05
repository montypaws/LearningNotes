# ubuntu 安装Java开发环境


## JRE vs OpenJDK vs Oracle JDK

在我们继续了解如何安装Java之前，让我们快速地了解JRE、OpenJDK和Oracle JDK之间的不同之处。

- JRE（Java Runtime Environment），它是你运行一个基于Java语言应用程序的所正常需要的环境。如果你不是一个程序员的话，这些足够你的需要。

- JDK代表Java开发工具包，如果你想做一些有关Java的开发（阅读程序），这正是你所需要的。

- `OpenJDK`是Java开发工具包的开源实现， `Oracle JDK`是Java开发工具包的官方`Oracle`版本。尽管`OpenJDK`已经足够满足大多数的案例，但是许多程序比如`Android Studio`建议使用Oracle JDK，以避免UI/性能问题。
- 
## 查看本机java版本。

打开终端，输入以下指令：
> java -version


## 安装并配置JAVA环境

传统安装方法就是将JDK下载下来，然后安装执行解压和环境变量配置，为了节省配置所带来的麻烦，我们使用两外一种安装方式：

**在Ubuntu和Linux Mint上安装JRE**

打开终端，使用下面的命令安装JRE：

> sudo apt-get install default-jre
**在Ubuntu和Linux Mint上安装OpenJDK**

在终端，使用下面的命令安装OpenJDK Java开发工具包：

> sudo apt-get install default-jdk

*特殊地，如果你想要安装Java 7或者Java 6等等，你可以使用openjdk-7-jdk/openjdk-6jdk，但是记住在此之前安装openjdk-7-jre/openjdk-6-jre。*

**在Ubuntu和Linux Mint上安装Oracle JDK**

使用下面的命令安装，只需一些时间，它就会下载许多的文件，所及你要确保你的网络环境良好：


> sudo add-apt-repository ppa:webupd8team/java

> sudo apt-get update

> sudo apt-get install oracle-java8-installer

> sudo apt-get install oracle-java8-set-default


如果你想安装Java 7(i.e Java 1.7)，在上面的命令中用java7代替java8。
