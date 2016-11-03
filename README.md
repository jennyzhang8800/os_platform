# MOOC 平台
以下配置文件涉及的内容有：gitlab的安装，Open edX的安装，利用Shibboleth实现以gitlab和Open edX作为两个SP实现SSO（单点登录）。
<hr/>
##1.gitlab的安装
###1.1环境

    gitlab版本：GitLab CE Omnibus 
    
    操作系统：ubuntu 12.04 64bit
    
###1.2安装gitlab

   下面的安装步骤参考的是[官方文档](https://about.gitlab.com/downloads/#ubuntu1204)
    
####(1). 安装前查看端口状态，并把80和8080端口解除占用

   由于gitlab安装结束后会占用80和8080端口，所以如果你的操作系统中己有apache,tomcat那么这两个端口是处于占用状态的，会导致安装gitlab后访问localhost时出现502错误。因此，我们先释放这两个端口。
   
输入下面的命令查看端口状态：
```
     sudo netstat -anptl
```
如下图所示：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-0.png)

然后找到80,和8080端口对应的pid
输入：
```
kill -9 <pid>  
```
例如上图中，这里只有8080端口被占用，80没有被占用，我只需解除8080的占用即可(8080端口对应的PID为1213)。输入“kill -9 1213”

-----下面内容可选做-------

因为在gitlab这里用不到tomcat，而tomcat默认端口为8080,与gitlab的unicorn端口(8080)冲突，所以为了不必要的麻烦，可以直接把tomcat的默认端口改掉。通过下的的操作实现把tomcat的端口改为8088:
```
vi /etc/tomcat7/server.xml

```
找到8080端口，如下
```
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />
```
把它改为8088，如下：
```
 <Connector port="8088" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
               redirectPort="8443" />
```
保存更改。
输入下面的命令重启tomcat：
```
sudo service tomcat7 restart
```

----------end--------------
####(2)安装和配置必要的依赖

输入下面的命令：
```
sudo apt-get install curl openssh-server ca-certificates postfix
```
在安装过程中选择“Internet Site”。

 出现下面的内容说明安装成功：
 
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-1.png)

####(3)添加gitlab 包服务并安装包
输入下面的命令:
```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
出现下面的内容仓库说明己安装好：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-2.png)

然后输入下面的命令安装包：
 ```
 sudo apt-get install gitlab-ce
 ```
 
 出现下面的内容说明安装好：
 
 ![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-3.png)
 
 
####(4)配置并重启Gitlab

输入下面的命令：
```
sudo gitlab-ctl reconfigure
```

出现下面的内容说明配置成功：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-4.png)

####(5)登录gitlab(访问http://localhost)
  在浏览器中输入：http://localhost 如果进入到登录页面，则说明gitlab己正确安装。
  首次访问gitlab,会直接重定向到设置密码屏幕，初始用户名是root。
##2.Open edX的安装
###2.1 环境

Open edX版本：Open edX Fullstack

操作系统：Ubuntu12.04 64bit

###2.2 服务器要求

+ Ubuntu 12.04 amd64
+ 最小4GB内存
+ 至少一个2GHz CPU 或 EC2 计算单元
+ 最小25GB可用硬盘，生产服务器需要最小50GB可用硬盘

###2.3 安装Open edX

以下安装步骤参考的是[此文档](https://openedx.atlassian.net/wiki/display/OpenOPS/Native+Open+edX+Ubuntu+12.04+64+bit+Installation#title-heading)，采用的是自动安装的方法进行安装。

####(1) 更新源和升级软件

依次输入下面的三条命令：
```
sudo apt-get update -y  
sudo apt-get upgrade -y  
sudo reboot  
```
####(2)安装Open edX

依次输入下面的两条命令：
```
wget https://raw.githubusercontent.com/edx/configuration/master/util/install/ansible-bootstrap.sh -O - | sudo bash
wget https://raw.githubusercontent.com/edx/configuration/master/util/install/sandbox.sh -O - | bash
```
**注意!** 一般来说第一条命令可以正常执行。第二条命令就开始安装了，耗时比较长，如果顺利的话2个小时左右，但一般都不会一次成功…………出现错误了（会以红色标示），安装会中止，这时候就需要根据出错的提示解决掉错误，然后重新执行第二条命令，如此反复，直到没有错误（failed=0）,就安装成功了！

安装成功后访问：http://localhost 可进行学生端LMS访问，出现登录页面。

访问：http://localhost:18010 可进行Studio访问

---------------------
下面是我在安装过程中出现的错误，及解决方法：

####错误1：
[insights | run r.js optimizer]*******************************************************************

找不到jquery.js脚本。错误提示如下：
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-0.png)

####解决方法：
(1)下载jquery.js 。下载地址：https://jquery.com/  ,把下载的jquery-3.1.1.js(任何一个版本都可以) 重命名为jquery.js

(2)把下载好的jquery.js文件放到/edx/app/insights/edx_analytics_dashboard/analytics_dashboard/static/bower_components/jquery/dist/目录下

(3) 重新执行
```
wget https://raw.githubusercontent.com/edx/configuration/master/util/install/sandbox.sh -O - | bash
```

####错误2：
[insights | run collectstatics]******************************************************************************

stderr:CommandError:Anerror occurred during rendering .....

错误截图如下：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-1.png)

####解决方法：
使用java-7-openjdk 设置环境变量，通过以下步骤实现：

(1)输入下面的命令：
```
sudo update-alternatives --config java  
```
选择java-7-openjdk

如下图所示：
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-2.png)

(2)设置环境变量：

输入下面的命令：
```
JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java  
```

把上面的/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java 换成你实际选择的java路径

(3)重新执行命令
```
wget https://raw.githubusercontent.com/edx/configuration/master/util/install/sandbox.sh -O - | bash
```
----------------------------
如果出现红色（fail）,但是又ignoring，则可以不用管，还是会安装成功的。如出现[mysql | Look for mysql 5.6]失败，后面它ignoring了，也不会影响安装成功。
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-3.png)

##3.Shibboleth

