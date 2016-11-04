# MOOC 平台
以下配置文件涉及的内容有：gitlab的安装，Open edX的安装，利用Shibboleth实现以gitlab和Open edX作为两个SP实现SSO（单点登录）。
<hr/>

**目录**

* [gitlab的安装](https://github.com/jennyzhang8800/os_platform#1gitlab的安装)
* [Open edX的安装](https://github.com/jennyzhang8800/os_platform#2open-edx的安装)
* [shibboleth](https://github.com/jennyzhang8800/os_platform#3shibboleth)
 + [部署LDAP服务器](https://github.com/jennyzhang8800/os_platform#31部署ldap服务器)
 + [部署IdP](https://github.com/jennyzhang8800/os_platform#32部署idp)


<hr/>

# 1.gitlab的安装
## 1.1环境

    gitlab版本：GitLab CE Omnibus 
    
    操作系统：ubuntu 12.04 64bit
    
## 1.2安装gitlab

   下面的安装步骤参考的是[官方文档](https://about.gitlab.com/downloads/#ubuntu1204)
    
### (1). 安装前查看端口状态，并把80和8080端口解除占用

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
### (2)安装和配置必要的依赖

输入下面的命令：
```
sudo apt-get install curl openssh-server ca-certificates postfix
```
在安装过程中选择“Internet Site”。

 出现下面的内容说明安装成功：
 
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-1.png)

### (3)添加gitlab 包服务并安装包
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
 
 
### (4)配置并重启Gitlab

输入下面的命令：
```
sudo gitlab-ctl reconfigure
```

出现下面的内容说明配置成功：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-install-4.png)

### (5)登录gitlab(访问http://localhost)
  在浏览器中输入：http://localhost 如果进入到登录页面，则说明gitlab己正确安装。
  首次访问gitlab,会直接重定向到设置密码屏幕，初始用户名是root。
  
# 2.Open edX的安装
## 2.1 环境

Open edX版本：Open edX Fullstack

操作系统：Ubuntu12.04 64bit

## 2.2 服务器要求

+ Ubuntu 12.04 amd64
+ 最小4GB内存
+ 至少一个2GHz CPU 或 EC2 计算单元
+ 最小25GB可用硬盘，生产服务器需要最小50GB可用硬盘

## 2.3 安装Open edX

以下安装步骤参考的是[此文档](https://openedx.atlassian.net/wiki/display/OpenOPS/Native+Open+edX+Ubuntu+12.04+64+bit+Installation#title-heading)，采用的是自动安装的方法进行安装。

### (1) 更新源和升级软件

依次输入下面的三条命令：
```
sudo apt-get update -y  
sudo apt-get upgrade -y  
sudo reboot  
```
### (2)安装Open edX

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

#### 错误1：
[insights | run r.js optimizer]*******************************************************************

找不到jquery.js脚本。错误提示如下：
![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-0.png)

#### 解决方法：
(1)下载jquery.js 。下载地址：https://jquery.com/  ,把下载的jquery-3.1.1.js(任何一个版本都可以) 重命名为jquery.js

(2)把下载好的jquery.js文件放到/edx/app/insights/edx_analytics_dashboard/analytics_dashboard/static/bower_components/jquery/dist/目录下

(3) 重新执行
```
wget https://raw.githubusercontent.com/edx/configuration/master/util/install/sandbox.sh -O - | bash
```

#### 错误2：
[insights | run collectstatics]******************************************************************************

stderr:CommandError:Anerror occurred during rendering .....

错误截图如下：

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/open-edx-install-1.png)

#### 解决方法：
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


---

# 3.Shibboleth

## 3.1 部署LDAP服务器.


### 3.1.1 安装OpenLDAP及可视化工具

[我们所使用的OpenLDAP镜像](http://www.turnkeylinux.org/openldap)

该镜像已包含phpldapadmin(方便网页端访问LDAP),若从其他渠道安装LDAP,请自行安装该工具。

### 3.1.2 利用eduperson.ldif创建模式eduPerson

    ldapadd -Y EXTERNAL -H ldapi:/// -f <path of eduperson.ldif>
    
### 3.1.3 登录管理员账号创建存储用户的结点,例如ou=Users,dc=cscw

或者使用命令行添加Users节点:

	ldapadd -x -D "cn=admin,dc=cscw" -W -f create_group.ldif
 
把上面代码中的"cn=admin,dc=cscw" 换成你登录ldapadmin时的LoginDN。LoginDN的位置如下图所示

### 3.1.4 test_ldap.py可用于测试OpenLDAP是否正常工作(修改其中的ip,baseDN以及searchFilter参数,保持与IDP中的配置一致,
详细可参考shibboleth仓库中的配置文件)

### 3.1.5 create_user.ldif用于手动创建用户(修改其中的用户参数)

    ldapadd -x -D "cn=admin,dc=cscw" -W -f create_user.ldif 
    
把上面代码中的"cn=admin,dc=cscw" 换成你登录ldapadmin时的LoginDN。

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/openLdap-0.png)

create_user.ldif的内容如下：
```
dn: uid=Tom,ou=Users,dc=cscw
objectClass: inetOrgPerson
objectClass: top
objectClass: eduPerson
uid: Tom
sn: Tom
givenName:Tom
cn:Tom
mail:yannizhang8800@163.com
userPassword: 123456
eduPersonPrincipalName:yannizhang8800@163.com
```

+ 把代码中所有的Tom换成你要新建的用户名;

+ yannizhang8800@163.com 换成该用户名对应的邮箱，注意邮箱应是唯一的，因为Open edX是以邮箱为主键的。

+ 123456 换成该用户的密码

+ 第一行中的 ou=Users,dc=cscw换成你的实际路径。

如下图，我们创建的新用户Tom，位于dc=cscw下面的ou=Users下，所以第一行写的是ou=Users,dc=cscw

![netstat -anptl](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/openldap-1.png)

## 3.2 布署IdP
### 3.2.1 环境
IDP版本：2.4.4

操作系统：Ubuntu12.04 64bit

### 3.2.2 安装IdP

#### 3.2.2.1安装oracle jdk
依次输入下面的4条命令：
```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
```

#### 3.2.2.2安装apache及tomcat7
依次输入下面的4条命令：
```
sudo apt-get install apache2
sudo a2enmod ssl
sudo a2enmod proxy_ajp
sudo apt-get install tomcat7
```

#### 3.2.2.3配置apache和tomcat

**(1)更改本机域名**

输入下面的命令：
```
sudo vi /etc/hosts  
```

加入下面的内容：
```
127.0.0.1	idp.edx.org	shibboleth
```
把idp.edx.org换成你的IdP机器的域名

**(2)更改apache ServerName**

输入下面的命令：
```
sudo vi /etc/apache2/apache2.conf
```

加入下面的内容：
```
ServerName idp.edx.org
```
把idp.edx.org换成你的IdP机器的域名

**(3)设置JAVA_OPTS 环境变量**

输入下面的命令：

```
sudo vi /etc/init.d/tomcat7  
```

找到JAVA_OPTS ，添加参数：-XX:+UseG1GC -Xmx1500m -XX:MaxPermSize=128m 

如下图所示：

![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-install-2.png)

**(4)设置POST提交限制**

输入下面的命令：
```
sudo vi /etc/tomcat7/server.xml  
```

找到 Connector ,添加属性maxPostSize ，设置值为100K(100000)

如下图：

![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-install-3.png)

**(5)使用Context Deployment Fragment**

输入下面的命令：

```
sudo vi /etc/tomcat7/Catalina/localhost/idp.xml 
```

输入下面的命令创建文件 *TOMCAT_HOME*/conf/Catalina/localhost/idp.xml （*TOMCAT_HOME* 是你的tomact安装路径，这里是/etc/tomcat7）
```
sudo vi /etc/tomcat7/Catalina/localhost/idp.xml

```
在新建的文件idp.xml中添加下面的内容：
```
<Context docBase="/opt/shibboleth-idp/war/idp.war"
         privileged="true"
	 antiResourceLocking="false"
	 antiJARLocking="false"
	 unpackWAR="false"
	 swallowOutput="true"
	 cookies="false" />
```

#### 3.2.2.4 安装IdP
输入下面的命令：
```
sudo wget http://shibboleth.net/downloads/identity-provider/2.4.4/shibboleth-identityprovider-2.4.4-bin.zip
sudo unzip shibboleth-identityprovider-2.4.4-bin.zip
cd shibboleth-identityprovider-2.4.4
sudo JAVA_HOME=/usr/lib/jvm/java-7-oracle ./install.sh
sudo chown -R tomcat6:tomcat6 /opt/shibboleth-idp
```

安装过程中安装路径不用改。hostname改为：idp.edx.org。(把idp.edx.org换成你的IdP机器的域名),会要求设置密码：这里设为shibboleth.如下图所示：

![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-install-5.png)
![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-install-6.png)

检查IdP是否安装成功，访问：http://idp.edx.org:8080/idp/profile/Status 把idp.edx.org换成你设置的hostname ，如果输出OK则说明安装正确

![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-install-8.png)

图片中Status文件的内容是OK

如果出现错误,请查看日志:/opt/shibboleth-idp/logs/idp-process.log

[官方的问题解答列表](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPTroubleshootingCommonErrors#NativeSPTroubleshootingCommonErrors-Unabletolocatemetadataforidentityprovider(https://identities.supervillain.edu/idp/shibboleth).)


### 3.2.3 配置IdP与OpenLdap连接

#### 3.2.3.1 在配置前,请使用test_ldap.py保证LDAP正常工作。

在IdP机器上运行test_ldap.py脚本。test_ldap.py脚本内容如下：

```
import ldap

ldap.set_option(ldap.OPT_X_TLS_REQUIRE_CERT, ldap.OPT_X_TLS_NEVER)
l = ldap.initialize("ldap://192.168.1.138")
l.set_option(ldap.OPT_REFERRALS, 0)
l.set_option(ldap.OPT_PROTOCOL_VERSION, 3)
l.set_option(ldap.OPT_X_TLS,ldap.OPT_X_TLS_DEMAND)
l.set_option( ldap.OPT_X_TLS_DEMAND, True )
l.set_option( ldap.OPT_DEBUG_LEVEL, 255 )
baseDN = "ou=Users,dc=cscw"
searchScope = ldap.SCOPE_SUBTREE
retrieveAttributes = None
searchFilter = "cn=*Tom*"
ldap_result_id = l.search(baseDN, searchScope, searchFilter)
result_set = []
while 1:
    result_type, result_data = l.result(ldap_result_id, 0)
    if (result_data == []):
        break
    else:
        if result_type == ldap.RES_SEARCH_ENTRY:
            result_set.append(result_data)
    print result_set
```

把上图中的
+ l = ldap.initialize("ldap://192.168.1.138")   中的192.168.1.138改为openldap的IP
+ baseDN = "ou=Users,dc=cscw"  改为你的baseDN
+ searchFilter = "cn=*Tom*"    *Tom*改为dc=cscw ou=Users下的一个用户名

在IdP机器上运行test_ldap.py后应该返回用户名为Tom的一些属性信息（这些属性就是3.1.5中创建用户Tom时所设置的属性）。如下图所示：

![picture-test_ldap.py](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/test_ldap.py%E7%BB%93%E6%9E%9C.png)

如果能正常获得上图所示的用户信息，则说明ldap正常工作，可以进行配置ldap验证了，通过以下步骤实现：

#### 3.2.3.2 属性定义attribute-resolver.xml

输入下面的命令：

`sudo vi /opt/shibboleth-idp/conf/attribute-resolver.xml`

(1)在Attribute Definitions下面加入下面的内容：

```
<resolver:AttributeDefinition xsi:type="ad:Simple" id="uid" sourceAttributeID="uid">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:uid" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:0.9.2342.19200300.100.1.1" friendlyName="uid" />
    </resolver:AttributeDefinition>

    <resolver:AttributeDefinition xsi:type="ad:Simple" id="email" sourceAttributeID="mail">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:mail" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:0.9.2342.19200300.100.1.3" friendlyName="mail" />
    </resolver:AttributeDefinition>

<resolver:AttributeDefinition xsi:type="ad:Simple" id="commonName" sourceAttributeID="cn">
    <resolver:Dependency ref="myLDAP" />
    <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:cn" />
    <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:2.5.4.3" friendlyName="cn" />
</resolver:AttributeDefinition>

<resolver:AttributeDefinition xsi:type="ad:Simple" id="eppn" sourceAttributeID="eduPersonPrincipalName">
    <resolver:Dependency ref="myLDAP" />
    <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" />
    <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eppn" />
</resolver:AttributeDefinition>

```

如下图所示：

![idp-ldap-conf-0](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-ldap-conf-0.png)


(2)取消下面内容的注释，并修改参数：

```
<resolver:DataConnector id="myLDAP" xsi:type="dc:LDAPDirectory"
    ldapURL="ldap://192.168.1.138:389" 
    baseDN="ou=Users,dc=cscw" 
    principal="cn=admin,dc=cscw"
    principalCredential="1234567">
    <dc:FilterTemplate>
        <![CDATA[
            (uid=$requestContext.principalName)
        ]]>
    </dc:FilterTemplate>
    </resolver:DataConnector>
```

+ 第二行 192.168.1.138:389 ， 改为 你的ldap的IP地址加上端口号389 ,  注意：如果你用的是http协议，一定不能把端口号389省去！
+ 第三行 baseDN 改为你存放的用户在ldap中的baseDN
+ 第四行 principal 改为登录ldap时的Login DN (如3.1.5的图片所示的Login DN)
+ 第五行 principalCredential 改为登录ldap时的密码（如3.1.5的图片所示的Password）


#### 3.2.3.3 更改attribute-filter.xml

输入下面的命令:

`sudo vi /opt/shibboleth-idp/conf/attribute-filter.xml`

加入下面的内容：

```
<afp:AttributeRule attributeID="uid">
    <afp:PermitValueRule xsi:type="basic:ANY" />
</afp:AttributeRule>

<afp:AttributeRule attributeID="email">
    <afp:PermitValueRule xsi:type="basic:ANY" />
</afp:AttributeRule>

<afp:AttributeRule attributeID="commonName">
    <afp:PermitValueRule xsi:type="basic:ANY" />
</afp:AttributeRule>

<afp:AttributeRule attributeID="eppn">
    <afp:PermitValueRule xsi:type="basic:ANY" />
</afp:AttributeRule>
```
如下图所示：

![picture-idp-ldap-conf-2](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-ldap-conf-2.png)


#### 3.2.3.4 更改handler.xml

输入下面的命令：

`sudo vi /opt/shibboleth-idp/conf/handler.xml`

取消下面内容的注释：

```
<!--  Username/password login handler -->
<ph:LoginHandler xsi:type="ph:UsernamePassword"
    jaasConfigurationLocation="file:///opt/shibboleth-idp/conf/login.config">
<ph:AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:
    PasswordProtectedTransport</ph:AuthenticationMethod>
</ph:LoginHandler></code>
```

注释下面的内容：

```
<ph:LoginHandler xsi:type="ph:RemoteUser">
        <ph:AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified</ph:AuthenticationMethod>
    </ph:LoginHandler>
```

```
 <ph:LoginHandler xsi:type="ph:PreviousSession">
        <ph:AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:PreviousSession</ph:AuthenticationMethod>
    </ph:LoginHandler>
```

如下图所示：

![idp-ldap-conf-3](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-ldap-conf-3.png)

#### 3.2.3.5 更改login.config

输入下面的命令：

`sudo vi /opt/shibboleth-idp/conf/login.config`

取消下面内容的注释：
```
edu.vt.middleware.ldap.jaas.LdapLoginModule required
    ldapUrl="ldap://192.168.1.138:389"
    baseDn="ou=Users,dc=cscw"
    ssl="false"
    //userFilter="uid={0}";
    userField="uid";
```

如下图所示：

![idp-ldap-conf-5](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-ldap-conf-5.png)

#### 3.2.3.6 测试idp 与ldap 连接是否正常

运行aacli.sh 脚本可测试两者连接是否正常

输入下面的命令：

```
cd /opt/shibboleth-idp/bin
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
./aacli.sh --configDir=/opt/shibboleth-idp/conf/ --principal=Tom

```
可以看到下图所示的结果，即ldap返回的Tom用户的有关属性（uid,cn,eppn,mail） 以及属性的值(Tom,Tom,yannizhang8800@163.com,yannizhang8800@163.com),如果能返回这些信息，说明idp与ldap能正常连接。
![idp-conf-6](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-conf-6.png)

### 3.3 配置IdP与SP连接
  3.3要在完成SP端的原数据生成之后才可以做，现在可以先跳过，等生成了sp-metadata.xml之后再回来做。

  在IdP端对与SP连接的配置只需两步：
  
#### 3.3.1 把SP端的原数据拷贝到IdP端的/opt/shibboleth-idp/metadata目录下

在SP端运行下面的命令：
```
scp sp-metadata.xml username@idp-ip:/opt/shibboleth-idp/metadata

```
然后到IdP端运行下面的命令：
```
chown tomcat7:tomcat7 sp-metadata.xml
```
把上述代码中的username 换成IdP机器的用户名，idp-ip换成IdP机器的IP地址。

如果直接运行上面代码没有权限直接持贝，则可采取下面的方法 ：

在SP机器输入面的命令：
```
scp sp-metadata.xml zyni@192.168.1.137:/home/zyni
```
再到IdP机器输入下面的命令：
```
cp /home/zyni/sp-metadata.xml /opt/shibboleth-idp/metadata
chown tomcat7:tomcat7 sp-metadata.xml
```
#### 3.3.2 修改relying-party.xml

在IdP端输入下面的命令：
```
vi /opt/shibboleth-idp/conf/relying-party.xml
```
在"Metadata Configuration"部分，加入下面的代码：
```
<metadata:MetadataProvider xsi:type="FilesystemMetadataProvider"
    xmlns="urn:mace:shibboleth:2.0:metadata" id="SPMETADATA"
    metadataFile="/opt/shibboleth-idp/metadata/sp-metadata.xml" />
```
如下图所示：
![idp-sp-metadata-config](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/idp-sp-metadta-config.png)

## 3.3 布署Open edX端的SP

下面是在Open edX机器上进行的配置，参考的是[官方配置文档](http://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/configuration/tpa/index.html) ,通过以下步骤实现：
### 3.3.1 打开第三方认证特性

由于默认情况下open edx的第三方认证是不可用的，因此首先需要打开第三方认证特性。通过下面的操作实现：

(1)输入下面的命令：
```
sudo su  
vi /edx/app/edxapp/lms.env.json 
```
(2)在lms.env.json文件中，把
"ENABLE_COMBINED_LOGIN_REGISTRATION" 和 "ENABLE_THIRD_PARTY_AUTH"
这两个属性的值都改为true，保存修改。
```
"FEATURES" : {
    ...
    "ENABLE_COMBINED_LOGIN_REGISTRATION": true,
    "ENABLE_THIRD_PARTY_AUTH": true
}
```

如下图所示:
![open-edx-sp-conf-0]()

### 3.3.2 配置Open edX作为SP
 首先生成credential key pair（即生成公钥和私钥）以保证SP和IDP之间数据传输的安全性，然后利用credential key pair生成SP端的原数据。
 
#### 3.3.2.1 生成公钥和私钥
（1）输入下面的命令：
```
openssl req -new -x509 -days 3652 -nodes -out saml.crt -keyout saml.key
```
(2)   输入要求填写的信息(Country Name, state or Province Name,等)，就会在当前路径下生成公钥saml.crt和私钥saml.key

如下图所示：

![open-edx-sp-conf-1]()

生成结果如下：生成公钥saml.crt和私钥saml.key

![open-edx-sp-conf-2]()

#### 3.3.2.2 把生成的公钥和私钥添加到LMS配置文件中
（1）打开lms.auth.json文件

输入下面的命令：
```
vi /edx/app/edxapp/lms.auth.json  
```

（2）把saml.crt文件的内容复制到SOCIAL_AUTH_SAML_SP_PUBLIC_CERT属性中。

 **注意：** saml.crt文件的内容去掉开头和结尾的注释，内容去掉换行。使之为一个单行的字符串添加入SOCIAL_AUTH_SAML_SP_PUBLIC_CERT属性中。
 
 saml.crt文件内容如下图：
 
 ![open-edx-sp-conf-3]()
 
  lms.auth.json加入公钥私钥内容后如下：
  
   ![open-edx-sp-conf-4]()
   
（3）把saml.key文件的内容复制到SOCIAL_AUTH_SAML_SP_PRIVATE_KEY属性中。

  saml.key内容如下。也是只复制中间的内容（内容去掉开头和结尾的注释，内容去掉换行。使之为一个单行的字符串）
   
   ![open-edx-sp-conf-5]()
   
#### 3.3.2.3 配置open edx作为SP，生成原数据

**（1）登录到Django 管理界面。**

URL为http://{your_URL}/admin ，如这里的是:http://cherry.os.cs.tsinghua.edu.cn

界面如下图：输入有管理员权限的用户名密码

   ![open-edx-sp-conf-6]()
   
 登录成功后，看到如下的界面：
 
 ![open-edx-sp-conf-7]()
 
**（2）在Third_Party_Auth下面的SAML Configuration 中点击Add.**

如下图：

 ![open-edx-sp-conf-8]()
 
**（3）选中Enabled，输入下面的信息：**
    
+ **Entity ID**: 这会作为生成的原数据中的entity_id。一般输入服务器名，如： http://saml.mydomain.com/. （我这里的是http://cherry.os.cs.tsinghua.edu.cn）

+ **Site**: 指定作为SP的站点

+ **Organization Info**: 组织信息，如下，把下面改为你的edx信息
```
{
   "en-US": {
       "url": "http://www.mydomain.com",
       "displayname": "{Complete Name}",
       "name": "{Short Name}"
   }
}
```
改为：
```
{
   "en-US": {
       "url": "http://cherry.os.cs.tsinghua.edu.cn",
       "displayname": "Tsinghua University",
       "name": "tsinghua"
   }
}
```

+ **Other config str**: 定义IdP原数据文件的安全设置，保持默认即可。如下：
```
{
   "SECURITY_CONFIG": {
     "signMetadata": false,
     "metadataCacheDuration": ""
   }
}
```
点右下角的保存按钮,如下图所示：

 ![open-edx-sp-conf-9]()
 
  ![open-edx-sp-conf-10]()
  
保存后结果如下图:
 
  ![open-edx-sp-conf-11]()

**（4）在 {your LMS URL}/auth/saml/metadata.xml 中可以看到生成的SP端原数据**

如我这里的是：http://cherry.os.cs.tsinghua.edu.cn/auth/saml/metadata.xml

如下图所示：

 ![open-edx-sp-conf-12]()
 
原数据的内容在IdP配置的时候会用到。

如果上面的步骤中没有生成成功原数据，在3.3.2.2做完后先重启edx服务，然后再进行3.3.2.3

重启edx服务的命令如下：

```
sudo /edx/bin/supervisorctl restart edxapp:   
sudo /edx/bin/supervisorctl restart edxapp_worker:   
```
#### 3.3.2.4 确保SAML 认证后端己加载

如果你没有改过/edx/app/edxapp/lms.env.json文件的设置（向它添加过THIRD_PARTY_AUTH_BACKENDS设置），这一步可以什么都不过，跳过就行。

### 3.3.3 使SP和IdP结合

#### 3.3.3.1交换原数据

(1) 先把3.3.2.3中生成的SP端原数据，保存到IdP端


我这里把http://cherry.os.cs.tsinghua.edu.cn/auth/saml/metadata.xml 的内容保存为单独的文件，命名为edx-metadata.xml


然后把edx-metadata.xml 拷贝到IdP机器中的/opt/shibboleth-idp/metadata/目录下。


(2)更改IdP配置文件relying-party.xml

在IdP机器输入下面的命令：

```
vi /opt/shibboleth-idp/conf/relying-party.xml   
```
在Metadata Configuration的部分加入面的内容：
```
<metadata:MetadataProvider xsi:type="FilesystemMetadataProvider"
                xmlns="urn:mace:shibboleth:2.0:metadata" id="SPMETADATA-EDX"
                metadataFile="/opt/shibboleth-idp/metadata/edx-metadata.xml" />
```

（你也可以不上传原数据文件到IdP机器，直接在把上面的 metadataFile="/opt/shibboleth-idp/metadata/edx-metadata.xml" 改为
metadataFile="http://cherry.os.cs.tsinghua.edu.cn/auth/saml.metadata.xml"
)

如下图：
 
  ![open-edx-sp-conf-12]()
  
 (3) 把IdP原数据（位于IdP机器中的/opt/shibboleth-idp/metadata/目录下，一般为idp-metadata.xml文件），保存到一个可以用URL获得的地方。
 
我这里把IdP原数据保存到gitlab仓库中,链接为：https://raw.githubusercontent.com/jennyzhang8800/os_platform/master/idp-metadata.xml

#### 3.3.3.2 添加并使SAML IdP可用

**（1）登录到Django 管理界面。**

 管理界面的URL，如http://cherry.os.cs.tsinghua.edu.cn/admin
 
**（2）在Third_Party_Auth下的Provider Configuration(SAML IdPs) 点击Add**

 如下图：
 
   ![open-edx-sp-conf-13]()
   
 **(3)输入下面的信息：**
 
+ **Icon class**: 这里输入fa-university.
+ **Name** 在登录页面出现的IdP名。如：Tsinghua_OS.
+ **Secondary**: 选中这一项则在登录时会有一个中间页列表出现。这里选中
+ **Backend name**: 默认为tpa-saml,不用改
+ **Site**: 站点名，输入edx的站点名。如：cherry.cs.tsinghua.edu.cn
+ **IdP slug**: 唯一识别这个IdP的名称。不含空格，可以作为CSS类名。如：shibboleth
+ **Entity ID**: 与IdP原数据（idp-metadata.xml）中的entity_id值保持一致（一定要一致！）。如：http://os.cs.tsinghua.edu.cn/idp/shibboleth
+ **Metadata source**: IdP原数据的URL。如：在3.1.2中我们己经有了：https://raw.githubusercontent.com/jennyzhang8800/os_platform/master/idp-metadata.xml

把Ship email verification 和Visible勾上。（可选）  

其他项可以不用填

在右上角：Enabled选中

右下角：点Save保存。

 ![open-edx-sp-conf-14]()
  
 ![open-edx-sp-conf-15]()
    
保存后结果如下图所示：

 ![open-edx-sp-conf-16]()
  
 ![open-edx-sp-conf-17]()
 
    
### 3.3.4 检测是否配置成功
 
 (1). 在管理界面（http://cherry.os.cs.tsinghua.edu.cn/admin），进入到Third_Party_Auth下的Provider Configuration(SAML IdPs)界面。查看Metadata Ready是否是绿色的勾，如果是测说明能够正确获取IdP原数据。
 
  ![open-edx-sp-conf-18]()
 
 如果不是绿色的勾，先重启edx服务，然后刷新页面。 如果还不行，请检查Metadata source是否正确，通过这个URL是不是能获取idp原数据。

(2).检查是否能正确获取IdP端原数据的另一个方法，点击Third_Parth_Auth下的SAML Provider Data选项，会获取一次原数据，如果能成功获取，Is valid会打上绿色的勾。

如下图：

  ![open-edx-sp-conf-19]()
  
  结果如下：（Is valid打上绿色勾，说明能正确获取IdP端的原数据
  
    ![open-edx-sp-conf-20]()
    
 (3) 登录
 
在open edx首页，点sign in 

  
 ![open-edx-sp-conf-21]()

点击Use my Institution/Campus credentials

    
 ![open-edx-sp-conf-22]()
 
 点击Tsinghua_OS(你设置的名称)
 
  ![open-edx-sp-conf-23]()
  
  跳到IdP页面，输入用户名密码进行登录。（用户名密码是在open ldap保己经创建好用户的）
  
    ![open-edx-sp-conf-24]()
    
 如果是第一次登录，账号没有在edx注册过，那么会跳到下面的页面。以后登录会直接跳到edx页面，不会再出现下面的页面
 
登录成功，返回到edx页面。

  ![open-edx-sp-conf-25]()
