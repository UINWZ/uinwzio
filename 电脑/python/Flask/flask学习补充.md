<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. path模糊处理结尾的slash</a></li>
<li><a href="#orgheadline2">2. jinja2模板渲染意外有很多乱码</a></li>
</ul>
</div>
</nav>


# path模糊处理结尾的slash<a id="orgheadline1"></a>

参看 [这个网页](http://stackoverflow.com/questions/33241050/trailing-slash-triggers-404-in-flask-path-rule) 。

```python
@app.route('/users', strict_slashes=False)
@app.route('/users/&lt;path:path&gt;')
def users(path=None):
    return str(path)

c = app.test_client()
print(c.get('/users'))  # 200 OK
print(c.get('/users/'))  # 200 OK
print(c.get('/users/test'))  # 200 OK
```

# jinja2模板渲染意外有很多乱码<a id="orgheadline2"></a>

这个前面讲过flask里面的jinja2默认全局auto escape的，也就是这几个符号:

    <  >  &  "

会转码。这单个字符大家都是清楚的，而且我们也知道要避免就是如下加入 `safe` 过滤器:

    {{ "<b>test</b>"|safe }}

之所以这里再提出来不是罗嗦，而是强调这个事实。因为有的时候在程序中字符串经过某些处理，中文字符等到网页中是 `&#22899;` 这样的东西，你如果弄个html文档，然后在里面添加这个东西，你会发现网页会正常显示一个中文字 "女" 字。但是如果不加上safe过滤器，则 `&` 会变成 `&amp;` ，也就是实际显示的是 `&` 符号，然后之前的 `&#22899;` 这样的东西就不直接这样显示。这里强调就是为了，因为python程序经常遇到字符编码问题，所以遇到这个问题的时候，我第一反应也是python那么encoding是不是出问题了，怎么是一些乱码。甚至会想到是不是SQL数据库转换的问题等等。实际上那些都不是乱码，在网页中那么写就是可以正常显示的，这里强调一下。