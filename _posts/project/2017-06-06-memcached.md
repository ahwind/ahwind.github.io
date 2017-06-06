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


```
memcached:
  pkg:
    - name: memcached
    - installed
  user.present:
    - name: memc
    - uid: 1501
    - createhome: False
    - gid_from_name: True
    - shell: /sbin/nologin


{% for port, size in pillar['memc_items'].iteritems() %}
memcached_{{ port }}:
  file.managed:
    - name: /etc/init.d/memcached_{{ port }}
    - source: salt://memcached/memcached.init
    - template: jinja
    - user: root
    - group: root
    - mode: 755
    - require:
      - pkg: memcached
    - defaults:
      port: {{ port }}
      size: {{ size }}

  service.running:
    - name: memcached_{{ port }}
    - enable: True
    - restart: True
    - watch:
      - file: /etc/init.d/memcached_{{ port }}
```

memcached.init


```
#! /bin/sh
#
# chkconfig:    2345 58 02
# description:  The memcached daemon is a network memory cache service.
# processname: memcached

# Source function library.
. /etc/init.d/functions
PORT={{ port }}
USER=nobody
MAXCONN=30240
CACHESIZE={{ size }}
```


```
start () {
        echo -n $"Starting $prog: "
        # insure that /var/run/memcached has proper permissions
    if [ ! -d /var/run/memcached ]; then
        mkdir -p /var/run/memcached
    fi

    if [ "`stat -c %U /var/run/memcached`" != "$USER" ]; then
        chown $USER /var/run/memcached
    fi

        $prog -v -d -p $PORT -u $USER  -m $CACHESIZE -c $MAXCONN  -P $pidfile
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && touch /var/lock/subsys/memcached_$PORT
}
```

### 推送测试


```
sudo salt 'www.example.com' state.sls memcached.init
```
