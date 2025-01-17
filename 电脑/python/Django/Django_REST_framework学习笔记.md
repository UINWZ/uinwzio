<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 前言</a></li>
<li><a href="#orgheadline5">2. 理解序列化的类</a>
<ul>
<li><a href="#orgheadline2">2.1. 多个查询集的操作</a></li>
<li><a href="#orgheadline3">2.2. ModelSerializer</a></li>
<li><a href="#orgheadline4">2.3. 初步实现API接口</a></li>
</ul>
</li>
<li><a href="#orgheadline6">3. API权限管理</a></li>
<li><a href="#orgheadline7">4. 参考资料</a></li>
</ul>
</div>
</nav>


# 前言<a id="orgheadline1"></a>

    pip install djangorestframework

然后在目标django项目上的 `settings` 那边加上

    INSTALLED_APPS = (
        ...
        'rest_framework',
    )

# 理解序列化的类<a id="orgheadline5"></a>

django rest framework的序列化类类似于django的表单类，不同的是django的表单类是用于沟通django的Model和网页form之间的桥梁；而序列化类是用于沟通django的Model类和JSON数据格式之间的桥梁。

    注: Model -> Serializer （其data挂载的是python的dict字典值了）
    
    serializer = SnippetSerializer(snippet)
    serializer.data
    # {'pk': 2, 'title': u'', 'code': u'print "hello, world"\n', 'linenos': False, 'language': u'python', 'style': u'friendly'}

注意 `created` 这个字段的值不在了，这是因为上面定义的SnippetSerializer自动忽略它了，然后就是 DateTime object不太好JSON格式化。一般日期时间类如果不需要的话将自动忽略，如果确实有需要则再考虑另外的处理，简单的处理方案就是按照格式将日期时间字符串化，然后再按照这种格式反解析即可。这个视具体需求来的。

然后将这个字典值JSON化，大体类似于json.dumps的意思吧:

    >>> content = JSONRenderer().render(serializer.data)
    >>> content
    b'{"pk":2,"title":"","code":"print(\\"hello world!\\"\\n)","linenos":false,"language":"python","style":"friendly"}'

这个就可以直接送入api进行传输了。然后我比较了其和json.dumps的区别，大体是这个过程:

    json.dumps(serializer.data).encode('utf-8')

不过具体其字节流并不完全一致，但数据是无误的，我们后面会看到。

接下来就是将这个字节流变成python的字典值，也就是json.loads的功能，大体是:

    json.loads(content.decode('utf-8'))

教程里给出的一个方法是:

    from django.utils.six import BytesIO
    
    stream = BytesIO(content)
    data = JSONParser().parse(stream)

不清楚这个JSONParser有何过人之处，后来我比较了下:

    >>> json.loads(content.decode('utf-8')) == serializer.data
    True

可以看到这个字节流反序列为python对象的过程是没有错误的。

然后下面就是将该python对象变成serializer对象，如下所示:

    serializer = SnippetSerializer(data=data)

用的还是上面定义的那个SnippetSerializer类，然后存储数据库是:

    serializer.save()

## 多个查询集的操作<a id="orgheadline2"></a>

    serializer = SnippetSerializer(Snippet.objects.all(), many=True)
    serializer.data

## ModelSerializer<a id="orgheadline3"></a>

类似于django的表单类，可以利用 `ModelSerializer` 类来更快地创建序列化类。

```python
from rest_framework import serializers

class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

为了证明这种简便写法对于上面的任务（包括create和update方法都会自动实现的）是完全应付得过来的。上面在shell里的代码再撸一遍。

    >>> from snippets.models import Snippet
    >>> snippet = Snippet(code='print "hello, world2"\n')
    >>> from snippets.serializers import SnippetSerializer
    >>> snippet.save()
    >>> serializer = SnippetSerializer(snippet)
    >>> serializer.data
    {'language': 'python', 'linenos': False, 'id': 3, 'title': '', 'code': 'print "hello, world2"\n', 'style': 'friendly'}
    >>>

比较之后还是发现有些区别的，字典顺序都随便，主要是这里的 'id' ，在之前是改成了 'pk' 的。

## 初步实现API接口<a id="orgheadline4"></a>

这里直接引入基于类的API接口编写风格。具体代码如下:

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
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
```

这里的 `request.data` 大体类似于 `request.POST` ，但更加通用，对于PUT等方法也适用。

大概是这个意思恩。然后用 `rest_framework` 提供的 `Response` 类封装一下返回内容使其成为json友好的格式。

然后 `urls.py` 如下所示，django项目相关urls.py配置这里就不赘述了。

```python
from django.conf.urls import url
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
]
```

这样我们就有了如下所示的API接口:

-   这是新插入一个记录

    bash>>> http POST http://127.0.0.1:8000/snippets/ code="new print"

-   这是显示所有的记录

    bash>>> http http://127.0.0.1:8000/snippets/

这里的 `is_valid` 使用来检测某个序列化类存入数据库是否会有错误，如果有错误则 `is_valid()` 返回的是False，然后错误存入了 `serializer.errors`

然后就是各个状态码的选中，要如上面例子所示的这样做，这样更加专业和规范。

然后参考资料1那边继续给出的所谓的restful-api的针对某个特定item的操作:

```python
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
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
```

urls.py那边加上的是:

    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),

这里正则表达式 `(?P<pk>[0-9]+)` 的意思是收集某一串数字，这一串数字被命名为 `pk` 。

然后还有什么后缀控制，还有什么minin类后面再说吧。

# API权限管理<a id="orgheadline6"></a>

# 参考资料<a id="orgheadline7"></a>

1.  [django rest framework教程中文版](https://whatwewant.gitbooks.io/django-rest-framework-tutorial-cn/content/1.Serialization.html)