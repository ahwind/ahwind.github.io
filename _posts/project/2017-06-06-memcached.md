---
layout:     post
title:      Memcached自助管理平台实现(后端架构)
category:   project
description: 通过平台可以释放运维工作，把部分权限放开给业务人员。
---

### 说明

- 通过平台可以释放运维工作，把部分权限放开给业务人员。
- 业务人员登录平台后自助选择集群和分配内存。

### 架构说明:

- 需要对saltstack简单改造
- 配置memcached的sls文件


### saltstack节点配置

修改配置文件master


```
ext_pillar:
  - saltapi:
      api: http://www.example.com/salt/pillar/
```
重启saltstack

这时agent请求过来时， 会读取API接口返回信息， 返回格式：


```
{"memc_items": {"11212": 33, "11211": 22, "11210":12}}
```

### 增加sls文件

代码片段：


![image](/images/memc/code1.jpg)

memcached.init

![image](/images/memc/code2.jpg)

![image](/images/memc/code3.jpg)

### 推送测试


```
sudo salt 'www.example.com' state.sls memcached.init
```


待续。。。