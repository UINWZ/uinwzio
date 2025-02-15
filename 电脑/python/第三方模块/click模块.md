<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 简介</a></li>
<li><a href="#orgheadline3">2. 必填参数，文件操作和如何测试</a>
<ul>
<li><a href="#orgheadline2">2.1. 标准输入和标准输出</a></li>
</ul>
</li>
<li><a href="#orgheadline4">3. 分组和多个子命令</a></li>
<li><a href="#orgheadline15">4. 命令行选项详解</a>
<ul>
<li><a href="#orgheadline5">4.1. default</a></li>
<li><a href="#orgheadline6">4.2. type</a></li>
<li><a href="#orgheadline7">4.3. 接受多个输入</a></li>
<li><a href="#orgheadline8">4.4. count</a></li>
<li><a href="#orgheadline9">4.5. 布尔值</a></li>
<li><a href="#orgheadline10">4.6. 短名选项和长名选项和布尔值</a></li>
<li><a href="#orgheadline11">4.7. 多个choice选项的用法</a></li>
<li><a href="#orgheadline14">4.8. 请求输入prompt控制</a>
<ul>
<li><a href="#orgheadline12">4.8.1. 请求输入密码</a></li>
<li><a href="#orgheadline13">4.8.2. 请求的默认值控制</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#orgheadline16">5. 全局环境变量控制</a></li>
<li><a href="#orgheadline17">6. 命令行选项控制其他动作</a></li>
<li><a href="#orgheadline18">7. 带颜色的终端回显</a></li>
</ul>
</div>
</nav>


# 简介<a id="orgheadline1"></a>

