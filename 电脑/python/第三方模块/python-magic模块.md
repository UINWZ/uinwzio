<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 简介</a></li>
<li><a href="#orgheadline3">2. 图片有效性检验</a>
<ul>
<li><a href="#orgheadline2">2.1. 使用imghdr模块</a></li>
</ul>
</li>
</ul>
</div>
</nav>


# 简介<a id="orgheadline1"></a>

python-magic模块的github地址在 [这里](https://github.com/ahupp/python-magic) 。其可以看作linux下file命令的模拟（不过这个模块在windows和OS X系统下似乎也有支持）。有兴趣的可以先看看linux下file命令的使用效果。

    file test.pdf

    >>>file test.pdf
    test.pdf: PDF document, version 1.2

这个模块也没什么官方文档，不过好在使用还是很简单的。如下所示:

    >>> import magic
    >>> magic.from_file("testdata/test.pdf")
    'PDF document, version 1.2'
    >>> magic.from_buffer(open("testdata/test.pdf").read(1024))
    'PDF document, version 1.2'
    >>> magic.from_file("testdata/test.pdf", mime=True)
    'application/pdf'

# 图片有效性检验<a id="orgheadline3"></a>

之前用pillow处理一张图片后来出错了，很是纠结，然后发现原来这张图片格式有误，用本地的所有图片查看器包括浏览器都打不开。刚开始想法是用pillow试探性open来检验图片有效性。后来发现原来file命令是可以发现这些图片格式有误的。如下所示:

    >>> magic.from_file("test_ok.jpg", mime=True)
    'image/jpeg'
    >>> magic.from_file("test_bad.jpg", mime=True)
    'application/octet-stream'
    >>> magic.from_file("test_png.jpg", mime=True)
    'image/png'
    >>> magic.from_file("test_jpg.png", mime=True)
    'image/jpeg'

这里分为几种情况:

1.  图片类型是jpg，后缀可能是jpg或者JPG或者jpeg之类的，在使用上没有什么麻烦，但可以考虑后缀名修复。
2.  图片完全格式错误，也就是不是image/what，这种情况下这个图片是完全不能用的。
3.  图片后缀是jpg，实际图片是png格式；或者图片后缀是png，实际图片是jpg格式。这在使用上可能某些浏览器能够正常打开，可能不行，可能某些图片查看器可以正常打开，也可能会出现问题。这些情况推荐统一和情况一一样进行图片后缀名修复。

## 使用imghdr模块<a id="orgheadline2"></a>

当然就作为图片有效性检验这个问题，可能使用python-magic模块有点杀鸡用牛刀了。python官方内置模块imghdr已经能够很好地胜任这个工作了。

    >>> import imghdr
    >>> imghdr.what("test_bad.jpg")
    >>> bool(imghdr.what("test_bad.jpg"))
    False
    >>> imghdr.what("test_ok.jpg")
    'jpeg'
    >>> imghdr.what("test_jpg.png")
    'jpeg'
    >>> bool(imghdr.what("test_bad.jpg"))
    False
    >>> imghdr.what("test_png.jpg")
    'png'

如果imghdr不能识别该图片类型:

    'rgb'   SGI ImgLib Files
    'gif'   GIF 87a and 89a Files
    'pbm'   Portable Bitmap Files
    'pgm'   Portable Graymap Files
    'ppm'   Portable Pixmap Files
    'tiff'  TIFF Files
    'rast'  Sun Raster Files
    'xbm'   X Bitmap Files
    'jpeg'  JPEG data in JFIF or Exif formats
    'bmp'   BMP files
    'png'   Portable Network Graphics

其将返回False——可视为不认为其为有效图片。这大部分时候应该已经能够满足你的需要了，但如果你需要处理svg图片:

    >>> imghdr.what("test02.svg")
    >>> import magic
    >>> magic.from_file("test02.svg", mime=True)
    'image/svg+xml'

那么imghdr是不支持的，而magic模块的分类更好一点。如果用imghdr模块能够解决你的问题，就用imghdr模块吧，如果不能满足你的要求那么再考虑使用magic模块。