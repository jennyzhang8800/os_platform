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

![netstat -anptl]()

然后找到80,和8080端口对应的pid
输入：
```
kill -9 <pid>  
```
例如上图中，这里只有8080端口被占用，80没有被占用，我只需解除8080的占用即可(8080端口对应的PID为1213)。输入“kill -9 1213”

-----下面内容可选做-------

因为在gitlab这里用不到tomcat，而tomcatllf默认端口为8080与gitlab的unicorn端口冲突，所以为了不必要的麻烦，可以直接把tomcat的默认端口改掉。通过下的的操作实现把tomcat的端口改为8088:
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
 
![netstat -anptl]()

####(3)添加gitlab 包服务并安装包
输入下面的命令:
```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
出现下面的内容仓库说明己安装好：

![netstat -anptl]()

然后输入下面的命令安装包：
 ```
 sudo apt-get install gitlab-ce
 ```
 
 出现下面的内容说明安装好：
 
 ![netstat -anptl]()
 
 
####(4)配置并重启Gitlab

输入下面的命令：
```
sudo gitlab-ctl reconfigure
```

出现下面的内容说明配置成功：

![netstat -anptl]()

####(5)登录gitlab(访问http://localhost)
  在浏览器中输入：http://localhost 如果进入到登录页面，则说明gitlab己正确安装。
  首次访问gitlab,会直接重定向到设置密码屏幕，初始用户名是root。
##2.Open edX的安装
##3.Shibboleth

