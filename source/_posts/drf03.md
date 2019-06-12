---
title: Django-Rest-Framework-03-请求和响应
date: 2019-06-11 18:45:38
tags: Django-Rest-Framework
categories: Python
---

Django-Rest-Framework之请求和响应（request和response）

<!-- more -->

## 1. 请求对象
    request扩展的对象HttpRequest
    核心功能， request.data属性，和request.POST类似
## 2. 响应对象
    Response对象，按指定的congtent-type渲染数据，并返回给客户端
## 3. 状态码（status）
    100继续请求者应当继续提出请求。服务器已收到请求的一部分，正在等待其余部分。
    101切换协议请求者已要求服务器切换协议，服务器已确认并准备切换。
    200成功服务器已成功处理了请求。
    201已创建请求成功并且服务器创建了新的资源。
    202已接受服务器已接受请求，但尚未处理。
    203非授权信息服务器已成功处理了请求，但返回的信息可能来自另一来源。
    204无内容服务器成功处理了请求，但没有返回任何内容。
    205重置内容服务器成功处理了请求，内容被重置。
    206部分内容服务器成功处理了部分请求。
    300多种选择针对请求，服务器可执行多种操作。
    301永久移动请求的网页已永久移动到新位置，即永久重定向。
    302临时移动请求的网页暂时跳转到其他页面，即暂时重定向。（django redirect 采用）
    303查看其他位置如果原来的请求是 POST，重定向目标文档应该通过 GET 提取。(将post方法重定向为get方法)
    304未修改此次请求返回的网页未修改，继续使用上次的资源。使用缓存
    305使用代理请求者应该使用代理访问该网页。
    307临时重定向请求的资源临时从其他位置响应。
    400错误请求服务器无法解析该请求。
    401未授权请求没有进行身份验证或验证未通过。
    403禁止访问服务器拒绝此请求。
    404未找到服务器找不到请求的网页。
    405方法禁用服务器禁用了请求中指定的方法。
    406不接受无法使用请求的内容响应请求的网页。
    407需要代理授权请求者需要使用代理授权。
    408请求超时服务器请求超时。
    409冲突服务器在完成请求时发生冲突。
    410已删除请求的资源已永久删除。
    411需要有效长度服务器不接受不含有效内容长度标头字段的请求。
    412未满足前提条件服务器未满足请求者在请求中设置的其中一个前提条件。
    413请求实体过大请求实体过大，超出服务器的处理能力。
    414请求 URI 过长请求网址过长，服务器无法处理。
    415不支持类型请求的格式不受请求页面的支持。
    416请求范围不符页面无法提供请求的范围。
    417未满足期望值服务器未满足期望请求标头字段的要求。
    500服务器内部错误服务器遇到错误，无法完成请求。
    501未实现服务器不具备完成请求的功能。
    502错误网关服务器作为网关或代理，从上游服务器收到无效响应。
    503服务不可用服务器目前无法使用。
    504网关超时服务器作为网关或代理，但是没有及时从上游服务器收到请求。
    505HTTP 版本不支持服务器不支持请求中所用的 HTTP 协议版本。

## 4. 包装API视图
    两种方法：利用@api_view装饰器，装饰函数，另一种是基于类的APIView.
    装饰器提供了少量的功能，如确保接收Request，和添加内容到Response对象中，也提供了一些常见的异常响应对象，当接收的数据request.data出现错误时，如405 Method Not Allowed，
    
## 5. 使用实例
```python
# 利用qpp_view装饰器时，不再需要JSONResponse对象
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


# 单独对象，使用方法'GET', 'PUT', 'DELETE'（终于有用了！！！）
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a snippet instance.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
  
## 6. 为网址添加后缀（format参数）
    def snippet_list(request, format=None):
    def snippet_detail(request, pk, format=None):
    #更新urls.py
    urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
    ]
    # 设置后缀
    urlpatterns = format_suffix_patterns(urlpatterns)
    
    
    # 在网页端，我们利用Accept设置接收数据的格式
    http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
    http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
    # 或者在服务器端为网址添上后缀
    http http://127.0.0.1:8000/snippets.json  # JSON suffix
    http http://127.0.0.1:8000/snippets.api   # Browsable API suffix
    # 所以我们向浏览器返回的其实是可浏览的HTML表示
    
    # 网页端利用Content-Type请求头设置，来格式化向服务器请求的数据
    
## 7. 常用的method
    GET /zoos：列出所有动物园
    POST /zoos：新建一个动物园
    GET /zoos/ID：获取某个指定动物园的信息
    PUT /zoos/ID：更新某个指定动物园的信息（必须提供该动物园的全部信息）
    PATCH /zoos/ID：更新某个指定动物园的信息（只需提供该动物园的部分信息），所以一般更新使用patch
    DELETE /zoos/ID：删除某个动物园
    GET /zoos/ID/animals：列出某个指定动物园的所有动物
    DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
    