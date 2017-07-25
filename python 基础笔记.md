#python 基础笔记

python三目运算: a if b else c，即为若b为true，则返回a，否则返回c。

python 的sort、sorted函数。

------

## python 装饰器

今天说说python的装饰器。以下纯属我个人理解，如有不对的地方，还请大家指出。

顾名思义，装饰器就是那些不会对整体框架有所改变，而是扮演一种修饰角色的对象。注意，在python的世界里，万事万物皆对象，函数也是对象。

所以，为什么特地提出了函数？因为装饰器就是修饰函数的函数。在 python的定义中，装饰器只能传递一个参数，即它要所修饰的函数，这里提一句为啥python能将函数作为参数传递，如前所述，聪明的可能会想到，既然 python里万事万物姐对象，那么函数这种对象作为参数传递也就不足为奇了。确实如此。

那么，到底怎样才是一个装饰器呢？先看一个简单的例子：
~~~python
def decoration(func): # 装饰器，装饰函数
  print "call", func.__name__
  func()
    print "call end..."

@decoration
def hello():
    print "Hello world!"

if __name__ == "__main__":
    hello()
~~~
如上所示，decoration就是一个简单的装饰器，装饰器的使用就是在需要添加该装饰器的函数定义之前加上python 的@符，然后调用hello函数就会输出如下的提示
![luozhen > 2017/07/12 > python 基础笔记 > image2017-7-25 14:57:14.png](https://wiki.bytedance.net/download/attachments/91201778/image2017-7-25%2014%3A57%3A14.png?version=1&modificationDate=1500965835000&api=v2)

以上导入的是我写装饰器的脚本文件名。在导入时python解释器就会判断脚本里是否存在主函数，如果存在则直接调用主函数输出。

一般的装饰器就是长这样，但是仅仅这样太呆板了，聪明的你一定发现了，以上被装饰的函数hello并没有参数，如果要装饰一个带参数的函数，这样还行么？我试了下在hello中带上name参数，重新导入模块
~~~python
@decoration
def hello(name):
    print "Hello world! ---to", name

if __name__ == "__main__":
    hello("luozhen")
~~~
![luozhen > 2017/07/12 > python 基础笔记 > image2017-7-25 15:5:53.png](https://wiki.bytedance.net/download/attachments/91201778/image2017-7-25%2015%3A5%3A53.png?version=1&modificationDate=1500966355000&api=v2)

如上所示，的确报错了。报错是hello需要一个参数，而自己并没有传参数进去，这是怎么回事呢？如上，我的确在调用hello的地方传进去了“luozhen”参数。这时候我们需要好好探究一下到底装饰器调用的时候是个什么过程了。把decoration放在hello定义处，在调用hello的时候，就相当于decoration（hello），注意此处的hello传的参数仅仅是一个指向hello函数的指针。于是执行hello（“luozhen”）时，执行的其实是decoration（hello），按照我们之前定义的装饰器，就是先输出“call 函数名”，然后调用该函数，接着输出“call end...”。此处大概能看出端倪了，因为我们在装饰器的定义处调用func()里并没有捕获func()函数的参数，所以如果func()函数有参数，调用时会报错，也就理所应当了。好了，基于这一点，我们改一改
~~~python
def decoration(func): # 装饰器，装饰函数
 print "call", func.__name__
 func(*args, **kw)
    print "call end..."
~~~
这个时候“编译”都不通过，直接语法错误，为啥呢？因为python函数可以调用函数，但是传进func()去的参数应该是调用者函数里可见的参数，因为args, kw是凭空出来的两个东东，肯定就会报错了。又由于装饰器只能传一个函数参数，那么我们该如何在装饰器里调用带参数的函数呢？这里我们就需要用到另外一个函数做辅助了。我们来看看如下：
~~~python
def decoration(func): # 装饰器，装饰函数
 print "call", func.__name__
 def wrapper(*args, **kw):
        func(*args, **kw)
        print "call end..."
 return wrapper
~~~
在decoration中定义了另外一个函数，作为需要传参数的工具，然后我们在decoration中返回该函数。为啥要返回wrapper呢？我们假设不反回wrapper而是直接调用wrapper，这个时候就可知道了，如果是直接在decoration里调用wrapper函数，那么wrapper需要的参数从哪里来？自然这里出现的问题与上一个问题一模一样，并且我们特地定义的wrapper就没啥意义了。还有一点需要说明的是一般函数的参数是放在栈区的，我们返回了wrapper的指针，在调用的时候python解释器就会在栈里查找上下文的参数，所以这个时候wrapper是有用的。并且我把“call end...”放在了wrapper函数里面，自己想一想，大概也知道为什么要这样定义了。如下输出

![luozhen > 2017/07/12 > python 基础笔记 > image2017-7-25 15:43:20.png](https://wiki.bytedance.net/download/attachments/91201778/image2017-7-25%2015%3A43%3A20.png?version=1&modificationDate=1500968600000&api=v2)

综上分析，我们也能知道能够调用正常。我们把以上的wrapper这个函数的功能起了个好听的名字：包装器，这个包装器的功能就是针对带参数的函数的，它的参数*args，**kw可以表示任意参数，至于为啥，可以看看python函数参数相关内容，这里不多做介绍。

还有一点，如果我们想装饰器也带参数，但是装饰器只能传一个函数参数，怎么办？根据以上wrapper 的经验，我们可以再定义一个函数，这个函数里面定义装饰器，并且这个函数可以接受参数。听起来可行的样子，我们试试：
~~~python
def outer(decribe_string):
    print decribe_string
    def decoration(func): # 装饰器，装饰函数
  print "call", func.__name__
  def wrapper(*args, **kw):
            func(*args, **kw)
            print "call end..."
  return wrapper
    return decoration

@outer("this is outer description")
def hello(name):
    print "Hello world! ---to", name

if __name__ == "__main__":
    hello("luozhen")
~~~
如上所示，在outer需要参数的@地方直接传参数。我们在outer里面也只是返回decoration函数指针，道理同上。运行结果如下

![luozhen > 2017/07/12 > python 基础笔记 > image2017-7-25 16:3:55.png](https://wiki.bytedance.net/download/attachments/91201778/image2017-7-25%2016%3A3%3A55.png?version=1&modificationDate=1500969835000&api=v2)

根据之前的介绍，我们把调用hello（“luozhen”）看作是outer（“blablabla...”）(hello)。如此python装饰器基本介绍完了。只是这之中还涉及到调用函数过程中的输出顺序，大家可以自行测试，在测试中要记住的是函数也是对象。只要知道这一点，很容易理清这其中的思路。

------

## python with

## python 连接mysql数据库 防止SQL注入