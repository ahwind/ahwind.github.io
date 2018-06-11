---
layout:     post
title:      salt-api-2015支持返回状态码
category:   article
description: 不升级支持返回命令执行返回状态码
---


salt 2015不支持返回状态码，升级又太麻烦， 还好可以通过修改源码实现我们的功能。

vim /usr/lib/python2.6/site-packages/salt/client/__init__.py


大概425行增加full_return=False,：


```
def cmd(
            self,
            tgt,
            fun,
            arg=(),
            timeout=None,
            expr_form='glob',
            ret='',
            jid='',
            full_return=False,
            kwarg=None,
            **kwargs):
            
            
```

大概546行， 修改：



```
                 
if fn_ret:
    for mid, data in fn_ret.items():
        if data.has_key('ret'):
            ret[mid] = (data if full_return
                 else data.get('ret', {}))                 
```

重启master

调用接口返回结果：



```
{"tt-op-test-05": {"retcode": 1, "ret": "Ok"}}
```



