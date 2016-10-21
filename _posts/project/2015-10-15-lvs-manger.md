---
layout:     post
title:      LVS管理系统实现思路
category:   project
description: LVS管理系统实现思路， 开发语言： python  WEB框架：Django 数据库: YAML
---
### 说明

- 项目适用于LVS+keepalvied架构的服务器
- 项目没有开源， 一是因为即使开源也不可能100%适用， 另一个原因是这个项目比较老了， Django框架用的还是1.6版本，所以这里只给出了大概思路。
- 图片看不清楚的， 可以在新窗口打开图片查看

### 环境

- python 2.7

- Django 1.6

- saltstack

![image](/images/lvsmanage/index.jpg)
![image](/images/lvsmanage/datacenter.jpg)
![image](/images/lvsmanage/globalconfig.jpg)
![image](/images/lvsmanage/globalconfigedit.jpg)
![image](/images/lvsmanage/clustermanager.jpg)
![image](/images/lvsmanage/addcluster.jpg)
![image](/images/lvsmanage/clusterrole.jpg)
![image](/images/lvsmanage/remotepush.jpg)
![image](/images/lvsmanage/back.jpg)


会先进行对比，对比无误后推送到远程服务器


### YAML数据片段：



```
virtual_server:

  192.168.6.6 80:
    delay_loop: '60'
    desc: esf
    lb_algo: wlc
    lb_kind: DR
    port: '80'
    protocol: TCP
    real_server: {}
    sorry_server: 192.168.6.55
    
  192.168.6.2 80:
    delay_loop: '60'
    desc: esf
    lb_algo: wlc
    lb_kind: DR
    port: '80'
    protocol: TCP
    real_server:
      192.168.6.2:
        connect_timeout: '7'
        protocol: TCP_CHECK
        stat: true
        weight: 1
    sorry_server: 192.168.6.148
```



### YAML模版片段：


**cat init.sls**

```

/usr/local/keepalived/etc/keepalived.conf:
  file.managed:
    - source: salt://lvs/keepalived.conf.jinja
    - template: jinja
    - user: root
    - group: root
    - mode: 644
    
```


**cat keepalived.conf.jinja**


![image](/images/lvsmanage/template1.jpg)
![image](/images/lvsmanage/template2.jpg)