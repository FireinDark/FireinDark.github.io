---
title: Django-Rest-Framework-04-基于类视图的视图函数
date: 2019-06-11 18:52:23
tags: Django-Rest-Framework
categories: Python
---

Django-Rest-Framework之基于类视图的视图函数

<!-- more -->

## 1. 基于类的视图函数（views.py)
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
# APIView 
from rest_framework.views import APIView
# 返回对象
from rest_framework.response import Response
# 状态码
from rest_framework import status

# 多个对象
class SnippetList(APIView):
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)
    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
        
# 单个对象的部分修改（APIView提供了一些request和response对象）
class SnippetDetail(APIView):
    """
    检索，更新或删除一个实例
    """
    # 首先获取对象， 发生异常就抛出404页面
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

# 重构urls.py
urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
]
# 后缀
urlpatterns = format_suffix_patterns(urlpatterns)
```

## 2. 基于Mixin类重构视图类
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    # 雷打不动，必须的
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```
-  CreateModelMixin: 定义了创建一个序列对象的方法create(self, request, *args, **kwargs)，保存方法perform_create(self, serializer)，成功获取请求头的方法：get_success_headers(self, data)
-  ListModelMixin： 定义了一个获取查询集的方法, many=True：list(self, request, *args, **kwargs)
-  RetrieveModelMixin: 定义了一个检索方法，retrieve(self, request, *args, **kwargs)
-  UpdateModelMixin： 更新一个模型实例，update(self, request, *args, **kwargs)
-  DestroyModelMixin： 删除一个模型实例，方法destroy(self, request, *args, **kwargs)

```
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics
# 重构单个实例处理方法
class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

## 3.复合功能的View类
1. GenericAPIView：基本的View类，定义了一些常用的方法
- get_queryset： 获取所有查询集，返回序列器中指定queryset模型的全部对象
- get_object： 根据传入的查询参数（lookup_url_kwarg or lookup_field)获取查询对象，然后返回， 一般进行联合查询时， 需要重写此方法
- get_serializer: 获取序列化实例，传入的参数需要通过验证
- get_serializer_context： 返回序列化的类，api视图中看到的那一大串
- paginate_queryset： 进行分页，返回分页后的单页结果集
- get_paginated_response：返回分也好的结果集到前台
2. CreateAPIView
3. ListAPIView
4. RetrieveAPIView
5. DestroyAPIView
6. UpdateAPIView
7. ListCreateAPIView
8. RetrieveUpdateAPIView
9. RetrieveDestroyAPIView
10. RetrieveUpdateDestroyAPIView