open edx在注册新账号时会向注册邮箱发送邮件，没有进行邮件服务器的配置会收不到邮件。

## 1.配置发件服务器地址和端口
```
sudo vi /edx/app/edxapp/cms.env.json
sudo vi /edx/app/edxapp/lms.env.json
```
在以上2个文件里设置
```
"EMAIL_HOST": "smtp.163.com",
"EMAIL_PORT": 25,
"EMAIL_USE_TLS": true,
"SITE_NAME": "cherry.cs.tsinghua.edu.cn", 
"DEFAULT_FROM_EMAIL":jennyzhang8800@163.com"

```

![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/lms-cms.env.json.png)

## 2.配置发件账号和密码
```
sudo vi /edx/app/edxapp/cms.auth.json
sudo vi /edx/app/edxapp/lms.auth.json
```
在以上2个文件里设置
```
"EMAIL_HOST_USER": "jennyzhang8800@163.com",
"EMAIL_HOST_PASSWORD":"*****",
```

## 3.确认无误，重启edxapp
```
sudo /edx/bin/supervisorctl  restart edxapp:
sudo /edx/bin/supervisorctl  restart edxapp_worker:
```