click模块是一个类似getopt和argparse的python第三方模块，在简单了解之后，觉得其简直就是python快速创建命令行工具的类似requests之于urllib的存在，writed human-friendly, 便民的。click的官方文档在 [这里](http://click.pocoo.org) ，下面就让我们来学习这么好的一个模块吧。

官方文档的第一个例子如下:

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

我们看到一切都是很直观明了的， `@click.command()` 用来装饰一个函数，然后用 `@click.option` 来具体添加命令行选项。其中第一个就是命令行选项的名字，然后 `help` ， `default` 的意义我们是清楚的（类似的还有 `type` 来控制数据类型）。然后我们注意到某个选项比如上面的name是可以请求输入来获得值的。然后我们可以用 `click.echo` 来进行打印操作（不是print函数是为了获得更好的python2和3的兼容性，这个是随意的，直接用print函数也是可以的）。

具体脚本运行情况如下所示:

    wanze@wanze-ubuntu:~/桌面$ python3 test2.py --help
    Usage: test2.py [OPTIONS]
    
      Simple program that greets NAME for a total of COUNT times.
    
    Options:
      --count INTEGER  Number of greetings.
      --name TEXT      The person to greet.
      --help           Show this message and exit.

我们看到该函数的 `__doc__` 说明文档就直接成了该命令行的描述性文档了。

更棒的是其 `prompt` 机制和具体命令行输入参数是兼容的:

    wanze@wanze-ubuntu:~/桌面$ python3 test2.py --count=3 --name=wanze
    Hello wanze!
    Hello wanze!
    Hello wanze!

# 必填参数，文件操作和如何测试<a id="orgheadline3"></a>

必填参数用 `@click.argument()` 来装饰添加之，如下所示:

```python
import click

@click.command()
@click.argument('input', type=click.File('rb'))
@click.argument('output', type=click.File('wb'))
def inout(input, output):
    while True:
        chunk = input.read(1024)
        if not chunk:
            break
        output.write(chunk)

if __name__ == '__main__':
    inout()
```

这里的 `click.File()` 接受一个文件名，然后就已经打开了，在函数里面直接作为一个文件对象可以使用了。文件就推荐使用click模块的这个类型来处理，其对Unicode和bytes处理较好，并有其他优化。当然你也可以使用 `click.Path` 类型，其在函数里面相当于一个优化的Path文件名，同样提供了Unicode和bytes兼容支持等。

```python
@click.command()
@click.argument('f', type=click.Path(exists=True))
def touch(f):
    click.echo(click.format_filename(f))
```

在上面的例子中我们看到脚本最后那里写上:

    if __name__ == '__main__':
        inout()

脚本就可以正常测试了:

    bash>>> python3 test5.py test.py output.txt

但是更好的做法是用测试式开发风格:

```python
import click

from click.testing import CliRunner

def test_inout():
    @click.command()
    @click.argument('input', type=click.File('rb'))
    @click.argument('output', type=click.File('wb'))
    def inout(input, output):
        while True:
            chunk = input.read(1024)
            if not chunk:
                break
            output.write(chunk)

    runner = CliRunner()

    with runner.isolated_filesystem():
        string = 'Hello World!'
        with open('hello.txt', 'w') as f:
            f.write(string)

        result = runner.invoke(inout, ['hello.txt','hello2.txt'])

        assert result.exit_code == 0
        with open('hello2.txt','r') as f:
            s = f.read()
            assert s == string


if __name__ == '__main__':
    test_inout()
```

具体是新建一个 `CliRunner` 对象，然后调用其 `invoke` 方法来具体执行某个命令，然后的 `Result` 对象有 `exit_code` 和 `output` 等属性。其中 `result.output` 一般为屏幕回显的文字。

然后我们看到上面的runner调用了 `isolated_filesystem()` 方法，通过暂停程序我们会发现这样在 `/tmp` 文件夹里面会出现一个临时文件夹，然后一切文件操作都在里面进行，完了就会被删除。

## 标准输入和标准输出<a id="orgheadline2"></a>

值得一提的是标准输入和标准输出可以用 '-' 简单表示。比如上面的例子:

    bash>>> python3 test4.py - output.txt
    test test
    
    bash>>> python3 test4.py test.py -

标准输入的那个例子你需要按下 `Ctrl-D` 来结束文件流。

# 分组和多个子命令<a id="orgheadline4"></a>

click模块在分组和建立多个子命令功能上也设计得很简洁:

```python
import click

@click.group()
def cli():
    pass

@cli.command()
def initdb():
    click.echo('Initialized the database')

@cli.command()
def dropdb():
    click.echo('Dropped the database')

if __name__ == '__main__':
    cli()
```

通过 `@click.group` 来定义某个命令组，然后通过这个命令组函数的command方法 `@cli.command()` 来添加某个子命令。

# 命令行选项详解<a id="orgheadline15"></a>

click模块必填参数通过 `argument()` 引入，然后可选参数通过 `option()` 引入，这里值得一提的是这两个函数的参数并不完全一样，比如说option可以跟 `prompt` 来做到当该可选参数没有输入的时候，则请求输入；但argument并无此概念。更多细节请参看下面的请求输入一小节。

## default<a id="orgheadline5"></a>

设置默认值，显然argument必填参数无此概念。

## type<a id="orgheadline6"></a>

控制数据类型，这个都有。

## 接受多个输入<a id="orgheadline7"></a>

`nargs` 如果设置为大于等于1的值，则命令行中要刷入这么多值，这个和argparse模块类似，不同的是不定量的多个值的情况是 `nargs=-1` ，相当于内置模块argparse的 `*` 。然后对应argparse的 `+` 也就是必须要有一个以上的值的情况，则需要额外加上 `required=True` 来控制。

## count<a id="orgheadline8"></a>

这个是option有，在某种情景下可能很有用。

    @click.command()
    @click.option('-v', '--verbose', count=True)
    def log(verbose):
        click.echo('Verbosity: %s' % verbose)
    $ log -vvv
    Verbosity: 3

## 布尔值<a id="orgheadline9"></a>

如果默认 `default=True` 这样设置了，那么默认就是存储的布尔值了，其实际上暗含加上了 `is_flag=True` 。所以如果没有设置default，则可以通过 `is_flag` 来控制具体存储的是布尔值。

## 短名选项和长名选项和布尔值<a id="orgheadline10"></a>

    import sys
    
    @click.command()
    @click.option('--shout/--no-shout', ' /-S', default=False)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

    bash>>> python3 test.py 
    linux
    bash>>> python3 test.py --shout
    LINUX!!!!111
    bash>>> python3 test.py -S
    linux

也就是通过上面的这种 `/` 分割语句来创建这种多个flag的布尔值控制，其中 `/` 左边是True，右边是False，然后短名选项跟着写入就是了。 **注意** 上面例子短名情况前面的空格是不可少的。

## 多个choice选项的用法<a id="orgheadline11"></a>

```python
@click.command()
@click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
def digest(hash_type):
    click.echo(hash_type)
```

## 请求输入prompt控制<a id="orgheadline14"></a>

这个只是option才有的概念。最简单的情况如下所示:

```python
@click.command()
@click.option('--name', prompt='Your name please')
def hello(name):
    click.echo('Hello %s!' % name)
```

弹出提示只有在你没有输入 `--name=` 给出值时才会出来。

### 请求输入密码<a id="orgheadline12"></a>

如下所示:

```python
@click.command()
@click.option('--password', prompt=True, hide_input=True,
              confirmation_prompt=True)
def encrypt(password):
    click.echo('Encrypting password to %s' % password.encode('rot13'))
```

也就是额外加上了两个选项控制: `hide_input=True` 和 `confirmation_prompt=True` 。上面的这种组合可以简单写为:

```python
@click.command()
@click.password_option()
def encrypt(password):
    click.echo('Encrypting password to %s' % password.encode('rot13'))
```

### 请求的默认值控制<a id="orgheadline13"></a>

请求prompt是可以通过 `default` 来设置默认值的，在那种情况下你直接按下Enter就相当于输入默认值了。然后你还可以如下来获取系统环境下的某个值作为默认值。

```python
@click.command()
@click.option('--username', prompt=True,
              default=lambda: os.environ.get('USER', ''))
def hello(username):
    print("Hello,", username)
```

此外click模块还提供了如下的prompt快捷请求输入命令。

    value = click.prompt('Please enter a valid integer', type=int)

还有如下的 `confirm` 函数也很有用:

    if click.confirm('Do you want to continue?'):
        click.echo('Well done!')

# 全局环境变量控制<a id="orgheadline16"></a>

# 命令行选项控制其他动作<a id="orgheadline17"></a>

如下所示，通过 `is_eager=True` 来让该选项优先级高于其他选项。然后 `expose_value=False` 意思是如果没有输入这个选项，则不影响原命令的执行流。然后 `callback` 就是具体要跳转到的那个函数上。

```python
def print_version(ctx, param, value):
    if not value or ctx.resilient_parsing:
        return
    click.echo('Version 1.0')
    ctx.exit()

@click.command()
@click.option('--version', is_flag=True, callback=print_version,
              expose_value=False, is_eager=True)
def hello():
    click.echo('Hello World!')
```

这里最关键性的问题是理解 `ctx` `param` 和 `value` 这几个参数。

# 带颜色的终端回显<a id="orgheadline18"></a>

click借助python模块 `colorama` 的力量有在终端显示带颜色的字体的功能，首先确认按照了 `colorama` :

    pip install colorama

简单的使用就是:

    import click
    
    click.secho('Hello World!', fg='green')
    click.secho('Some more text', bg='white', fg='black')
    click.secho('ATTENTION', blink=True, bold=True)

其中fg是前景颜色，也可以说字体颜色吧，颜色选项有:

    Fore: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE.

然后bg是背景颜色:

    Back: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE.

然后还有其他一些style:

    dim=True
    bold=True
    blink=True
    underline=True
    reverse=True

其中dim是淡化，bold是加粗，blink意思应该是闪烁，但是没看到效果。underline是下划线，reverse是前景色和背景色翻转。