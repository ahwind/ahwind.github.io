---
layout:     post
title:      LVS管理系统实现思路
category:   project
description: LVS管理系统实现思路
---


- 项目适用于LVS+keepalvied架构的服务器
- 项目没有开源， 一是因为即使开源也不可能100%适用， 另一个原因是这个项目比较老了， Django框架用的还是1.6版本，所以这里只给出了大概思路。

![image](/images/lvsmanage/index.jpg)
![image](/images/lvsmanage/datacenter.jpg)
![image](/images/lvsmanage/globalconfig.jpg)
![image](/images/lvsmanage/globalconfigedit.jpg)
![image](/images/lvsmanage/clustermanager.jpg)
![image](/images/lvsmanage/addcluster.jpg)
![image](/images/lvsmanage/clustrole.jpg)
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


```
global_defs {
    notification_email {
{%- for mail in pillar['global_defs'] %}
        {{ mail }}
{%- endfor %}
    }
    notification_email_from {{ grains['nodename'] }}@example.com
    smtp_server mail.example.com
    smtp_connect_timeout 30
    router_id lvs_ha
}

vrrp_sync_group VG1 {
    group {
    {%- for group in pillar['vrrp_sync_group'] %}
        {{ group }}
    {%- endfor %}
    }
}

{% for  vi, vrrp in pillar['vrrp_instance'].iteritems() -%}
{% if vrrp != {} %}
vrrp_instance {{ vi }} {
    {% if vrrp['interface'] == 'lan' -%}
    interface {{ grains['lan_card'] }}
    {% elif vrrp['interface'] == 'wan' -%}
    interface {{ grains['wan_card'] }}
    {%- endif -%}
    state {{ pillar[grains['id']]['role'] }}
    virtual_router_id 51
    priority {{ pillar[grains['id']]['priority'] }}
    advert_int 3
    smtp_alert
    virtual_ipaddress {
        {%- for vip in vrrp['virtual_ipaddress'] %}
        {{ vip }}
{%- endfor %}
     }
    authentication {
        auth_type PASS
        auth_pass SfHa7
    }
}
{% endif %}
{% endfor %}
{% for key, list in pillar['virtual_server'].iteritems() %}
{%- if list['stat'] -%}
#{{ list['desc'] }}
virtual_server {{ key }} {
    delay_loop {{ list['delay_loop'] }}
    lb_algo {{ list['lb_algo'] }}
    lb_kind {{ list['lb_kind'] }}
    protocol {{ list['protocol'] }}
    {% for k, v in list['real_server'].iteritems() %}
    {%- if v['stat'] -%}
    real_server {{ k }} {{ list['port'] }} {
        weight {{ v['weight'] }}
        {{ v['protocol'] }} {
        {% if v['misc_path'] is defined -%}
            misc_path {{ v['misc_path'] }}
        {%- endif %}
        {% if v['misc_timeout'] is defined -%}
            misc_timeout {{ v['misc_timeout'] }}
        {%- endif -%}
        {% if v['connect_timeout'] is defined -%}
            connect_timeout {{ v['connect_timeout'] }}
        {%- endif %}
        }
    }
    {% endif %}
    {%- endfor %}
    {%- if list['sorry_server'] is defined -%}
    sorry_server {{ list['sorry_server'] }} {{ list['port'] }}
    {%- endif %}
}
{% endif %}
{% endfor %}
```
