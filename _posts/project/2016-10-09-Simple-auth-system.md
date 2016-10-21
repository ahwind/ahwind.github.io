---
layout:     post
title:      简单权限认证管理系统
category:   project
description: 实现简单认证系统， 密码强度不高。对保密性要求比较高的系统不适用。开发语言:LUA  基于nginx基本认证和NGINX_LUA模块
---


## 这套程序主要用于静态页的简单权限管理

- 实现用户名登录和基于URL的权限认证
- 登录系统基于nginx的HTTP Basic Authentication和nginx_lua模块
效果图：

![image](/images/sams/sams1.jpg)

本地记录信息


```
Authorization:Basic bxdjcmdmZQ5nan6xOnVzDXJbbeddbcb=
```



用户配置文件：

```
#auth后面保存一级目录, 比如允许访问http://www.example.com/docs/， 后面加入docs即可。

{
    "zhangsan": {
        "auth":"*",
        "group": "admin"
    },
    "lisi": {
        "auth":"docs  work  ",
        "group": "dev"
    }
}
```



组配置文件：


```
{
    "admin": "*",
    "dev": "dev"

}
```


非允许访问会跳转到403页面


[项目地址](https://github.com/ahwind/sams)
