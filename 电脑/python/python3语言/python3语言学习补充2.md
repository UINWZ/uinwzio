<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. reversed函数</a></li>
<li><a href="#orgheadline2">2. deque结构</a></li>
<li><a href="#orgheadline3">3. 大规模字符串连接推荐使用join方法</a></li>
<li><a href="#orgheadline4">4. 查找多个最大最小元素的情况</a></li>
<li><a href="#orgheadline5">5. 元组和列表的比较大小</a></li>
<li><a href="#orgheadline6">6. 获取一个月最后的一天</a></li>
<li><a href="#orgheadline7">7. OrderedDict类</a></li>
<li><a href="#orgheadline8">8. Counter类</a></li>
<li><a href="#orgheadline9">9. 参考资料</a></li>
</ul>
</div>
</nav>


# reversed函数<a id="orgheadline1"></a>

我学过python有段时间了，竟然第一次听过这个函数，而且这个函数还是python语言的关键词内置函数。学过一点python的懂得炫技的同学都会这样来进行列表反序操作:

    lst[::-1]

而推荐的做法是直接用reversed函数来做:

    reversed(lst)

然后我们马上就想到，列表有 `reverse` 方法，其是破坏型的方法，然后类似的还有 `sort` 方法，破坏型的，其对应非破坏型方法有 `sorted` 。一般使用没有特别需求是都应该使用非破坏型方法，reversed，sorted等等。

# deque结构<a id="orgheadline2"></a>

