---
layout:     post
title:      django管理后台添加另存为选项
category:   project
description: django管理后台添加另存为选项
---

Django版本：1.6

django后台添加数据时，可能会有这种情况：

新添加的数据和原来添加的数据大部分是重复的,我们没有必要重新添加存在的数据，

只要修改几个特定的内容就可以了。所以这里我们用到admin的另存为功能（默认是关闭的)

打开:


```
vim site-packages/django/contrib/admin/templatetags/admin_modify.py
```


找到:


```
'show_save_as_new': not is_popup and change and save_as,
```


添加注释：


```
#'show_save_as_new': not is_popup and change and save_as,
```


在下面增加：


```
'show_save_as_new': context['has_add_permission'] and
                           not is_popup and (not save_as or context['add']),
```


保存
在打开后台页面，如下：


![image](/images/django/saveas.jpg)