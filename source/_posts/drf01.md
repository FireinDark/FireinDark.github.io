---
title: Django-Rest-Framework-01-快速入门
date: 2019-06-11 18:34:26
tags: Django-Rest-Framework
categories: Python
---

Django-Rest-Framework快速入门

<!-- more -->

## 1. 创建模型(models.py)
from django.db import models
# 高亮文本， 两种风格
    from pygments.lexers import get_all_lexers
    from pygments.styles import get_all_styles

    LEXERS = [item for item in get_all_lexers() if item[1]]
    LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
    STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


    class Snippet(models.Model):
        created = models.DateTimeField(auto_now_add=True)
        title = models.CharField(max_length=100, blank=True, default='')
        code = models.TextField()
        linenos = models.BooleanField(default=False)
        language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
        style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

        class Meta:
            ordering = ('created',)

## 2. 创建自定义的Serializer类（serializers.py）
```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    # read_only 只读
    id = serializers.IntegerField(read_only=True)
    # required：必要， allow_blank:允许为空，
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    # style: 设置显示样式，控制如何显示可浏览的API
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')
    # 一般是与model一一对应的，一般可以少，但不可以多，如果想增加需要另外进行处理！
    # 字段设置是为了验证前端通过get等方法传递过来的数据
    def create(self, validated_data):
        """
        根据传入的数据创建一个实例
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        更新一个实例，若没有传值，就使用原来的数据
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

## 3. 序列化与反序列化
    from rest_framework.renderers import JSONRenderer
    from rest_framework.parsers import JSONParser
    # 序列化，将数据转化成json格式
    serializer = SnippetSerializer(snippet)
    content = JSONRenderer().render(serializer.data)
    # 反序列化，转换成本机类型
    from django.utils.six import BytesIO
    stream = BytesIO(content)
    # 首先将数据转化成二进制流，然后再将其渲染成python中的字典
    data = JSONParser().parse(stream)
    
## 4. 简便的ModelSerializer，要求不高的数据，不用自定义
```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
    # 默认实现了create()和 update()方法
```


## 5.重点，编写我们的视图处理函数（views.py）
```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

# 不用进行csrf验证
@csrf_exempt
def snippet_list(request):
    # 删除多个对象的方法
    if request.method == 'GET':
        # 获取所有的对象
        snippets = Snippet.objects.all()
        # 序列化所有的查询集，必须设置many=True，模型对象不用设置
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        # 验证我们的数据
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
        
# 处理单个对象的方法（get, put, delete）
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        # 序列化数据
        serializer = SnippetSerializer(snippet)
        # 将数据转换成json格式，作为响应返回给客户端
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        # 将请求对象反序列化为python对象
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)        
        
```




## 6. serializer支持的字段
    models.AutoField: IntegerField,
    models.BigIntegerField: IntegerField,
    models.BooleanField: BooleanField,
    models.CharField: CharField,
    models.CommaSeparatedIntegerField: CharField,
    models.DateField: DateField,
    models.DateTimeField: DateTimeField,
    models.DecimalField: DecimalField,
    models.EmailField: EmailField,
    models.Field: ModelField,
    models.FileField: FileField,
    models.FloatField: FloatField,
    models.ImageField: ImageField,
    models.IntegerField: IntegerField,
    models.NullBooleanField: NullBooleanField,
    models.PositiveIntegerField: IntegerField,
    models.PositiveSmallIntegerField: IntegerField,
    models.SlugField: SlugField,
    models.SmallIntegerField: IntegerField,
    models.TextField: CharField,
    models.TimeField: TimeField,
    models.URLField: URLField,
    models.GenericIPAddressField: IPAddressField,
    models.FilePathField: FilePathField,

## 7. 自定义字段，支持的参数
    LIST_SERIALIZER_KWARGS = (
    'read_only', 'write_only', 'required', 'default', 'initial', 'source',
    'label', 'help_text', 'style', 'error_messages', 'allow_empty',
    'instance', 'data', 'partial', 'context', 'allow_null')