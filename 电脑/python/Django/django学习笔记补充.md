<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 如何根据django的模型对象来获取其对应的表格的名字</a></li>
<li><a href="#orgheadline2">2. 如何使用好django的ImageField</a></li>
<li><a href="#orgheadline3">3. django的messages系统</a></li>
<li><a href="#orgheadline8">4. django项目的部署</a>
<ul>
<li><a href="#orgheadline4">4.1. apache serve 静态文件</a></li>
<li><a href="#orgheadline5">4.2. nginx serve 静态文件</a></li>
<li><a href="#orgheadline6">4.3. apache对接wsgi接口</a></li>
<li><a href="#orgheadline7">4.4. nginx对接wsgi接口</a></li>
</ul>
</li>
</ul>
</div>
</nav>


# 如何根据django的模型对象来获取其对应的表格的名字<a id="orgheadline1"></a>

参看 [这个网页](http://stackoverflow.com/questions/233045/how-to-read-the-database-table-name-of-a-model-instance) 。

答: 

    model_instance._meta.db_table

# 如何使用好django的ImageField<a id="orgheadline2"></a>

参考了 [这篇文章](http://gregblogs.com/django-saving-an-image-using-imagefield-explain-a-little/) 。

ImageField和FileField很类似，除了还多了 `width` 和 `height` 属性，然后就是在上传的时候确保文件是图片文件。

具体在模型文件中的定义如下:

    banner = models.ImageField(upload_to=game_2048_images, blank=True,
                               storage=OverwriteStorage(), default="placeholder.jpg")

上面的 `upload_to` 是控制图片在计算机中的保存路径，可以直接指定一个文件夹路径，但这通常不够灵活，这里通过一个函数来实现更加灵活的路径指定:

    def game_2048_images(instance, filename):
        """
        where image upload to.
        """
        return 'game/2048/images/{}/{}'.format(instance.user.username, filename)

这里具体路径是根据你在 `settings` 里面指定的 `MEDIA_ROOT` 而来，然后再指定里面的具体的文件夹路径。我们看到函数还可以接受具体模型对应的实例，从而建立自动根据user用户名来分配不同的文件夹路径。

`storage=OverwriteStorage()` 实现了如果文件名重复则覆盖的逻辑:

    class OverwriteStorage(FileSystemStorage):
        '''
        存储文件或图片，如果文件名重复则覆盖。
        '''
    
        def get_available_name(self, name):
            if self.exists(name):
                os.remove(os.path.join(settings.MEDIA_ROOT, name))
            return name

ImageField 可以和rest\_framework的序列化类形成很好的联动，最后序列化之后返回的是文件路径url字符串，测试的时候我们可以如下用django来挂载这些静态资源文件，实际运营的时候则推荐用nginx怎么设置一下url分发。

    from django.conf.urls.static import static
    from django.conf import settings
    
    if settings.DEBUG:
        urlpatterns += static('/data/', document_root=settings.MEDIA_ROOT)

在保存传过来的图片文件的时候，常规构建form对象也是可行的:

    form = Game2048InfoForm(
        request.POST, request.FILES, instance=target_info)
    
    if form.is_valid():
        new_game_info = form.save()
    else:
        logger.warning('form invalid')

否则你需要通过:

    request.FILES['imgfield']

这样的语法来获取图片内容。

# django的messages系统<a id="orgheadline3"></a>

本小节主要参看了 [这个网页](https://www.webforefront.com/django/setupdjangomessages.html) 。

使用django的messages系统，首先需要如下所示在settings里面进行一些配置：

    INSTALLED_APPS = [
        'django.contrib.messages',
        ......

    MIDDLEWARE_CLASSES = [
        'django.contrib.messages.middleware.MessageMiddleware',
        ......

    TEMPLATES = [
        {
             'OPTIONS': {
                'context_processors': [
                    'django.contrib.messages.context_processors.messages',
                    ......

然后在views那边，使用 `messages.add_message()` 来往django信息系统里面发送一个信息，此外还有如下的这些快捷方法：

-   messages.debug()
-   messages.info()
-   messages.success()
-   messages.warning()
-   messages.error()

你还可以如下设置每个request请求下的信息系统级别：

    from django.contrib import messages
    messages.set_level(request, messages.DEBUG)

下面我定义了一个简单的flash函数

```python
def flash(request, title, text, level='info'):
    """
    利用django的message系统发送一个信息，对接模板的sweetalert。
    """
    level_map = {
        'info': messages.INFO,
        'debug': messages.DEBUG,
        'success': messages.SUCCESS,
        'warning': messages.WARNING,
        'error': messages.ERROR
        }

    level = level_map[level]

    messages.add_message(request, level, text, extra_tags=title)
    return 'ok'
```

之所以做这样的封装是为了更好地对接sweetalert这个javascript库，然后模块加入如下内容从而从而实现信息的具体弹出行为。

    {% if messages %}
    <script src="{% static 'js/sweetalert.min.js' %}"></script>
    <script>
    {% for msg in messages %}
        sweetAlert({
            title: '{{msg.extra_args}}',
            text: '{{ msg.message }}',
            type: '{{ msg.level_tag }}',
          })
    {% endfor %}
    </script>
    {% endif %}

# django项目的部署<a id="orgheadline8"></a>

这里所谓的部署就是用apache或nginx这样的web服务器来对接django项目，说的再具体一点就把django作为一个wsgi程序服务起来。

django的静态文件和django项目的部署是密不可分的，严格讲起来django项目部署的时候其静态文件都是依赖于apache或者nginx这样的web服务器的。具体在这些工具的配置文档里面都有描述，大体是类似这样的配置：

## apache serve 静态文件<a id="orgheadline4"></a>

    <VirtualHost *:80>
        ......
      
        Alias /media/ /path/to/media/
        Alias /static/ /path/to/static/
      
        ......
    </VirtualHost>

## nginx serve 静态文件<a id="orgheadline5"></a>

    server {
        listen      80;
        ......
     
        client_max_body_size 75M;
     
        location /media  {
            alias /path/to/media;
        }
     
        location /static {
            alias /path/to/static;
        }
        
        ......
    }

然后所谓的 `python manage.py collectstatic` 命令就是为了把整个项目的静态文件都收集到一个文件夹里面来方便这些web服务器来serve 。

django的这个配置也是必要的：

    INSTALLED_APPS = [
        'django.contrib.staticfiles',
        ......

而我们谈论的修改url的那种做法只是在开发django项目的时候不想麻烦web服务器而这样做的修改。

    urlpatterns = [
      ......
    
    ] + static('/data/', document_root=settings.MEDIA_ROOT)

然后接下来就是web服务器要把django的wsgi接口对接好。

## apache对接wsgi接口<a id="orgheadline6"></a>

    <VirtualHost *:80>
        ......
      
        WSGIScriptAlias / /path/to/mysite.com/mysite/wsgi.py
        WSGIPythonPath /path/to/mysite.com
    
    </VirtualHost>

apache能这么直接对接wsgi接口是因为它装了mod\_wsgi模块

    # Python 2
    sudo apt-get install libapache2-mod-wsgi
     
    # Python 3
    sudo apt-get install libapache2-mod-wsgi-py3

apache服务django的wsgi.py文件还需要做一些修改：

    import os
    from os.path import join,dirname,abspath
     
    PROJECT_DIR = dirname(dirname(abspath(__file__)))
    import sys
    sys.path.insert(0,PROJECT_DIR) 
     
    os.environ["DJANGO_SETTINGS_MODULE"] = "blog.settings"
     
    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()

apache这边还有很多文件夹访问权限的控制配置，大体如下所示：

    <Directory /path/to/mysite.com/static>
    Require all granted
    </Directory>
    
    <Directory /path/to/mysite.com/media>
    Require all granted
    </Directory>
    
    WSGIScriptAlias / /path/to/mysite.com/mysite/wsgi.py
    
    <Directory /path/to/mysite.com/mysite>
    <Files wsgi.py>
    Require all granted
    </Files>
    </Directory>

更多细节请参看 [官方文档](https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/modwsgi/) 。

## nginx对接wsgi接口<a id="orgheadline7"></a>

nginx要对接wsgi接口需要uwsgi这个模块将wsgi接口服务起来。

    pip install uwsgi

更多细节请参看uwsgi的 [官方文档](http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html) 。

ngnix的配置如下：

    upstream django {
        server 127.0.0.1:8001; 
    }
    
    server {
        location / {
            uwsgi_pass  django;
            include     /path/to/your/mysite/uwsgi_params; 
        }
    }