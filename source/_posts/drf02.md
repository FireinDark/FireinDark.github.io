---
title: Django-Rest-Framework-02-Rest接口新版写法
date: 2019-06-11 18:30:43
tags: Django-Rest-Framework
categories: Python
---

Rest接口新版写法

<!-- more -->

## REST框架简介
1. 身份验证策略OAuth1a和OAuth2的包
2. 支持ORM和非ORM数据源的序列化
3. 可以自定义，基于功能的常规视图

## 简单的接口写法  

1. 设置settings  

```
    INSTALLED_APPS = (
         ...
         'rest_framework',
    )
    
    # 配置我们框架
    REST_FRAMEWORK = {
    # 设置权限，即谁可以访问，如认证过的，匿名的等
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]，
    # 每页显示的个数
     'PAGE_SIZE': 10
    }
```

2. REST框架的登录和注销视图,设置urls.py  

```
    from rest_framework import routers
    # 建立一个路由器对象
    router = routers.DefaultRouter()
    # 将我们的路由注册到url里， /users/
    router.register(r'users', UserViewSet)
    urlpatterns = [
        url(r'^', include(router.urls)),
        url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
    ]
    # URL路径可以是任何你想要的，但你必须包括'rest_framework.urls'与'rest_framework'命名空间, 1.9以前
```  

3. 在app中添加我们的序列器，serializer.py  

```
    from django.conf.urls import url, include
    from django.contrib.auth.models import User
    from rest_framework import routers, serializers, viewsets

    # Serializers define the API representation.
        class UserSerializer(serializers.HyperlinkedModelSerializer):
            class Meta:
                model = User
                # 定义了我们序列化的模型和显示的字段， url为网络接口
                fields = ('url', 'username', 'email', 'is_staff')
                

    class UserViewSet(viewsets.ModelViewSet):
        # 查询对象集
        queryset = User.objects.all()
        # 序列化的类名
        serializer_class = UserSerializer
        # 固定写法！不能改名字！
```