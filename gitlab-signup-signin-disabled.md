# 去除gitlab首页的登录和注册功能，使其只能用shibboleth登录

## 1. 以管理员身份登录gitlab

  如果你没有改过密码，默认的管理员账号密码为：
  
     Username: root
     Password: 5iveL!fe 
     
## 2.进入admin area

    点击右上角admin area
    
## 3. 进入Settings页面

    点击左下角Settings
    
    在Sign-in Restriction区域

    取消Sign-up enabled和Sign-in enabled的勾选
      
    保存
 
![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-7.14.1-signup-disable.png)

## 4. 重新返回首页
 
 可以看到只能用shibbolth登录了
 
 ![picture](https://github.com/jennyzhang8800/os_platform/blob/master/pictures/gitlab-signin-disbale.png)
