# open edx汉化.md

> 参考的是edustack.org给出的方法 [点此查看](http://edustack.org/2015/10/26/open-edx-cypress%E5%AE%8C%E6%95%B4%E6%B1%89%E5%8C%96%E8%AF%AD%E8%A8%80%E5%8C%85/)

## 1.切换至edxapp账户并加载环境变量

```
sudo -u edxapp bash
source /edx/app/edxapp/edxapp_env
```

## 2.删除现有语言包并上传新语言包
```
cd /edx/app/edxapp/edx-platform/conf/locale/zh_CN/LC_MESSAGES/
rm *
wget http://mirrors.edustack.org/LC_MESSAGES/django.po
wget http://mirrors.edustack.org/LC_MESSAGES/djangojs.po

```
## 3.执行翻译
```
cd /edx/app/edxapp/edx-platform
paver i18n_fastgenerate
```
## 4.退出edxapp账户并重启edxapp
```
exit
sudo /edx/bin/supervisorctl restart edxapp:
```
