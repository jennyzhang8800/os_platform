# 备份文件

> 该备份是对所有修改过的配置文件进行备份，备份的是修改后的结果

此备份包含的内容有：

 + edx-backup: 这是open edx作为SP端的备份。此处只涉及修改过的配置文件的备份。
 + openLDAP-backup：这是openLDAP的备份。包含创建用户的脚本create_user.ldif和创建eduperson的脚本eduperson.ldif
 + gitlab-backup: 这是gitlab作为SP端的备份。此处只备份了修改过的文件，该文件的实际路径为/etc/gitlab和 /etc/apache2 和/etc/shibboleth
 + shibboleth-idp: 这是IdP配置文件的备份。同样只备份修改过的文件，实际路径为/etc 和 /opt/shibboleth-idp