本小节主要参考了 [这个网页](http://python3-cookbook.readthedocs.io/zh_CN/latest/c01/p03_keep_last_n_items.html) 。

我想读者可能已经接触过queue结构了吧，queue结构是一端进data，然后另一端出data，这样形成了先进先出的数据流。而deque结构两端都可以进两端都可以出，这看上有点古怪，如果你只使用一端的话，那么其好像一个堆栈结构，是先进后出的；而如果一端只是进，另一端只是出，其又好像一个queue结构。那么其有什么优势呢？deque结构最大的优势，也就是我们需要使用它的原因是: 其两端插入元素和删除元素的时间复杂度是O(1)，是一个常数级，而列表开头插入或删除元素的时间复杂度是O(N)，所以如果我们需要一个类似列表的数据存储结构，而这个数据结构中，开头的几个元素和末尾的几个元素都比较重要，经常被访问，那么就应该使用deque结构。

上面的网页介绍了这么一个函数，用来返回一个文件最后的几行:

```python
from collections import deque
def tail(filename, n=10):
    'Return the last n lines of a file'
    with open(filename) as f:
        return deque(f, n)
```

其是利用了deque还有一个size定长的概念，输入的队列进入deque时较老的元素会被丢弃。我不太清楚这种做法效率如何，不过这种写法还是很优雅的。

# 大规模字符串连接推荐使用join方法<a id="orgheadline3"></a>

参看参考资料1的建议27。大规模字符串连接，join方法更有效率。

# 查找多个最大最小元素的情况<a id="orgheadline4"></a>

如果只是想要获知某些数据的一个最大值或者一个最小值，那么当然用 `max` 或 `min` 方法就可以了。这里讨论的情况是如果你想要获知某些数据的多个最大值或多个最小值。一般想到的就是先对这些数据进行排序，然后进行切片操作。参考资料2的第一章第四节讨论的方法实际上是利用最小堆结构进行堆排序然后提出最大或最小的那个几个元素。

大体过程就是:

    lst = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
    import heapq
    heapq.heapify(lst)
    heapq.nlargest(3,lst)
    heapq.nsmallest(3,lst)

# 元组和列表的比较大小<a id="orgheadline5"></a>

元组和列表的相等判断还是很好理解的，而对于这样的东西:

    >>> (1,-1) < (2,-2)

确实就有点古怪了。请读者参考 [这个网页](http://stackoverflow.com/questions/5292303/python-tuple-comparison) ，按照官方文档的说明:

> Tuples and lists are compared lexicographically using comparison of corresponding elements. This means that to compare equal, each element must compare equal and the two sequences must be of the same type and have the same length.

官方文档对于大于小于的情况并没有说得很清楚，然后我们从字里行间大体领会的精神是:

1.  可迭代对象比较大小，是逐个比较的。
2.  逐个比较首先比较是不是相等，如果相等则跳过这个元素的比较，直到遇到某两个不相等的元素，然后返回的就是这两个元素的比较结果。
3.  最后快比较完了（以最小的可迭代对象长度为准），然后如果是相等判断操作，则长度相等就返回相等了。而如果是大小判断操作，则认为长度更长的那个对象更大。

下面是一些例子:

    >>> (1,-1) < (2,-2)
    True
    >>> (1,-1) < (-1,-2)
    False
    >>> (1,-1,-3) < (1,-1)
    False
    >>> (1,-1,) < (1,-1,0)
    True

我们再来看到参考资料第一章第五节的内容，其主要利用两个东西来实现了一种优先级队列。第一个就是前面谈及的最小堆结构，第二个就是这样的 `(priority, index, item)` 三元元组，有了上面的讨论，我想读者马上就看出这其中的最小堆的挤出逻辑了: 那就是优先级最小的先出来，如果优先级相同则索引值最小的先出来，然后索引值没有重复的了。

# 获取一个月最后的一天<a id="orgheadline6"></a>

首先要说的是利用python的datetime和timedelta对于 `days` 的加减操作是能够很好地支持跨月问题的:

    >>> from datetime import datetime
    >>> d = datetime.now()
    >>> d
    datetime.datetime(2016, 5, 29, 8, 50, 20, 337204)
    >>> from datetime import timedelta
    >>> d - timedelta(days = 29)
    datetime.datetime(2016, 4, 30, 8, 50, 20, 337204)
    >>> d - timedelta(days = 28)
    datetime.datetime(2016, 5, 1, 8, 50, 20, 337204)

但是有的时候你就是需要直接获知某个月份的最后一天是30还是31等等，然后利用replace来获得一个月的最后一天。这个时候你需要利用 calendar 的 `monthrange` 函数。参考了[这个网页](http://stackoverflow.com/questions/42950/get-last-day-of-the-month-in-python) 。

    >>> d.replace(year = 2016,month=4,day = monthrange(2016,4)[-1])
    datetime.datetime(2016, 4, 30, 8, 50, 20, 337204)

# OrderedDict类<a id="orgheadline7"></a>

字典一般没有排序的需求吧，就是有也可以输出的时候再排序，再说OrderedDict和一般字典比较起来存储开销大了一倍，能不用就不用吧。不过在某些情况下，用这个类确实能带来一些便利。我第一次遇到这种情况大体是在bilibili的api对接那里，其计算密钥需要将所有参数排序然后urlencode为字符串然后再基于这个字符串进行一些计算。（B站以前的API，似乎后来取消这种机制了。）

    params = OrderedDict(sorted(params.items(), key=lambda t: t[0]))
    string = urlencode(params)

大体在某些情况下，总是要求某个字典值变量按照某个顺序输出，那么用OrderedDict还是很便利的。其顺序就是按照其插入顺序来的，所以进入之前我们还是要做字典排序工作，所以我们可以看作这是一个自动进行了某种操作的便捷对象吧。

# Counter类<a id="orgheadline8"></a>

Counter类是真有用，而且还不是一般的好用。下面的例子来自参考资料2，不多说，看看代码大体就了解了:

    words = [
        'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
        'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
        'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
        'my', 'eyes', "you're", 'under'
    ]
    from collections import Counter
    word_counts = Counter(words)
    # 出现频率最高的3个单词
    top_three = word_counts.most_common(3)
    print(top_three)
    # Outputs [('eyes', 8), ('the', 5), ('look', 4)]

Counter 对象是字典的子类，所以字典的一般方法它都有，下面就不赘述了。然后 `update` 方法我们应该理解为同key之间的加法， 此外还有 `subtract` 方法可以看作同key之间的减法。此外你还可以做:

这种加减运算和上面提及的 update 方法和 subtract 方法还是有点区别的，加法大体类似，主要是减法将会自动去掉计数小于等于零的项，而 `subtract` 方法不会。

    >>> a = Counter(words)
    >>> b = Counter(morewords)
    >>> a
    Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,
    "you're": 1, "don't": 1, 'under': 1, 'not': 1})
    >>> b
    Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,
    'my': 1, 'why': 1})
    >>> # Combine counts
    >>> c = a + b
    >>> c
    Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,
    'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,
    'looking': 1, 'are': 1, 'under': 1, 'you': 1})
    >>> # Subtract counts
    >>> d = a - b
    >>> d
    Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,
    "you're": 1, "don't": 1, 'under': 1})
    >>>

这个数据结构最为人们数值的统计频数了，通过调用 `most_common(n)` 方法，n是排行榜的前n名。

# 参考资料<a id="orgheadline9"></a>

1.  编写高质量代码 改善Python程序的91个建议 张颖 赖勇浩 著
2.  [python3 cookbook](http://python3-cookbook.readthedocs.io/)