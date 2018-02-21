title: django 给模板增加全局变量
date: 2017-11-14 21:54
tags:
- django
- 模板
---

用后台template渲染页面的时候，经常会遇到每个页面都会用到一些共通数据。比如登录的用户名，用户权限等等，如果每次都在view函数中每次都去处理这些变量，显得非常冗余。

想要在模板中预定义变量，只要利用template配置的context_processors，就可以轻易实现了~

<!-- more -->

首先编写一个函数，输入参数为request，返回值为dict类型

```python
def globar_var(request):
    return {
      'username': request.user.name,
      'role':  request.user.role,
      'menu': menu,
    }
```
假设把以上保存为my_context_processors.py，存储在站点下的common目录中。

然后在setting中的template字段加入以下配置
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            # insert your TEMPLATE_DIRS here
            os.path.join(BASE_DIR, 'templates'),
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                # Insert your TEMPLATE_CONTEXT_PROCESSORS here or use this
                # list if you haven't customized them:
                'django.contrib.auth.context_processors.auth',
                'django.template.context_processors.debug',
                'django.template.context_processors.i18n',
                'django.template.context_processors.media',
                'django.template.context_processors.static',
                'django.template.context_processors.tz',
                'django.contrib.messages.context_processors.messages',
                # 这是自定义的cp
                'common.my_context_processors.globar_var',
            ],
            'debug': True,
        },
    },
]
```
如果django版本低于1.8，则在setting.py里面定义TEMPLATES_CONTEXT_PROCESSORS 这个变量即可
```python
TEMPLATES_CONTEXT_PROCESSORS = [
  'common.my_context_processors.globar_var',
]
```

然后就可以直接在template中使用自定义的processors返回的内容了



