---
title: Django-Rest-Framework-05-用户认证和权限管理
date: 2019-06-12 10:28:38
tags: Django-Rest-Framework
categories: Python
---


Django-Rest-Framework之用户认证和权限管理

<!-- more -->

## 1. 目的
- 实例对象始终与创建者相关联
- 只有通过身份验证的用户可以创建实例
- 只有对象的创建者能够更新和删除该对象
- 未认证用户应该只有只读权限

## 2. 数据库中字段的on update 和 on delete参数
- 数据库外键定义的可选项，用来设置当主键表中的被参考列的数据发生变化时，外键表中响应字段的变化规则。update则是主键表中被参考字段的值更新，delete是指在主键表中删除一条记录
- on_update 和 on_delete的值有四个：DO_NOTHING  ， SET_NULL ，  SET_DEFAULT  ，CASCADE，PROTECT
- no action 表示不做任何操作
- set null 表示在外件表中将相应字段设置为null
- ser default 表示设置默认值
- cascade 表示级联操作，如果主键表中被参考字段更新，外键表中也更新，主键表中的记录被删除，外键表中该行也相应删除

## 3. 删除原数据库和迁移记录
    rm -f tmp.db db.sqlite3
    rm -r snippets/migrations

## 4.添加用户控制入口
- 首先在模型文件中设置关联外键
    
```
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
    highlighted = models.TextField()
```

- 设置文本高亮，重写保存方法

```
 def save(self, *args, **kwargs):
        """
        利用pygments，创建高亮的html文本呈现code块
        """
        lexer = get_lexer_by_name(self.language)
        linenos = self.linenos and 'table' or False
        options = self.title and {'title': self.title} or {}
        formatter = HtmlFormatter(style=self.style, linenos=linenos,
                                  full=True, **options)
        self.highlighted = highlight(self.code, lexer, formatter)
        super(Snippet, self).save(*args, **kwargs)
```

## 5. 设置断点
1. 在serializers.py中添加UserSerializer

```
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
        # snippets为反向引用，不会被默认包含，所以需要添加显示字段
```
2. 在views.py中使用只读属性

```
from django.contrib.auth.models import User
from snippets.serializers import UserSerializer

class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```
3. 添加url

```
url(r'^users/$', views.UserList.as_view()),
url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
```
4. 连接Snippets和Users
```
# 通过重写perform_create()方法
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```
## 6. 更新serializer
添加一行，另外在meta类中也要添加'owner'字段

```
# 只被用于序列化呈现，而不会被用于更新模型实例
owner = serializers.ReadOnlyField(source='owner.username')
```

## 7.为我们的视图函数添加权限认证
- IsAuthenticatedOrReadOnly： 确保认证用户拥有读写权限，匿名用户只读

```
from rest_framework import permissions
# 在SnippetList and SnippetDetail视图函数类中添加
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

## 8. 添加登录窗口url,rest_framework自带

```
urlpatterns += [
    # 前面正则表达式，可以随便怎么写，但是namespace,必须要是'rest_framework'
    url(r'^api-auth/', include('rest_framework.urls',
                               namespace='rest_framework')),
]
```

## 9. 用户权限内等级管理
- 在app中创建permissions.py
```
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    设置权限只允许创建者编辑
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        # 为不同的请求设置权限，GET, HEAD or OPTIONS 为安全请求
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the snippet.
        # 写权限只有代码拥有者有，判断拥有者和请求者是否是同一个用户
        return obj.owner == request.user
```
- 然后在SnippetDetail视图类中，设置权限类

```
from snippets.permissions import IsOwnerOrReadOnly
permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                      IsOwnerOrReadOnly,)
```

## 10. 用户认证
（SessionAuthentication and BasicAuthentication）
