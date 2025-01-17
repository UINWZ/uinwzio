<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 前言</a></li>
<li><a href="#orgheadline2">2. api接口</a></li>
<li><a href="#orgheadline3">3. api类初始化过程</a></li>
<li><a href="#orgheadline4">4. 动作阶段</a></li>
<li><a href="#orgheadline5">5. 内部小缓存</a></li>
</ul>
</div>
</nav>


# 前言<a id="orgheadline1"></a>

客户端的api对接设计很少为大家谈论，人们更多是谈论服务器那边的restful风格之类的，但实际上因为客户端的api需要对接多个服务器的api，并且承担着入口数据的初步规整功能，客户端的api设计是很值得我们花心思去思考的。

不管对面的服务器那边的api是何种用途，资源何种格式输入进来，到了客户端这边，大体要分为这几个步骤:

1.  api接口
2.  资源过滤从而获得目标item的id
3.  根据目标item的id调用某种动作来获取目标item的某种属性
4.  反复调用的内部简单缓存机制

下面我们就这几个步骤详细讨论之。

# api接口<a id="orgheadline2"></a>

api接口在和服务器对接的过程中，应该保证开放出来的客户端接口参数和服务器那边api接口参数具有一致性。然后api接口部分应该自动处理好认证过程。大体如下风格:

```python
def authenticate(func):
    @wraps(func)
    def _authenticate(self,*args,**kwargs):
        params = inspect.getcallargs(func,*args,**kwargs)
        params.update({'appkey':self.appkey})
        params = OrderedDict(sorted(params.items(), key=lambda t: t[0]))
        string = urlencode(params)
        sign = hashlib.md5((string+self.secretkey).encode('utf-8')).hexdigest()
        params.update({'sign':sign})
        logging.debug('sending params: {}'.format(params))

        if func.__name__ == 'download':
            endpoint = 'http://interface.bilibili.com/playurl'
            import xmltodict
            response = requests.get(endpoint,params=params,
                headers=self.headers)
            xml = response.text
            return xmltodict.parse(xml)
        else:
            endpoint = 'http://api.bilibili.cn/{}'.format(func.__name__)
            jsond = wget_json(endpoint,params=params,headers=self.headers)
            return jsond
    return _authenticate

@authenticate
def search(keyword,pagesize=20,order='default',page=1):
    '''
    order：排序方式  默认default
        - default 综合排序
        - pubdate 发布日期倒序
        - senddate 修改日期倒序
        - id 投稿id倒序
        - ranklevel 相关度倒序
        - click 点击从高到底
        - scores 评论数
        - damku 弹幕数
        - stow 收藏数
    keyword：关键词
    pagesize:返回条目多少
    page：页码

    返回
    type mylist我的列表 video视频 special专题
    lid 我的列表编号（on mylist）
    aid 视频编号（on video）
    spid 专题编号（on special）
    author 视频作者
    play 播放次数
    review 评论数
    video_review 弹幕数
    favorites 收藏数
    title 标题
    description 描述
    tag 标签
    pic 封面图片地址

    '''
```

这里api在调用search方法的时候，接受的参数和对面目标服务器 `/api/search/` 下的参数具有一致性，并且通过这个装饰器自动完成了认证过程。

# api类初始化过程<a id="orgheadline3"></a>

api类在初始化过程也就是初始化函数 `__init__` 那里，就能够完成大部分的对于目标item的搜索定位操作。

```python
def __init__(self, url=''):
    self.page = 0 # 默认页码为0
    self.url = url
    if self.url:
        self.page = self.get_pagenum(self.url)
        self.aid = self.get_aid(self.url)
        self.cid = self.get_cid(self.url)

def set_url(self,url):
    '''changing the url you are working on.'''
    self.url = url
    self.page = self.get_pagenum(self.url)
    self.aid = self.get_aid(self.url)
    self.cid = self.get_cid(self.url)
```

这整个初始化过程除了具体实例化本api的特定变量环境外，就是根据这些变量环境的定义了目标item。然后的很多属性等等大多都根据这些identifier来获取目标信息。

# 动作阶段<a id="orgheadline4"></a>

根据identifier获取的目标item之后，然后就可以针对这个目标item，来设计各种输出动作，这里最常见的是属性风格:

```python
@property
def download_urls(self):
    if self.cid:
        urls = self.get_download_urls(self.cid)
        return urls
    else:
        logging.warning('you must set the target url')
```

api类另外还定义了一些支持动作来支持这些属性输出动作。

我们更常见的是获取属性，但是其他更改属性的动作等等也在这里属于动作阶段。

# 内部小缓存<a id="orgheadline5"></a>

这个是我们不可回避的问题，而且不需要杀鸡用牛刀的采用广域的那种缓冲机制，而应该在本api内部就建立一种记忆缓存机制。同一api获取某个目标item的多个属性的操作可能多次进行，而每一次都网络请求一次显然是效率极其低下的，我们必须建立起一种内部的小缓存机制。

缓存过程从理论上就是要生成一个具有唯一性标识的key，然后将输出 `@property` 那里的值存入，通过函数装饰器修改其行为即可。

下面的代码参考了 [这里的代码](https://github.com/mcs07/PubChemPy/blob/master/pubchempy.py) 。具体item的唯一性标识试情况而定，下面就是最简单的当前工作url即可，复杂点的情况 `gen_id` 需要好好设计一下。我们缓存都放入 `self.memory` 中。

```python
def gen_id(self)
    return self.url

def memoized_property(fget):
    """Decorator to create memoized properties.
    """
    @wraps(fget)
    def fget_memoized(self):
        key = '{}{}'.format(self.gen_id(),fget.__name__)
        if key not in self.memory:
            self.memory[key] = fget(self)

        return self.memory[key]
    return property(fget_memoized)
```