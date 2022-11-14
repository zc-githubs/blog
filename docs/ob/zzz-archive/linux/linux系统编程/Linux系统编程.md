
# Gcc

## gcc编译4步骤

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

## gcc编译常用参数

当头文件和源码不在一个目录下时，需要指定头文件

下图是头文件和源码在同一个目录下

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

将hello.h放入新建的文件夹hellodir之后，编译会失败

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

gcc -I ./hellodir hello.c -o hello

其中-I参数指定头文件所在位置，位置可以在编译文件前，也可以在后面

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

-I         指定头文件所在目录位置

-c         只做预处理，编译，汇编。得到二进制文件

-g         编译时添加调试文件，用于gdb调试

-Wall      显示所有警告信息

-D         向程序中“动态”注册宏定义

-l         指定动态库库名

-L         指定动态库路径

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image009.png)

# 自定义库

## 动态库和静态库理论对比

静态库在文件中静态展开，所以有多少文件就展开多少次，非常吃内存，100M展开100次，就是1G，但是这样的好处就是静态加载的速度快

使用动态库会将动态库加载到内存，10个文件也只需要加载一次，然后这些文件用到库的时候临时去加载，速度慢一些，但是很省内存

动态库和静态库各有优劣，根据实际情况合理选用即可。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image011.jpg)

## 静态库制作

静态库名字以lib开头，以.a结尾

例如：libmylib.a

静态库生成指令

ar rcs libmylib.a file1.o

步骤一：

写好源代码

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image013.jpg)

步骤二：

编译源代码生成.o文件

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image015.jpg)

步骤三：

制作静态库

ar rcs libname.a file1.o file2.o …

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image017.jpg)

静态库的使用：

gcc test.c lib库名.a -o a.out

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image019.jpg)

上图中，test.c直接使用了mymath库中的add，sub，div1函数，所以在编译时要导入静态库

编译时出现了函数未定义的警告，可以忽略，让系统生成默认的定义。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image021.jpg)

test.c只有296，然而test有8776，所以静态库使用时，是直接编译到文件里的。

## 静态库使用及头文件对应

上一节出现的警告，用编译器隐式声明来解决的

编译器只能隐式声明返回值为int的函数形式：int add(int ,int )；

如果函数不是返回的int，则隐式声明失效，会报错

在test.c中加入函数声明：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image022.png)

再次进行编译，就不会报错了：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image024.jpg)

上面这个方法，需要库的使用者知道库里的函数，完事儿一个一个加到代码里，不太行

下面使用头文件的方法来加载静态库

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image026.jpg)

左边的define为头文件守卫，防止在代码中多次include头文件，多次展开静态库，带来的额外开销

这样也不会报错了，而且更加科学

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image028.jpg)

下面将静态库和头文件分别放至其他目录下

运行过程如下

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image030.jpg)

## 动态库制作-生成与位置无关代码

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image031.jpg)

写在源代码里的函数，相对main函数偏移是一定的，链接时，回填main函数地址之后，其他源代码里的函数也就得到了地址。

动态库里的函数会用一个@plt来标识，当动态库加载到内存时，再用加载进去的地址将@plt替换掉。

制作动态库的步骤

步骤一：生成位置无关的.o文件

gcc -c add.c -o add.o -fPIC

使用这个参数过后，生成的函数就和位置无关，挂上@plt标识，等待动态绑定

## 动态库制作-演示

制作动态库的步骤

1.  生成位置无关的.o文件

gcc -c add.c -o add.o -fPIC

使用这个参数过后，生成的函数就和位置无关，挂上@plt标识，等待动态绑定

2.   

    使用 gcc -shared制作动态库

    gcc -shared -o lib库名.so add.o sub.o div.o

3.   

    编译可执行程序时指定所使用的动态库。-l:指定库名 -L:指定库路径

    gcc test.c -o a.out -l mymath -L ./lib

4.  运行可执行程序./a.out

过程演示：

步骤一：生成位置无关的.o文件

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image033.jpg)

步骤二：制作动态库 gcc -shared -o lib库名.so add.o sub.o div.o

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image035.jpg)

步骤三：编译程序

文件分布如下：动态库在lib目录下，头文件在inc目录下

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image037.jpg)

下面编译文件

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image039.jpg)

步骤四：执行文件，出错

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image041.jpg)

## 动态库加载错误原因及解决方式

出错原因分析：

连接器：      工作于链接阶段，工作时需要 -l 和 -L

动态链接器：  工作于程序运行阶段，工作时需要提供动态库所在目录位置

指定动态库路径并使其生效，然后再执行文件

通过环境变量指定动态库所在位置：export LD_LIBRARY_PATH=动态库路径

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image043.jpg)

当关闭终端，再次执行a.out时，又报错。

这是因为，环境变量是进程的概念，关闭终端之后再打开，是两个进程，环境变量发生了变化。

要想永久生效，需要修改bash的配置文件：vi ~./bashrc

修改后要使配置文件立即生效：. .bashrc    或者  source .bashrc  或者重开终端让其自己加载

这下再执行a.out就不会报错了

## 动态库加载错误原因及解决方式2

解决方式：          

【1】 通过环境变量：  export LD_LIBRARY_PATH=动态库路径

./a.out 成功！！！  （临时生效， 终端重启环境变量失效）

【2】 永久生效： 写入 终端配置文件。  .bashrc  建议使用绝对路径。

1) vi ~/.bashrc

2) 写入 export LD_LIBRARY_PATH=动态库路径  保存

3）. .bashrc/  source .bashrc / 重启 终端  ---> 让修改后的.bashrc生效

4）./a.out 成功！！！

【3】 拷贝自定义动态库 到 /lib (标准C库所在目录位置)

【4】 配置文件法

1）sudo vi /etc/ld.so.conf

2) 写入 动态库绝对路径  保存

3)  sudo ldconfig -v  使配置文件生效。

4）./a.out 成功！！！--- 使用 ldd  a.out 查看

## 扩展讲解-数据段合并

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image045.jpg)

# Gdb & makefile

## gdb调试基础指令

使用gdb之前，要求对文件进行编译时增加-g参数，加了这个参数过后生成的编译文件会大一些，这是因为增加了gdb调试内容

gdb调试工具：   大前提：程序是你自己写的。  ---逻辑错误

基础指令：

   
```
 -g：使用该参数编译可以执行文件，得到调试表。

    gdb ./a.out

    list： list 1  列出源码。根据源码指定 行号设置断点。

    b： b 20   在20行位置设置断点。

    run/r: 运行程序

    n/next: 下一条指令（会越过函数）

    s/step: 下一条指令（会进入函数）

    p/print：p i  查看变量的值。

    continue：继续执行断点后续指令。

    finish：结束当前函数调用。

    quit：退出gdb当前调试。

```

其他指令：

```

    run：使用run查找段错误出现位置。

    set args： 设置main函数命令行参数 （在 start、run 之前）

    run 字串1 字串2 ...: 设置main函数命令行参数

    info b: 查看断点信息表

    b 20 if i = 5：   设置条件断点。

    ptype：查看变量类型。

    bt：列出当前程序正存活着的栈帧。

    frame： 根据栈帧编号，切换栈帧。

    display：设置跟踪变量

    undisplay：取消设置跟踪变量。 使用跟踪变量的编号。
```


gcc gdbtest.c -o a.out -g

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image046.png)

gdb test  启动对test的调试

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image048.jpg)

l/list n  从第n行开始显示程序，后续继续输入list/l，就可以显示后面的代码

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image050.jpg)

break/b n   在第n行设置断点，断点那一行不会执行

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image052.jpg)

run/r      运行程序

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image053.png)

接下来按next/n/step/s继续向下执行

next/n :下一个，调用函数就跑

step/s :单步，会进入调用的函数

要注意的是，如果是系统函数，按s就出不来了，这时用until+行号直接执行到行号处

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image054.png)

出不来的示例

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image056.jpg)

使用until出来

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image057.png)

print/p i     查看i变量的值

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image058.png)

continue  直接运行到结束

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image060.jpg)

## gdb调试其他指令

其他指令：

    run：使用run查找段错误出现位置。

    set args： 设置main函数命令行参数 （在 start、run 之前）

    run 字串1 字串2 ...: 设置main函数命令行参数

    info b: 查看断点信息表

    b 20 if i = 5：   设置条件断点。

    ptype：查看变量类型。

    bt：列出当前程序正存活着的栈帧。

    frame： 根据栈帧编号，切换栈帧。

    display：设置跟踪变量

    undisplay：取消设置跟踪变量。 使用跟踪变量的编号。

## gdb常见错误说明

没有符号被读取—编译时没加-g参数

file后面加使用-g编译的文件，可以不用退出，gdb直接读取后进行调试。

## makefile基础规则

makefile： 管理项目。

    
```
命名：makefile    Makefile  --- make 命令

    1 个规则：

       目标：依赖条件

       （一个tab缩进）命令

       1. 目标的时间必须晚于依赖条件的时间，否则，更新目标

       2. 依赖条件如果不存在，找寻新的规则去产生依赖条件。

    ALL：指定 makefile 的终极目标。

    2 个函数：

       src = $(wildcard ./*.c): 匹配当前工作目录下的所有.c 文件。将文件名组成列表，赋值给变量 src。  src = add.c sub.c div1.c

       obj = $(patsubst %.c, %.o, $(src)): 将参数3中，包含参数1的部分，替换为参数2。 obj = add.o sub.o div1.o

    clean: (没有依赖)

       -rm -rf $(obj) a.out “-”：作用是，删除不存在文件时，不报错。顺序执行结束。

    3 个自动变量：

        $@: 在规则的命令中，表示规则中的目标。

       $^: 在规则的命令中，表示所有依赖条件。

       $<: 在规则的命令中，表示第一个依赖条件。如果将该变量应用在模式规则中，它可将依赖条件列表中的依赖依次取出，套用模式规则。

    模式规则：

       %.o:%.c

          gcc -c $< -o %@

    静态模式规则：

       $(obj):%.o:%.c

          gcc -c $< -o %@  

    伪目标：

       .PHONY: clean ALL

    参数：

       -n：模拟执行make、make clean 命令。

       -f：指定文件执行 make 命令。              xxxx.mk
```


下面来一步一步升级makefile

第一个版本的Makefile：

makefile的依赖是从上至下的，换句话说就是目标文件是第一句里的目标，如果不满足执行依赖，就会继续向下执行。如果满足了生成目标的依赖，就不会再继续向下执行了。

make会自动寻找规则里需要的材料文件，执行规则下面的行为生成规则中的目标。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image061.png)

执行make指令

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image062.png)

运行hello，没有问题

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image064.jpg)

## makefile的一个规则

修改hello.c如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image066.jpg)

此时要进行编译，则需要多文件联合编译：

gcc hello.c add.c sub.c div1.c -o a.out

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image068.jpg)

对这个新的代码，写出下面的makefile

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image070.jpg)

执行：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image072.jpg)

此时，修改add.c为下图

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image073.png)

此时，再使用make，发现了问题

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image075.jpg)

可以看到，只修改add.c，但是编译的时候，其他.c文件也重新编译了，这不太科学。

明明只改了一个，全部都重新编译了

于是将makefile改写如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image077.jpg)

执行make指令如下

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image079.jpg)

执行程序输出如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image081.jpg)

此时修改sub为下图：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image083.jpg)

再次make

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image085.jpg)

可以看到，只重新编译了修改过的sub.c和最终目标

执行程序：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image087.jpg)

makefile检测原理：

修改文件后，文件的修改时间发生变化，会出现目标文件的时间早于作为依赖材料的时间，出现这种情况的文件会重新编译。

修改sub.c后，sub.o的时间就早于sub.c ，a.out的时间也早于sub.o的时间了，于是重新编译这俩文件了。

关于makefile指定目标问题，先修改makefile如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image089.jpg)

只是将a.out放在了文件末尾

执行make，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image091.jpg)

这是因为，makefile默认第一个目标文件为终极目标，生成就跑路，这时候可以用ALL来指定终极目标

指定目标的makefile

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image093.jpg)

执行：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image095.jpg)

## makefile两个函数和clean

src = $(wildcard *.c)

找到当前目录下所有后缀为.c的文件，赋值给src

obj = $(patsubset %.c,%.o, $(src))

把src变量里所有后缀为.c的文件替换成.o

用这两个函数修改makefile如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image097.jpg)

执行，make指令，如下所示：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image099.jpg)

每次要删除.o文件，很恶心，于是改写makefile如下：

加了clean部分

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image101.jpg)

rm前面的-，代表出错依然执行。比如，待删除文件集合是5个，已经手动删除了1个，就只剩下4个，然而删除命令里面还是5个的集合，就会有删除不存在文件的问题，不加这-，就会报错，告诉你有一个文件找不到。加了-就不会因为这个报错。

执行make：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image103.jpg)

由于没有文件变动，a.out的时间戳晚于所有依赖文件，这里make就没干活

于是，执行时加新指令，先模拟执行clean部分：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image105.jpg)

可以看到模拟执行后，会删除哪些文件。

确定没有问题，执行

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image107.jpg)

## makefile3个自动变量和模式规则

3个自动变量

$@ ：在规则命令中，表示规则中的目标

$< ：在规则命令中，表示规则中的第一个条件，如果将该变量用在模式规则中，它可以将依赖条件列表中的依赖依次取出，套用模式规则

$^ ：在规则命令中，表示规则中的所有条件，组成一个列表，以空格隔开，如果这个列表中有重复项，则去重

用自动变量修改makefile，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image109.jpg)

sub，add这些指令中使用$<和$^都能达到效果，但是为了模式规则，所以使用的$<

执行make，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image111.jpg)

再来，上面这个makefile，可扩展性不行。比如，要添加一个乘法函数，就需要在makefile里面增加乘法函数的部分。不科学，所以，模式规则就来了

%.o:%.c

    gcc -c $< -o $@

修改makefile，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image113.jpg)

执行make，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image115.jpg)

这时，增加一个mul函数，并添加mul.c文件如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image117.jpg)

mul.c如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image119.jpg)

直接执行make：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image121.jpg)

增加函数的时候，不用改makefile，只需要增加.c文件，改一下源码，就行。很强势。

继续优化makefile，使用静态模式规则，就是指定模式规则给谁用，这里指定模式规则给obj用，以后文件多了，文件集合会有很多个，就需要指定哪个文件集合用什么规则

$(obj):%.o:%.c

gcc -c $< -o $@

修改后makefile如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image123.jpg)

运行如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image125.jpg)

再来一个扩展

当前文件夹下有ALL文件或者clean文件时，会导致makefile瘫痪，如下所示，make clean没有工作

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image127.jpg)

用伪目标来解决，添加一行   .PHONY: clean ALL

makefile如下所示：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image129.jpg)

再来执行make clean，就不会受到干扰了

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image131.jpg)

还有一个扩展就是，编译时的参数，-g,-Wall这些，可以放在makefile里面

修改后makefile如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image133.jpg)

执行makefile，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image135.jpg)




## 总结

## gdb调试工具：   大前提：程序是你自己写的。  ---逻辑错误
### 基础指令：
```
 -g：使用该参数编译可以执行文件，得到调试表。

    gdb ./a.out

    list： list 1  列出源码。根据源码指定 行号设置断点。

    b： b 20   在20行位置设置断点。

    run/r: 运行程序

    n/next: 下一条指令（会越过函数）

    s/step: 下一条指令（会进入函数）

    p/print：p i  查看变量的值。

    continue：继续执行断点后续指令。

    finish：结束当前函数调用。

    quit：退出gdb当前调试。

```

其他指令：
```
 run：使用run查找段错误出现位置。

    set args： 设置main函数命令行参数 （在 start、run 之前）

    run 字串1 字串2 ...: 设置main函数命令行参数

    info b: 查看断点信息表

    b 20 if i = 5：   设置条件断点。

    ptype：查看变量类型。

    bt：列出当前程序正存活着的栈帧。

    frame： 根据栈帧编号，切换栈帧。

    display：设置跟踪变量

    undisplay：取消设置跟踪变量。 使用跟踪变量的编号。

```

makefile： 管理项目。

  
```
  命名：makefile    Makefile  --- make 命令

    1 个规则：

       目标：依赖条件

       （一个tab缩进）命令

       1. 目标的时间必须晚于依赖条件的时间，否则，更新目标

       2. 依赖条件如果不存在，找寻新的规则去产生依赖条件。

    ALL：指定 makefile 的终极目标。

    2 个函数：

       src = $(wildcard ./*.c): 匹配当前工作目录下的所有.c 文件。将文件名组成列表，赋值给变量 src。  src = add.c sub.c div1.c

       obj = $(patsubst %.c, %.o, $(src)): 将参数3中，包含参数1的部分，替换为参数2。 obj = add.o sub.o div1.o

    clean: (没有依赖)

       -rm -rf $(obj) a.out “-”：作用是，删除不存在文件时，不报错。顺序执行结束。

    3 个自动变量：

       $@: 在规则的命令中，表示规则中的目标。

       $^: 在规则的命令中，表示所有依赖条件。

       $<: 在规则的命令中，表示第一个依赖条件。如果将该变量应用在模式规则中，它可将依赖条件列表中的依赖依次取出，套用模式规则。

    模式规则：

       %.o:%.c

          gcc -c $< -o %@

    静态模式规则：

       $(obj):%.o:%.c

          gcc -c $< -o %@  

    伪目标：

       .PHONY: clean ALL

    参数：

       -n：模拟执行make、make clean 命令。

       -f：指定文件执行 make 命令。              xxxx.mk

    作业：编写一个 makefile 可以将其所在目录下的所有独立 .c 文件编译生成同名可执行文件。
```


open函数：

# IO
## 系统编程阶段说在前面的话

系统调用          内核提供的函数

库调用            程序库中的函数

## open函数

manpage 第二卷，open函数如下，有两个版本的


```
返回一个文件描述符，理解为整数，出错返回-1

pathname      文件路径

flags         权限控制，只读，只写，读写。  O_RDONLY, O_WRONLY, O_RDWR
```



```
第二个open

多了一个mode参数，用来指定文件的权限，数字设定法

文件权限 = mode & ~umask
```


open常见错误：

1.  打开文件不存在

2.  以写方式打开只读文件（权限问题）

3.  以只写方式打开目录

4.   当open出错时，程序会自动设置errno，可以通过strerror(errno)来查看报错数字的含义
open函数：


``` c
int open(char *pathname, int flags)    #include <unistd.h>

参数：

    pathname: 欲打开的文件路径名

    flags：文件打开方式：    #include <fcntl.h>

    O_RDONLY|O_WRONLY|O_RDWR    O_CREAT|O_APPEND|O_TRUNC|O_EXCL|O_NONBLOCK ....

返回值：

成功： 打开文件所得到对应的 文件描述符（整数）

    失败： -1， 设置errno  

int open(char *pathname, int flags， mode_t mode)       123  775  

参数：

    pathname: 欲打开的文件路径名

    flags：文件打开方式：    O_RDONLY|O_WRONLY|O_RDWR    O_CREAT|O_APPEND|O_TRUNC|O_EXCL|O_NONBLOCK ....

    mode: 参数3使用的前提， 参2指定了 O_CREAT。 取值8进制数，用来描述文件的 访问权限。 rwx    0664

           创建文件最终权限 = mode & ~umask

返回值：

    成功： 打开文件所得到对应的 文件描述符（整数）

    失败： -1， 设置errno  

close函数：

    int close(int fd);
```

## read和write实现cp

### read函数：

   
```
 ssize_t read(int fd, void *buf, size_t count);

    参数：

       fd：文件描述符

       buf：存数据的缓冲区

       count：缓冲区大小

    返回值：

       0：读到文件末尾。

       成功； > 0 读到的字节数。

       失败： -1， 设置 errno

       -1： 并且 errno = EAGIN 或 EWOULDBLOCK, 说明不是read失败，而是read在以非阻塞方式读一个设备文件（网络文件），并且文件无数据。
```


### write函数：

   
```
 ssize_t write(int fd, const void *buf, size_t count);

    参数：

       fd：文件描述符

       buf：待写出数据的缓冲区

       count：数据大小

    返回值：

       成功； 写入的字节数。

       失败： -1， 设置 errno

```


### 错误检测：


```
if(fd1 == -1){

    perror(“open argv[1] error”);

    exit(1);

}

错误处理函数：       与 errno 相关。

printf("xxx error: %d\n", errno);

char *strerror(int errnum);

printf("xxx error: %s\n", strerror(errno));

void perror(const char *s);

    perror("open error");
```




## 系统调用和库函数比较—预读入缓输出

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image165.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image167.png)
![](Pasted%20image%2020221012144914.png)


![](Pasted%20image%2020221012144936.png)
![](Pasted%20image%2020221012152956.png)

首先是fputc/fgetc很快就拷贝完成了。

下面修改read那边的缓冲区，一次拷贝一个字符。速度缓慢。

原因分析：

read/write这块，每次写一个字节，会疯狂进行内核态和用户态的切换，所以非常耗时。

fgetc/fputc，有个缓冲区，4096，所以它并不是一个字节一个字节地写，内核和用户切换就比较少

预读入，缓输出机制。

所以系统函数并不是一定比库函数牛逼，能使用库函数的地方就使用库函数。

标准IO函数自带用户缓冲区，系统调用无用户级缓冲。系统缓冲区是都有的。

## 文件描述符

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image178.jpg)

文件描述符是指向一个文件结构体的指针

PCB进程控制块：本质 结构体。

   
```
 成员：文件描述符表。

    文件描述符：0/1/2/3/4。。。。/1023     表中可用的最小的。

    0 - STDIN_FILENO

    1 - STDOUT_FILENO

    2 - STDERR_FILENO

```

## 阻塞和非阻塞

阻塞、非阻塞：  是设备文件、网络文件的属性。

 
```
   产生阻塞的场景。 读设备文件。读网络文件。（读常规文件无阻塞概念。）

    /dev/tty -- 终端文件。

    open("/dev/tty", O_RDWR|O_NONBLOCK)    --- 设置 /dev/tty 非阻塞状态。(默认为阻塞状态)

```

一个例子，从标准输入读，写到标准输出：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image179.png)

执行程序，就会发现程序在阻塞等待输入

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image181.jpg)

下面是一段更改非阻塞读取终端的代码：

``` C

1.  #include <unistd.h>  

2.  #include <fcntl.h>  

3.  #include <stdlib.h>  

4.  #include <stdio.h>  

5.  #include <errno.h>  

6.  #include <string.h>  

7.    

8.  #define MSG_TRY "try again\n"  

9.  #define MSG_TIMEOUT "time out\n"  

10.   

11. **int** main(**void**)  

12. {  

13.     **char** buf[10];  

14.     **int** fd, n, i;  

15.   

16.     fd = open("/dev/tty", O_RDONLY|O_NONBLOCK);  

17.     **if**(fd < 0){  

18.         perror("open /dev/tty");  

19.         exit(1);  

20.     }  

21.     printf("open /dev/tty ok... %d\n", fd);  

22.   

23.     **for** (i = 0; i < 5; i++){  

24.         n = read(fd, buf, 10);  

25.         **if** (n > 0) {                    //说明读到了东西  

26.             **break**;  

27.         }  

28.         **if** (errno != EAGAIN) {          //EWOULDBLOCK    

29.             perror("read /dev/tty");  

30.             exit(1);  

31.         } **else** {  

32.             write(STDOUT_FILENO, MSG_TRY, strlen(MSG_TRY));  

33.             sleep(2);  

34.         }  

35.     }  

36.   

37.     **if** (i == 5) {  

38.         write(STDOUT_FILENO, MSG_TIMEOUT, strlen(MSG_TIMEOUT));  

39.     } **else** {  

40.         write(STDOUT_FILENO, buf, n);  

41.     }  

42.   

43.     close(fd);  

44.   

45.     **return** 0;  

46. }  

```

执行，如图所示：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image183.jpg)

## fcntl改文件属性

fcntl用来改变一个【已经打开】的文件的 访问控制属性

重点掌握两个参数的使用， F_GETFL，F_SETFL


```
fcntl：

    int (int fd, int cmd, ...)

fd     文件描述符

cmd    命令，决定了后续参数个数

    int flgs = fcntl(fd,  F_GETFL);

    flgs |= O_NONBLOCK

    fcntl(fd,  F_SETFL, flgs);

    获取文件状态： F_GETFL

    设置文件状态： F_SETFL

```

终端文件默认是阻塞读的，这里用fcntl将其更改为非阻塞读

```

1.  #include <unistd.h>  

2.  #include <fcntl.h>  

3.  #include <errno.h>  

4.  #include <stdio.h>  

5.  #include <stdlib.h>  

6.  #include <string.h>  

7.    

8.  #define MSG_TRY "try again\n"  

9.    

10. **int** main(**void**)  

11. {  

12.     **char** buf[10];  

13.     **int** flags, n;  

14.   

15.     flags = fcntl(STDIN_FILENO, F_GETFL); //获取stdin属性信息  

16.     **if**(flags == -1){  

17.         perror("fcntl error");  

18.         exit(1);  

19.     }  

20.     flags |= O_NONBLOCK;  

21.     **int** ret = fcntl(STDIN_FILENO, F_SETFL, flags);  

22.     **if**(ret == -1){  

23.         perror("fcntl error");  

24.         exit(1);  

25.     }  

26.   

27. tryagain:  

28.     n = read(STDIN_FILENO, buf, 10);  

29.     **if**(n < 0){  

30.         **if**(errno != EAGAIN){          

31.             perror("read /dev/tty");  

32.             exit(1);  

33.         }  

34.         sleep(3);  

35.         write(STDOUT_FILENO, MSG_TRY, strlen(MSG_TRY));  

36.         **goto** tryagain;  

37.     }  

38.     write(STDOUT_FILENO, buf, n);  

39.   

40.     **return** 0;  

41. }  

```

可以看到，是非阻塞读取。

## lseek函数

lseek函数：

 
```
   off_t lseek(int fd, off_t offset, int whence);

    参数：

       fd：文件描述符

       offset： 偏移量，就是将读写指针从whence指定位置向后偏移offset个单位

       whence：起始偏移位置： SEEK_SET/SEEK_CUR/SEEK_END

    返回值：

       成功：较起始位置偏移量

       失败：-1 errno

    应用场景：

       1. 文件的“读”、“写”使用同一偏移位置。

       2. 使用lseek获取文件大小

       3. 使用lseek拓展文件大小：要想使文件大小真正拓展，必须引起IO操作。

           使用 truncate 函数，直接拓展文件。 int ret = truncate("dict.cp", 250);

```

lseek示例，写一个句子到空白文件，完事调整光标位置，读取刚才写那个文件。

这个示例中，如果不调整光标位置，是读取不到内容的，因为读写指针在内容的末尾

代码如下：

```

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <unistd.h>  

4.  #include <string.h>  

5.  #include <fcntl.h>  

6.    

7.  **int** main(**void**)  

8.  {  

9.      **int** fd, n;  

10.     **char** msg[] = "It's a test for lseek\n";  

11.     **char** ch;  

12.   

13.     fd = open("lseek.txt", O_RDWR|O_CREAT, 0644);  

14.     **if**(fd < 0){  

15.         perror("open lseek.txt error");  

16.         exit(1);  

17.     }  

18.   

19.     write(fd, msg, strlen(msg));    //使用fd对打开的文件进行写操作，问价读写位置位于文件结尾处。  

20.   

21.     lseek(fd, 0, SEEK_SET);         //修改文件读写指针位置，位于文件开头。 注释该行会怎样呢？  

22.   

23.     **while**((n = read(fd, &ch, 1))){  

24.         **if**(n < 0){  

25.             perror("read error");  

26.             exit(1);  

27.         }  

28.         write(STDOUT_FILENO, &ch, n);   //将文件内容按字节读出，写出到屏幕  

29.     }  

30.   

31.     close(fd);  

32.   

33.     **return** 0;  

34. }  
```


![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image203.jpg)

简单小结一下：

对于写文件再读取那个例子，由于文件写完之后未关闭，读写指针在文件末尾，所以不调节指针，直接读取不到内容。

lseek读取的文件大小总是相对文件头部而言。

用lseek读取文件大小实际用的是读写指针初末位置的偏移差，一个新开文件，读写指针初位置都在文件开头。如果用这个来扩展文件大小，必须引起IO才行，于是就至少要写入一个字符。上面代码出现lseek返回799，ls查看为800的原因是，lseek读取到偏移差的时候，还没有写入最后的‘$’符号.

末尾那一大堆^@，是文件空洞，如果自己写进去的也想保持队形，就写入“\0”。

拓展文件直接使用truncate，简单粗暴：

    使用 truncate 函数，直接拓展文件。 int ret = truncate("dict.cp", 250);

## 传入传出参数

传入参数：


```
    1. 指针作为函数参数。

    2. 同常有const关键字修饰。

    3. 指针指向有效区域， 在函数内部做读操作。
```


传出参数：


```
    1. 指针作为函数参数。

    2. 在函数调用之前，指针指向的空间可以无意义，但必须有效。

    3. 在函数内部，做写操作。

    4。函数调用结束后，充当函数返回值。
```


传入传出参数：

  
```
  1. 指针作为函数参数。

    2. 在函数调用之前，指针指向的空间有实际意义。

    3. 在函数内部，先做读操作，后做写操作。

    4. 函数调用结束后，充当函数返回值。
```


## 目录项和inode

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image205.png)

一个文件主要由两部分组成，dentry(目录项)和inode

inode本质是结构体，存储文件的属性信息，如：权限、类型、大小、时间、用户、盘快位置…

也叫做文件属性管理结构，大多数的inode都存储在磁盘上。

    少量常用、近期使用的inode会被缓存到内存中。

所谓的删除文件，就是删除inode，但是数据其实还是在硬盘上，以后会覆盖掉。

## stat函数

获取文件属性，（从inode结构体中获取）

stat/lstat 函数：

```

    int stat(const char *path, struct stat *buf);

    参数：

       path： 文件路径

       buf：（传出参数） 存放文件属性，inode结构体指针。

    返回值：

       成功： 0

       失败： -1 errno

    获取文件大小： buf.st_size

    获取文件类型： buf.st_mode

    获取文件权限： buf.st_mode

    符号穿透：stat会。lstat不会。

```

下面这个例子是获取文件大小的正规军解法，用stat：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image207.jpg)

编译运行，并查看mystat.c文件的大小，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image209.jpg)


![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image215.jpg)

## 传出参数当返回值

其实就是传出参数在函数中被改变，可以用来传出。而且传出参数数量不限，比单纯的返回值更灵活。

## link和Unlink隐式回收

硬链接数就是dentry数目

link就是用来创建硬链接的

link可以用来实现mv命令

函数原型：
### link函数
```

int link(const char *oldpath, const char *newpath)

用这个来实现mv，用oldpath来创建newpath，完事儿删除oldpath就行。

删除一个链接  int unlink(const char *pathname)
```

### unlink函数
unlink是删除一个文件的目录项dentry，使硬链接数-1

unlink函数的特征：清除文件时，如果文件的硬链接数到0了，没有dentry对应，但该文件仍不会马上被释放，要等到所有打开文件的进程关闭该文件，系统才会挑时间将该文件释放掉。

下面用一段代码来验证unlink是删除dentry：


```

4.  #include <unistd.h>  

5.  #include <fcntl.h>  

6.  #include <stdlib.h>  

7.  #include <string.h>  

8.  #include <stdio.h>  

9.    

10.   

11. **int** main(**void**)  

12. {  

13.     **int** fd, ret;  

14.     **char** *p = "test of unlink\n";  

15.     **char** *p2 = "after write something.\n";  

16.   

17.     fd = open("temp.txt", O_RDWR|O_CREAT|O_TRUNC, 0644);  

18.     **if**(fd < 0){  

19.         perror("open temp error");  

20.         exit(1);  

21.     }  

22.   

23.     ret = write(fd, p, strlen(p));  

24.     **if** (ret == -1) {  

25.         perror("-----write error");  

26.     }  

27.   

28.     printf("hi! I'm printf\n");  

29.     ret = write(fd, p2, strlen(p2));  

30.     **if** (ret == -1) {  

31.         perror("-----write error");  

32.     }  

33.   

34.     printf("Enter anykey continue\n");  

35.     getchar();  

36.   

37.     ret = unlink("temp.txt");        //具备了被释放的条件  

38.     **if**(ret < 0){  

39.         perror("unlink error");  

40.         exit(1);  

41.     }  

42.   

43.     close(fd);  

44.   

45.     **return** 0;  

46. }  

```

编译程序并运行，程序阻塞，此时打开新终端查看临时文件temp.c如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image217.jpg)

可以看到，临时文件没有被删除，这是因为当前进程没结束。

输入字符使当前进程结束后，temp.txt就不见了

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image218.png)

下面开始搞事，在程序中加入段错误成分，段错误在unlink之前，由于发生段错误，程序后续删除temp.txt的dentry部分就不会再执行，temp.txt就保留了下来，这是不科学的。

解决办法是检测fd有效性后，立即释放temp.txt，由于进程未结束，虽然temp.txt的硬链接数已经为0，但还不会立即释放，仍然存在，要等到程序执行完才会释放。这样就能避免程序出错导致临时文件保留下来。

因为文件创建后，硬链接数立马减为0，即使程序异常退出，这个文件也会被清理掉。这时候的内容是写在内核空间的缓冲区。

修改后代码如下： 

``` c

 

4.  #include <unistd.h>  

5.  #include <fcntl.h>  

6.  #include <stdlib.h>  

7.  #include <string.h>  

8.  #include <stdio.h>  

9.    

10.   

11. **int** main(**void**)  

12. {  

13.     **int** fd, ret;  

14.     **char** *p = "test of unlink\n";  

15.     **char** *p2 = "after write something.\n";  

16.   

17.     fd = open("temp.txt", O_RDWR|O_CREAT|O_TRUNC, 0644);  

18.     **if**(fd < 0){  

19.         perror("open temp error");  

20.         exit(1);  

21.     }  

22.   

23.     ret = unlink("temp.txt");        //具备了被释放的条件  

24.     **if**(ret < 0){  

25.         perror("unlink error");  

26.         exit(1);  

27.     }  

28.   

29.     ret = write(fd, p, strlen(p));  

30.     **if** (ret == -1) {  

31.         perror("-----write error");  

32.     }  

33.   

34.     printf("hi! I'm printf\n");  

35.     ret = write(fd, p2, strlen(p2));  

36.     **if** (ret == -1) {  

37.         perror("-----write error");  

38.     }  

39.   

40.     printf("Enter anykey continue\n");  

41.     getchar();  

42.   

43.     close(fd);  

44.   

45.     **return** 0;  

46. }  

```

隐式回收：

当进程结束运行时，所有进程打开的文件会被关闭，申请的内存空间会被释放。系统的这一特性称之为隐式回收系统资源。

比如上面那个程序，要是没有在程序中关闭文件描述符，没有隐式回收的话，这个文件描述符会保留，多次出现这种情况会导致系统文件描述符耗尽。所以隐式回收会在程序结束时收回它打开的文件使用的文件描述符。

## 目录操作函数

目录操作函数：
### opendir closedir  readdir 

```
    DIR * opendir(char *name);

    int closedir(DIR *dp);

    struct dirent *readdir(DIR * dp);

       struct dirent {

           inode

           char dname[256];

       }

```

没有写目录操作，因为目录写操作就是创建文件。可以用touch

下面用目录操作函数实现一个ls操作：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image222.jpg)

编译执行，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image224.jpg)

要隐藏这个.和..的话，在输出文件名的时候判定一下，只输出不是.和..的就行了

## 总结
### 标准io 

![](Pasted%20image%2020221012152956.png)
``` c


FIFE *fopen(const char *path, const char *mode);
/*
path: 打开文件路径
mode: 打开文件模式
*/
 
int fclose(FIFE *stream)
// 关闭当前文件流
 
int fgetc(FIFE *stream)
// 读取一个字符，返回该字符转换为int类型的值
 
int fputc(int c, FIFE *stream);
// 把c转换为对应的字符，写到stream里
 
char *fgets(char *s, int size, FIFE *stream);
// 读到换行符或者EOF截止，结果保存到s中，会多保存一个'\0'
 
int fputs(const char *s, FIFE *stream);
// 把s除去'\0'之外的字符写到stream里
 
size_t fread(void *ptr, size_t size, size_t nmemb, FIFE *stream);
// 读nmemb个元素，每个大小size字节，保存到ptr里
 
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FIFE *stream);
// 写nmemb个元素，每个大小size字节，写到stream里
 
// 文件位置指针相关：
int fseek(FIFE *stream, long offset, int whence)
long ftell(FIFE *stream)
void rewind(FIFE *stream)
int fflush(FIFE *stream)
 
// printf/scanf一族
int printf(const char *format);
int fprintf(FIFE *stream, const char *format, ...);
int sprintf(char *str, const  char *format, ...);   //可以看成是atoi的反向函数
//sprintf()可能会造成缓冲区溢出
int snprintf(char *str, size_t size, const char *format, ...);  
 
int scanf(const char *format, ...);\
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char* format, ...);
```


### open函数：


```
    int open(char *pathname, int flags)    #include <unistd.h>

    参数：

       pathname: 欲打开的文件路径名

       flags：文件打开方式：    #include <fcntl.h>

           O_RDONLY|O_WRONLY|O_RDWR    O_CREAT|O_APPEND|O_TRUNC|O_EXCL|O_NONBLOCK ....

    返回值：

       成功： 打开文件所得到对应的 文件描述符（整数）

       失败： -1， 设置errno  

    int open(char *pathname, int flags， mode_t mode)       123  775   

    参数：

       pathname: 欲打开的文件路径名

       flags：文件打开方式：    O_RDONLY|O_WRONLY|O_RDWR    O_CREAT|O_APPEND|O_TRUNC|O_EXCL|O_NONBLOCK ....

       mode: 参数3使用的前提， 参2指定了 O_CREAT。 取值8进制数，用来描述文件的 访问权限。 rwx    0664

           创建文件最终权限 = mode & ~umask

    返回值：

       成功： 打开文件所得到对应的 文件描述符（整数）

       失败： -1， 设置errno  
```


### close函数：

  
```
  int close(int fd);
```


错误处理函数：       与 errno 相关。

 
```
   printf("xxx error: %d\n", errno);

    char *strerror(int errnum);

       printf("xxx error: %s\n", strerror(errno));

    void perror(const char *s);

       perror("open error");
```


### read函数：

  
```
  ssize_t read(int fd, void *buf, size_t count);

    参数：

       fd：文件描述符

       buf：存数据的缓冲区

       count：缓冲区大小

    返回值：

       0：读到文件末尾。

       成功； > 0 读到的字节数。

       失败： -1， 设置 errno

       -1： 并且 errno = EAGIN 或 EWOULDBLOCK, 说明不是read失败，而是read在以非阻塞方式读一个设备文件（网络文件），并且文件无数据。
```


### write函数：

  
```
  ssize_t write(int fd, const void *buf, size_t count);

    参数：

       fd：文件描述符

       buf：待写出数据的缓冲区

       count：数据大小

    返回值：

       成功； 写入的字节数。

       失败： -1， 设置 errno
```


### 文件描述符：

  
```
  PCB进程控制块：本质 结构体。

    成员：文件描述符表。

    文件描述符：0/1/2/3/4。。。。/1023     表中可用的最小的。

    0 - STDIN_FILENO

    1 - STDOUT_FILENO

    2 - STDERR_FILENO

```

### 阻塞、非阻塞：  是设备文件、网络文件的属性。

```

    产生阻塞的场景。 读设备文件。读网络文件。（读常规文件无阻塞概念。）

    /dev/tty -- 终端文件。

    open("/dev/tty", O_RDWR|O_NONBLOCK)    --- 设置 /dev/tty 非阻塞状态。(默认为阻塞状态)
```


### fcntl：

   
```
 int (int fd, int cmd, ...)

    int flgs = fcntl(fd,  F_GETFL);

    flgs |= O_NONBLOCK

    fcntl(fd,  F_SETFL, flgs);

    获取文件状态： F_GETFL

    设置文件状态： F_SETFL
```


### lseek函数：

 
```
   off_t lseek(int fd, off_t offset, int whence);

    参数：

       fd：文件描述符

       offset： 偏移量

       whence：起始偏移位置： SEEK_SET/SEEK_CUR/SEEK_END

    返回值：

       成功：较起始位置偏移量

       失败：-1 errno

    应用场景：

       1. 文件的“读”、“写”使用同一偏移位置。

       2. 使用lseek获取文件大小

       3. 使用lseek拓展文件大小：要想使文件大小真正拓展，必须引起IO操作。

           使用 truncate 函数，直接拓展文件。 int ret = truncate("dict.cp", 250);

```

### 传入参数：

```

    1. 指针作为函数参数。

    2. 同常有const关键字修饰。

    3. 指针指向有效区域， 在函数内部做读操作。
```


### 传出参数：


```
    1. 指针作为函数参数。

    2. 在函数调用之前，指针指向的空间可以无意义，但必须有效。

    3. 在函数内部，做写操作。

    4。函数调用结束后，充当函数返回值。

```

### 传入传出参数：


```
    1. 指针作为函数参数。

    2. 在函数调用之前，指针指向的空间有实际意义。

    3. 在函数内部，先做读操作，后做写操作。

    4. 函数调用结束后，充当函数返回值。

```


### stat/lstat 函数：

 
```
   int stat(const char *path, struct stat *buf);

    参数：

       path： 文件路径

       buf：（传出参数） 存放文件属性。

    返回值：

       成功： 0

       失败： -1 errno

    获取文件大小： buf.st_size

    获取文件类型： buf.st_mode

    获取文件权限： buf.st_mode

    符号穿透：stat会。lstat不会。
```


### link/unlink:

    
```
隐式回收。
```


### 目录操作函数：


```
    DIR * opendir(char *name);

    int closedir(DIR *dp);

    struct dirent *readdir(DIR * dp);

       struct dirent {

           inode

           char dname[256];

       }

```



### dup 和 dup2：

    int dup(int oldfd);      文件描述符复制。

       oldfd: 已有文件描述符

       返回：新文件描述符。

    int dup2(int oldfd, int newfd); 文件描述符复制。重定向。

### fcntl 函数实现 dup：

    int fcntl(int fd, int cmd, ....)

    cmd: F_DUPFD

    参3:  被占用的，返回最小可用的。

       未被占用的， 返回=该值的文件描述符。


![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image257.jpg)

# 进程
## 进程和程序以及CPU相关

进程：

  
```
  程序：死的。只占用磁盘空间。       ——剧本。

    进程；活的。运行起来的程序。占用内存、cpu等系统资源。   ——戏。

```

并发和并行：并行是宏观上并发，微观上串行

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image251.jpg)
![](Pasted%20image%2020221012150816.png)
## 虚拟内存和物理内存映射关系
![](Pasted%20image%2020221012150837.png)
![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image253.jpg)

## pcb进程控制块

PCB进程控制块：


```
    进程id

    文件描述符表

    进程状态： 初始态、就绪态、运行态、挂起态、终止态。

    进程工作目录位置

    *umask掩码 （进程的概念）

    信号相关信息资源。

    用户id和组id
```


ps aux 返回结果里，第二列是进程id

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image255.jpg)



## fork函数原理

fork函数：


```
    pid_t fork(void)

    创建子进程。父子进程各自返回。父进程返回子进程pid。 子进程返回 0.

    getpid();getppid();

    循环创建N个子进程模型。 每个子进程标识自己的身份。
```


父子进程相同：


```
    刚fork后。 data段、text段、堆、栈、环境变量、全局变量、宿主目录位置、进程工作目录位置、信号处理方式
```


父子进程不同：

   
```
 进程id、返回值、各自的父进程、进程创建时间、闹钟、未决信号集
```


父子进程共享：

 
```
   读时共享、写时复制。———————— 全局变量。
```


1.  文件描述符 2. mmap映射区。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image259.jpg)

## fork创建子进程

fork之前的代码，父子进程都有，但是只有父进程执行了，子进程没有执行，fork之后的代码，父子进程都有机会执行。

两个函数：

### pid_t getpid()       获取当前进程id

### pid_t getppid()      获取当前进程的父进程id


## 父子进程gdb调试

gdb调试：


```
    设置父进程调试路径：set follow-fork-mode parent (默认)

    设置子进程调试路径：set follow-fork-mode child
```


注意，一定要在fork函数调用之前设置才有效。

## exec函数族

exec函数族：

  
```

#include <unistd.h>
extern char **environ;

int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg,..., char * const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[],char *const envp[]);
  使进程执行某一程序。成功无返回值，失败返回 -1

  返回值：
　　exec函数族的函数执行成功后不会返回，调用失败时，会设置errno并返回-1，然后从原程序的调用点接着往下执行。
参数说明：
path：可执行文件的路径名字
arg：可执行程序所带的参数，第一个参数为可执行文件名字，没有带路径且arg必须以NULL结束
file：如果参数file中包含/，则就将其视为路径名，否则就按 PATH环境变量，在它所指定的各目录中搜寻可执行文件。

exec族函数参数极难记忆和分辨，函数名中的字符会给我们一些帮助：
l : 使用参数列表
p：使用文件名，并从PATH环境进行寻找可执行文件
v：应先构造一个指向各参数的指针数组，然后将该数组的地址作为这些函数的参数。
e：多了envp[]数组，使用新的环境变量代替调用进程的环境变量

下面将exac函数归为带l、带p、带v、带e 四类来说明参数特点。

一、带l的一类exac函数（l表示list），包括execl、execlp、execle，要求将新程序的每个命令行参数都说明为 一个单独的参数。这种参数表以空指针结尾。
二、带p的一类exac函数，包括execlp、execvp、execvpe，如果参数file中包含/，则就将其视为路径名，否则就按 PATH环境变量，在它所指定的各目录中搜寻可执行文件。举个例子，PATH=/bin:/usr/bin

三、带v不带l的一类exac函数，包括execv、execvp、execve，应先构造一个指向各参数的指针数组，然后将该数组的地址作为这些函数的参数

四、带e的一类exac函数，包括execle、execvpe，可以传递一个指向环境字符串指针数组的指针。** 参数例如char *env_init[] = {“AA=aa”,”BB=bb”,NULL}; 带e表示该函数取envp[]数组，而不使用当前环境。。


```


ps ajx --> pid ppid gid sid

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image291.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image293.jpg)

## execlp和ececl函数

int execlp(const char *file, const char *arg, …)

成功，无返回，失败返回-1

参数1：要加载的程序名字，该函数需要配合PATH环境变量来使用，当PATH所有目录搜素后没有参数1则返回出错。

该函数通常用来调用系统程序。如ls、date、cp、cat命令。

execlp这里面的p，表示要借助环境变量来加载可执行文件


可变参数那里，是从argv[0]开始计算的。



int execl(const char *path, const char *arg, …)

这里要注意，和execlp不同的是，第一个参数是路径，不是文件名。

这个路径用相对路径和绝对路径都行。

## exec函数族特性



exec函数族一般规律：

exec函数一旦调用成功，即执行新的程序，不返回。只有失败才返回，错误值-1，所以通常我们直接在exec函数调用后直接调用perror()，和exit()，无需if判断。

l(list)              命令行参数列表

p(path)              搜索file时使用path变量

v(vector)            使用命令行参数数组

e(environment)       使用环境变量数组，不适用进程原有的环境变量，设置新加载程序运行的环境变量

事实上，只有execve是真正的系统调用，其他5个函数最终都调用execve，是库函数，所以execve在man手册第二节，其它函数在man手册第3节。

## 孤儿进程和僵尸进程

孤儿进程：

    
```
父进程先于子进终止，子进程沦为“孤儿进程”，会被 init 进程领养。
```


僵尸进程：

 
```
   子进程终止，父进程尚未对子进程进行回收，在此期间，子进程为“僵尸进程”。  kill 对其无效。这里要注意，每个进程结束后都必然会经历僵尸态，时间长短的差别而已。

    子进程终止时，子进程残留资源PCB存放于内核中，PCB记录了进程结束原因，进程回收就是回收PCB。回收僵尸进程，得kill它的父进程，让孤儿院去回收它。

```

## wait回收子进程

### wait函数：    回收子进程退出资源， 阻塞回收任意一个。


```
    pid_t wait(int *status)

    参数：（传出） 回收进程的状态。

    返回值：成功： 回收进程的pid

       失败： -1， errno

    函数作用1：   阻塞等待子进程退出

    函数作用2：   清理子进程残留在内核的 pcb 资源

    函数作用3：   通过传出参数，得到子进程结束状态

```

  
```
  获取子进程正常终止值：

       WIFEXITED(status) --》 为真 --》调用 WEXITSTATUS(status) --》 得到 子进程 退出值。

    获取导致子进程异常终止信号：

       WIFSIGNALED(status) --》 为真 --》调用 WTERMSIG(status) --》 得到 导致子进程异常终止的信号编号。
```


一个进程终止时会关闭所有文件描述符，释放在用户空间分配的内存，但它的PCB还保留着，内核在其中保存了一些信息：如果是正常终止则保存着退出状态，如果是异常终止则保存着导致该进程终止的信号是哪个。这个进程的父进程可以调用wait或者waitpid获取这些信息，然后彻底清除掉这个进程。我们知道一个进程的退出状态可以在shell中用特殊变量$？查看，因为shell是它的父进程，当它终止时，shell调用wait或者waitpid得到它的退出状态，同时彻底清除掉这个进程。

pid_t wait(int *status)

其中status是传出参数

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image314.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image320.jpg)

## waitpid回收子进程

waitpid函数： 指定某一个进程进行回收。可以设置非阻塞。        

waitpid(-1, &status, 0) == wait(&status);

```

    pid_t waitpid(pid_t pid, int *status, int options)

    参数：

       pid：指定回收某一个子进程pid

           > 0: 待回收的子进程pid

           -1：任意子进程

           0：同组的子进程。

       status：（传出） 回收进程的状态。

       options：WNOHANG 指定回收方式为，非阻塞。

    返回值：

       > 0 : 表成功回收的子进程 pid

       0 : 函数调用时， 参3 指定了WNOHANG， 并且，没有子进程结束。

       -1: 失败。errno
```


一次wait/waitpid函数调用，只能回收一个子进程。上一个例子，父进程产生了5个子进程，wait会随机回收一个，捡到哪个算哪个。

```
//wait(NULL);                           
// 一次wait/waitpid函数调用,只能回收一个子进程.  

//wpid = waitpid(-1, NULL, WNOHANG);    
//回收任意子进程,没有结束的子进程,父进程直接返回0   

//wpid = waitpid(tmpid, NULL, 0);       
//指定一个进程回收, 阻塞等待  

//wpid = waitpid(tmpid, NULL, WNOHANG);   //指定一个进程回收, 不阻塞  

wpid = waitpid(tmpid, NULL, 0);         //指定一个进程回收, 阻塞回收 


```


![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image326.jpg)


## 进程间通信常见方式

IPC(InterProcess Communication)进程间通信

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image330.jpg)

进程间通信的常用方式，特征：

    
```
    管道：简单
    

    信号：开销小

    mmap映射：非血缘关系进程间
	system-v-ipc（内存共享、消息队列、信号量）

    socket（本地套接字）：稳定

```

## 管道的特质

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image331.jpg)

管道：

  
```
  实现原理： 内核借助环形队列机制，使用内核缓冲区实现。

    特质； 1. 伪文件

       2. 管道中的数据只能一次读取。

       3. 数据在管道中，只能单向流动。

    局限性：1. 自己写，不能自己读。

       2. 数据不可以反复读。

       3. 半双工通信。

       4. 血缘关系进程间可用。

```

## 管道的基本用法

pipe函数：    创建，并打开管道。


```
    int pipe(int fd[2]);

    参数： fd[0]: 读端。

       fd[1]: 写端。

    返回值： 成功： 0

        失败： -1 errno

```

管道通信原理：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image333.jpg)

一个管道通信的示例，父进程往管道里写，子进程从管道读，然后打印读取的内容：


```
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.  **void** sys_err(**const** **char** *str)  

9.  {  

10.     perror(str);  

11.     exit(1);  

12. }  

13.   

14. **int** main(**int** argc, **char** *argv[])  

15. {  

16.     **int** ret;  

17.     **int** fd[2];  

18.     pid_t pid;  

19.   

20.     **char** *str = "hello pipe\n";  

21.     **char** buf[1024];  

22.   

23.     ret = pipe(fd);  

24.     **if** (ret == -1)  

25.         sys_err("pipe error");  

26.   

27.     pid = fork();  

28.     **if** (pid > 0) {  

29.         close(fd[0]);       // 关闭读段  

30.         //sleep(3);  

31.         write(fd[1], str, strlen(str));  

32.         close(fd[1]);  

33.     } **else** **if** (pid == 0) {  

34.         close(fd[1]);       // 子进程关闭写段  

35.         ret = read(fd[0], buf, **sizeof**(buf));  

36.         printf("child read ret = %d\n", ret);  

37.         write(STDOUT_FILENO, buf, ret);  

38.   

39.         close(fd[0]);  

40.     }  

41.   

42.     **return** 0;  

43. }  

```



## 管道读写行为

管道的读写行为：

    
```
读管道：

       1. 管道有数据，read返回实际读到的字节数。

       2. 管道无数据：   1）无写端，read返回0 （类似读到文件尾）

              2）有写端，read阻塞等待。

```

 
```
   写管道：

       1. 无读端， 异常终止。 （SIGPIPE导致的）

       2. 有读端：   1） 管道已满， 阻塞等待

2） 管道未满， 返回写出的字节个数。
```




## 总结

gdb调试：

    设置父进程调试路径：set follow-fork-mode parent (默认)

    设置子进程调试路径：set follow-fork-mode child

exec函数族：

    使进程执行某一程序。成功无返回值，失败返回 -1

    int execlp(const char *file, const char *arg, ...);     借助 PATH 环境变量找寻待执行程序

       参1： 程序名

       参2： argv0

       参3： argv1

       ...： argvN

       哨兵：NULL

    int execl(const char *path, const char *arg, ...);      自己指定待执行程序路径。

    int execvp();

ps ajx --> pid ppid gid sid

孤儿进程：

    父进程先于子进终止，子进程沦为“孤儿进程”，会被 init 进程领养。

僵尸进程：

    子进程终止，父进程尚未对子进程进行回收，在此期间，子进程为“僵尸进程”。  kill 对其无效。

wait函数：    回收子进程退出资源， 阻塞回收任意一个。

    pid_t wait(int *status)

    参数：（传出） 回收进程的状态。

    返回值：成功： 回收进程的pid

       失败： -1， errno

    函数作用1：   阻塞等待子进程退出

    函数作用2：   清理子进程残留在内核的 pcb 资源

    函数作用3：   通过传出参数，得到子进程结束状态

    获取子进程正常终止值：

       WIFEXITED(status) --》 为真 --》调用 WEXITSTATUS(status) --》 得到 子进程 退出值。

    获取导致子进程异常终止信号：

       WIFSIGNALED(status) --》 为真 --》调用 WTERMSIG(status) --》 得到 导致子进程异常终止的信号编号。

waitpid函数： 指定某一个进程进行回收。可以设置非阻塞。          waitpid(-1, &status, 0) == wait(&status);

    pid_t waitpid(pid_t pid, int *status, int options)

    参数：

       pid：指定回收某一个子进程pid

           > 0: 待回收的子进程pid

           -1：任意子进程

           0：同组的子进程。

       status：（传出） 回收进程的状态。

       options：WNOHANG 指定回收方式为，非阻塞。

    返回值：

       > 0 : 表成功回收的子进程 pid

       0 : 函数调用时， 参3 指定了WNOHANG， 并且，没有子进程结束。

       -1: 失败。errno

总结：

    wait、waitpid 一次调用，回收一个子进程。

           想回收多个。while

===========================

进程间通信的常用方式，特征：

    管道：简单

    信号：开销小

    mmap映射：非血缘关系进程间

    socket（本地套接字）：稳定

管道：

    实现原理： 内核借助环形队列机制，使用内核缓冲区实现。

    特质； 1. 伪文件

       2. 管道中的数据只能一次读取。

       3. 数据在管道中，只能单向流动。

    局限性：1. 自己写，不能自己读。

       2. 数据不可以反复读。

       3. 半双工通信。

       4. 血缘关系进程间可用。

pipe函数：    创建，并打开管道。

    int pipe(int fd[2]);

    参数： fd[0]: 读端。

       fd[1]: 写端。

    返回值： 成功： 0

        失败： -1 errno

管道的读写行为：

    读管道：

       1. 管道有数据，read返回实际读到的字节数。

       2. 管道无数据：   1）无写端，read返回0 （类似读到文件尾）

              2）有写端，read阻塞等待。

    写管道：

       1. 无读端， 异常终止。 （SIGPIPE导致的）

       2. 有读端：   1） 管道已满， 阻塞等待

              2） 管道未满， 返回写出的字节个数。



## 107P-父子进程lswc-l

练习：使用管道实现父子进程间通信，完成：ls | wc -l  假定父进程实现ls，子进程实现wc

ls命令正常会将结果集写到stdout，但现在会写入管道写端

wc -l命令正常应该从stdin读取数据，但此时会从管道的读端读。

要用到 pipe  dup2  exec

代码如下，还是蛮简单的：

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.  **void** sys_err(**const** **char** *str)  

9.  {  

10.     perror(str);  

11.     exit(1);  

12. }  

13. **int** main(**int** argc, **char** *argv[])  

14. {  

15.     **int** fd[2];  

16.     **int** ret;  

17.     pid_t pid;  

18.   

19.     ret = pipe(fd);                 // 父进程先创建一个管道,持有管道的读端和写端  

20.     **if** (ret == -1) {  

21.         sys_err("pipe error");  

22.     }  

23.   

24.     pid = fork();                   // 子进程同样持有管道的读和写端  

25.     **if** (pid == -1) {  

26.         sys_err("fork error");  

27.     }  

28.     **else** **if** (pid > 0) {           // 父进程 读, 关闭写端  

29.         close(fd[1]);  

30.         dup2(fd[0], STDIN_FILENO);  // 重定向 stdin 到 管道的 读端  

31.         execlp("wc", "wc", "-l", NULL);     // 执行 wc -l 程序  

32.         sys_err("exclp wc error");  

33.     }  

34.     **else** **if** (pid == 0) {  

35.         close(fd[0]);  

36.         dup2(fd[1], STDOUT_FILENO);     // 重定向 stdout 到 管道写端  

37.         execlp("ls", "ls", NULL);       // 子进程执行 ls 命令  

38.         sys_err("exclp ls error");  

39.     }  

40.   

41.     **return** 0;  

42. }  

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image336.jpg)

直接执行命令，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image338.jpg)

其实代码和题目要求有出入，但是，问题不大，就这样吧。

## 108P-兄弟进程间通信

练习题：兄弟进程间通信

兄：ls

弟：wc -l

父：等待回收子进程

要求，使用循环创建N个子进程模型创建兄弟进程，使用循环因子i标识，注意管道读写行为

测试：

       是否允许，一个pipe有一个写端多个读端     可

       是否允许，一个pipe有多个写端一个读端     可

课后作业，统计当前系统中进程ID大于10000的进程个数

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image340.jpg)

代码如下：


```
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <sys/wait.h>  

5.  #include <unistd.h>  

6.  #include <errno.h>  

7.  #include <pthread.h>  

8.    

9.  **void** sys_err(**const** **char** *str)  

10. {  

11.     perror(str);  

12.     exit(1);  

13. }  

14. **int** main(**int** argc, **char** *argv[])  

15. {  

16.     **int** fd[2];  

17.     **int** ret, i;  

18.     pid_t pid;  

19.   

20.     ret = pipe(fd);  

21.     **if** (ret == -1) {  

22.         sys_err("pipe error");  

23.     }  

24.   

25.     **for**(i = 0; i < 2; i++) {        // 表达式2 出口,仅限父进程使用  

26.         pid = fork();  

27.         **if** (pid == -1) {  

28.             sys_err("fork error");  

29.         }   

30.         **if** (pid == 0)               // 子进程,出口  

31.             **break**;  

32.     }   

33.   

34.    **if** (i == 2) {                    // 父进程 . 不参与管道使用.   

35.         close(fd[0]);               // 关闭管道的 读端/写端.  

36.         close(fd[1]);  

37.   

38.         wait(NULL);                 // 回收子进程  

39.         wait(NULL);  

40.     } **else** **if** (i == 0) {  // xiong  

41.         close(fd[0]);  

42.         dup2(fd[1], STDOUT_FILENO);     // 重定向stdout  

43.         execlp("ls", "ls", NULL);  

44.         sys_err("exclp ls error");  

45.     } **else** **if** (i == 1) {            //弟弟  

46.         close(fd[1]);  

47.         dup2(fd[0], STDIN_FILENO);      // 重定向 stdin  

48.         execlp("wc", "wc", "-l", NULL);  

49.         sys_err("exclp wc error");  

50.     }  

51.       

52.     **return** 0;  

53. }  
```


编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image341.png)

这个代码需要注意一点，父进程不使用管道，所以一定要关闭父进程的管道，保证数据单向流动。

## 109P-多个读写端操作管道和管道缓冲区大小

下面是一个父进程读，俩子进程写的例子，也就是一个读端多个写端。需要调控写入顺序才行。

1.
```
  #include <stdio.h>  

2.  #include <unistd.h>  

3.  #include <sys/wait.h>  

4.  #include <string.h>  

5.  #include <stdlib.h>  

6.    

7.  **int** main(**void**)  

8.  {  

9.      pid_t pid;  

10.     **int** fd[2], i, n;  

11.     **char** buf[1024];  

12.   

13.     **int** ret = pipe(fd);  

14.     **if**(ret == -1){  

15.         perror("pipe error");  

16.         exit(1);  

17.     }  

18.   

19.     **for**(i = 0; i < 2; i++){  

20.         **if**((pid = fork()) == 0)  

21.             **break**;  

22.         **else** **if**(pid == -1){  

23.             perror("pipe error");  

24.             exit(1);  

25.         }  

26.     }  

27.   

28.     **if** (i == 0) {             

29.         close(fd[0]);                 

30.         write(fd[1], "1.hello\n", strlen("1.hello\n"));  

31.     } **else** **if**(i == 1) {   

32.         close(fd[0]);                 

33.         write(fd[1], "2.world\n", strlen("2.world\n"));  

34.     } **else** {  

35.         close(fd[1]);       //父进程关闭写端,留读端读取数据      

36.         sleep(1);  

37.         n = read(fd[0], buf, 1024);     //从管道中读数据  

38.         write(STDOUT_FILENO, buf, n);  

39.   

40.         **for**(i = 0; i < 2; i++)       //两个儿子wait两次  

41.             wait(NULL);  

42.     }  

43.   

44.     **return** 0;  

45. }  
```


编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image343.jpg)

这个例子需要注意，父进程必须等一下，不然可能俩子进程只写了一个，父进程就读完跑路了。

管道大小，默认4096

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image344.png)

## 命名管道fifo的创建和原理图

管道的优劣：

```

    优点：简单，相比信号，套接字实现进程通信，简单很多

    缺点：1.只能单向通信，双向通信需建立两个管道

          2.只能用于有血缘关系的进程间通信。该问题后来使用fifo命名管道解决。
```


![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image346.jpg)

fifo管道：可以用于无血缘关系的进程间通信。

```

    命名管道：  mkfifo

    无血缘关系进程间通信：

       读端，open fifo O_RDONLY

       写端，open fifo O_WRONLY
```


fifo操作起来像文件

下面的代码创建一个fifo：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image347.png)

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image349.jpg)

如图，管道就通过程序创建出来了。

## fifo实现非血缘关系进程间通信

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image355.jpg)

下面测试多个写管道，一个读管道，就是多开两个fifo.w，就一个fifo.r，这是可以的，懒得做了，就这样吧。

    测试一个写端多个读端的时候，由于数据一旦被读走就没了，所以多个读端的并集才是写端的写入数据。

## 文件用于进程间通信

文件实现进程间通信：

    打开的文件是内核中的一块缓冲区。多个无血缘关系的进程，可以同时访问该文件。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image357.jpg)

文件通信这个，有没有血缘关系都行，只是有血缘关系的进程对于同一个文件，使用的同一个文件描述符，没有血缘关系的进程，对同一个文件使用的文件描述符可能不同。这些都不是问题，打开的是同一个文件就行。

## mmap函数原型

存储映射I/O(Memory-mapped I/O) 使一个磁盘文件与存储空间中的一个缓冲区相映射。于是从缓冲区中取数据，就相当于读文件中的相应字节。与此类似，将数据存入缓冲区，则相应的字节就自动写入文件。这样，就可在不使用read和write函数的情况下，使地址指针完成I/O操作。

    使用这种方法，首先应该通知内核，将一个指定文件映射到存储区域中。这个映射工作可以通过mmap函数来实现。


```
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);       创建共享内存映射

    参数：

       addr：     指定映射区的首地址。通常传NULL，表示让系统自动分配

       length：共享内存映射区的大小。（<= 文件的实际大小）

       prot： 共享内存映射区的读写属性。PROT_READ、PROT_WRITE、PROT_READ|PROT_WRITE

       flags：    标注共享内存的共享属性。MAP_SHARED、MAP_PRIVATE

       fd: 用于创建共享内存映射区的那个文件的 文件描述符。

       offset：默认0，表示映射文件全部。偏移位置。需是 4k 的整数倍。

    返回值：

       成功：映射区的首地址。

       失败：MAP_FAILED (void*(-1))， errno

flags里面的shared意思是修改会反映到磁盘上

           private表示修改不反映到磁盘上

int munmap(void *addr, size_t length);    释放映射区。

    addr：mmap 的返回值

    length：大小
```



![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image361.jpg)

## mmap使用注意事项1

使用注意事项：

  
```
  1. 用于创建映射区的文件大小为 0，实际指定非0大小创建映射区，出 “总线错误”。

    2. 用于创建映射区的文件大小为 0，实际制定0大小创建映射区， 出 “无效参数”。

    3. 用于创建映射区的文件读写属性为，只读。映射区属性为 读、写。 出 “无效参数”。

    4. 创建映射区，需要read权限。当访问权限指定为 “共享”MAP_SHARED时， mmap的读写权限，应该 <=文件的open权限。    只写不行。

    5. 文件描述符fd，在mmap创建映射区完成即可关闭。后续访问文件，用 地址访问。

    6. offset 必须是 4096的整数倍。（MMU 映射的最小单位 4k ）

    7. 对申请的映射区内存，不能越界访问。

    8. munmap用于释放的 地址，必须是mmap申请返回的地址。

    9. 映射区访问权限为 “私有”MAP_PRIVATE, 对内存所做的所有修改，只在内存有效，不会反应到物理磁盘上。

    10.  映射区访问权限为 “私有”MAP_PRIVATE, 只需要open文件时，有读权限，用于创建映射区即可。

```

## mmap使用注意事项2



mmap函数的保险调用方式：

    1. fd = open（"文件名"， O_RDWR）;

2.  mmap(NULL, 有效文件大小， PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);

## mmap总结

1.  创建映射区的过程中，隐含着一次对映射文件的读操作

2.  当MAP_SHARED时，要求：映射区的权限应该<=文件打开的权限（出于对映射区的保护）。而MAP_PRIVATE则无所谓，因为mmap中的权限是对内存的限制

3.  映射区的释放与文件关闭无关。只要映射建立成功，文件可以立即关闭

4.  特别注意，当映射文件大小为0时，不能创建映射区。所以：用于映射的文件必须要有实际大小！！mmap使用时常常会出现总线错误，通常是由于共享文件存储空间大小引起的。如，400字节大小的文件，在简历映射区时，offset4096字节，则会报出总线错误

5.  munmap传入的地址一定是mmap返回的地址。坚决杜绝指针++操作

6.  文件偏移量必须为4K的整数倍

7.  mmap创建映射区出错概率非常高，一定要检查返回值，确保映射区建立成功再进行后续操作。

## 父子进程间mmap通信

父子进程使用 mmap 进程间通信：

    父进程 先 创建映射区。 open（ O_RDWR） mmap( MAP_SHARED );

    指定 MAP_SHARED 权限

    fork() 创建子进程。

    一个进程读， 另外一个进程写。

下面这段代码，父子进程mmap通信，共享内存是一个int变量：


```
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <unistd.h>  

4.  #include <fcntl.h>  

5.  #include <sys/mman.h>  

6.  #include <sys/wait.h>  

7.    

8.  **int** var = 100;  

9.    

10. **int** main(**void**)  

11. {  

12.     **int** *p;  

13.     pid_t pid;  

14.   

15.     **int** fd;  

16.     fd = open("temp", O_RDWR|O_CREAT|O_TRUNC, 0644);  

17.     **if**(fd < 0){  

18.         perror("open error");  

19.         exit(1);  

20.     }  

21.     ftruncate(fd, 4);  

22.   

23.     p = (**int** *)mmap(NULL, 4, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);  

24.     //p = (int *)mmap(NULL, 4, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);  

25.     **if**(p == MAP_FAILED){        //注意:不是p == NULL  

26.         perror("mmap error");  

27.         exit(1);  

28.     }  

29.     close(fd);                  //映射区建立完毕,即可关闭文件  

30.   

31.     pid = fork();               //创建子进程  

32.     **if**(pid == 0){  

33.        *p = 7000;               // 写共享内存  

34.         var = 1000;  

35.         printf("child, *p = %d, var = %d\n", *p, var);  

36.     } **else** {  

37.         sleep(1);  

38.         printf("parent, *p = %d, var = %d\n", *p, var);     // 读共享内存  

39.         wait(NULL);  

40.   

41.         **int** ret = munmap(p, 4);             //释放映射区  

42.         **if** (ret == -1) {  

43.             perror("munmap error");  

44.             exit(1);  

45.         }  

46.     }  

47.   

48.     **return** 0;  

49. }  
```


编译运行，如下所示：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image362.png)

如图，子进程修改p的值，也反映到了父进程上，这是因为共享内存定义为shared的。

如果将共享内存定义为private，运行结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image364.jpg)

## 无血缘关系进程间mmap通信

无血缘关系进程间 mmap 通信：              【会写】

    两个进程 打开同一个文件，创建映射区。

    指定flags 为 MAP_SHARED。

    一个进程写入，另外一个进程读出。

    【注意】：无血缘关系进程间通信。mmap：数据可以重复读取。

                  fifo：数据只能一次读取。

下面是两个无血缘关系的通信代码，先是写进程：


```
1.  #include <stdio.h>  

2.  #include <sys/stat.h>  

3.  #include <sys/types.h>  

4.  #include <fcntl.h>  

5.  #include <unistd.h>  

6.  #include <stdlib.h>  

7.  #include <sys/mman.h>  

8.  #include <string.h>  

9.    

10. **struct** STU {  

11.     **int** id;  

12.     **char** name[20];  

13.     **char** sex;  

14. };  

15.   

16. **void** sys_err(**char** *str)  

17. {  

18.     perror(str);  

19.     exit(1);  

20. }  

21.   

22. **int** main(**int** argc, **char** *argv[])  

23. {  

24.     **int** fd;  

25.     **struct** STU student = {10, "xiaoming", 'm'};  

26.     **char** *mm;  

27.   

28.     **if** (argc < 2) {  

29.         printf("./a.out file_shared\n");  

30.         exit(-1);  

31.     }  

32.   

33.     fd = open(argv[1], O_RDWR | O_CREAT, 0664);  

34.     ftruncate(fd, **sizeof**(student));  

35.   

36.     mm = mmap(NULL, **sizeof**(student), PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);  

37.     **if** (mm == MAP_FAILED)  

38.         sys_err("mmap");  

39.   

40.     close(fd);  

41.   

42.     **while** (1) {  

43.         memcpy(mm, &student, **sizeof**(student));  

44.         student.id++;  

45.         sleep(1);  

46.     }  

47.   

48.     munmap(mm, **sizeof**(student));  

49.   

50.     **return** 0;  

51. }  

```

然后是读进程：

1. 
```
 #include <stdio.h>  

2.  #include <sys/stat.h>  

3.  #include <fcntl.h>  

4.  #include <unistd.h>  

5.  #include <stdlib.h>  

6.  #include <sys/mman.h>  

7.  #include <string.h>  

8.    

9.  **struct** STU {  

10.     **int** id;  

11.     **char** name[20];  

12.     **char** sex;  

13. };  

14.   

15. **void** sys_err(**char** *str)  

16. {  

17.     perror(str);  

18.     exit(-1);  

19. }  

20.   

21. **int** main(**int** argc, **char** *argv[])  

22. {  

23.     **int** fd;  

24.     **struct** STU student;  

25.     **struct** STU *mm;  

26.   

27.     **if** (argc < 2) {  

28.         printf("./a.out file_shared\n");  

29.         exit(-1);  

30.     }  

31.   

32.     fd = open(argv[1], O_RDONLY);  

33.     **if** (fd == -1)  

34.         sys_err("open error");  

35.   

36.     mm = mmap(NULL, **sizeof**(student), PROT_READ, MAP_SHARED, fd, 0);  

37.     **if** (mm == MAP_FAILED)  

38.         sys_err("mmap error");  

39.       

40.     close(fd);  

41.   

42.     **while** (1) {  

43.         printf("id=%d\tname=%s\t%c\n", mm->id, mm->name, mm->sex);  

44.         sleep(2);  

45.     }  

46.     munmap(mm, **sizeof**(student));  

47.   

48.     **return** 0;  

49. }  

```

编译并运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image366.jpg)

如图，一读一写，问题不大。

多个写端一个读端也没问题，打开多个写进程即可，完事儿读进程会读到所有写进程写入的内容。

这里要注意一个，内容被读走之后不会消失，所以如果读进程的读取时间间隔短，它会读到很多重复内容，就是因为写进程没来得及写入新内容。

## mmap总结

无血缘关系进程间 mmap 通信：              【会写】

    两个进程 打开同一个文件，创建映射区。

    指定flags 为 MAP_SHARED。

    一个进程写入，另外一个进程读出。

    【注意】：无血缘关系进程间通信。mmap：数据可以重复读取。

                  fifo：数据只能一次读取。

## 匿名映射区

匿名映射：只能用于 血缘关系进程间通信。

    p = (int *)mmap(NULL, 40, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0);

## 总结

pipe管道： 用于有血缘关系的进程间通信。  ps aux | grep     ls | wc -l   

    父子进程间通信：

    兄弟进程间通信：

fifo管道：可以用于无血缘关系的进程间通信。

    命名管道：  mkfifo

    无血缘关系进程间通信：

       读端，open fifo O_RDONLY

       写端，open fifo O_WRONLY

文件实现进程间通信：

    打开的文件是内核中的一块缓冲区。多个无血缘关系的进程，可以同时访问该文件。

共享内存映射:

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);       创建共享内存映射

    参数：

       addr：     指定映射区的首地址。通常传NULL，表示让系统自动分配

       length：共享内存映射区的大小。（<= 文件的实际大小）

       prot： 共享内存映射区的读写属性。PROT_READ、PROT_WRITE、PROT_READ|PROT_WRITE

       flags：    标注共享内存的共享属性。MAP_SHARED、MAP_PRIVATE

       fd: 用于创建共享内存映射区的那个文件的 文件描述符。

       offset：默认0，表示映射文件全部。偏移位置。需是 4k 的整数倍。

    返回值：

       成功：映射区的首地址。

       失败：MAP_FAILED (void*(-1))， errno

int munmap(void *addr, size_t length);    释放映射区。

    addr：mmap 的返回值

    length：大小

使用注意事项：

    1. 用于创建映射区的文件大小为 0，实际指定非0大小创建映射区，出 “总线错误”。

    2. 用于创建映射区的文件大小为 0，实际制定0大小创建映射区， 出 “无效参数”。

    3. 用于创建映射区的文件读写属性为，只读。映射区属性为 读、写。 出 “无效参数”。

    4. 创建映射区，需要read权限。当访问权限指定为 “共享”MAP_SHARED是， mmap的读写权限，应该 <=文件的open权限。    只写不行。

    5. 文件描述符fd，在mmap创建映射区完成即可关闭。后续访问文件，用 地址访问。

    6. offset 必须是 4096的整数倍。（MMU 映射的最小单位 4k ）

    7. 对申请的映射区内存，不能越界访问。

    8. munmap用于释放的 地址，必须是mmap申请返回的地址。

    9. 映射区访问权限为 “私有”MAP_PRIVATE, 对内存所做的所有修改，只在内存有效，不会反应到物理磁盘上。

    10.  映射区访问权限为 “私有”MAP_PRIVATE, 只需要open文件时，有读权限，用于创建映射区即可。

mmap函数的保险调用方式：

    1. fd = open（"文件名"， O_RDWR）;

    2. mmap(NULL, 有效文件大小， PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);

父子进程使用 mmap 进程间通信：

    父进程 先 创建映射区。 open（ O_RDWR） mmap( MAP_SHARED );

    指定 MAP_SHARED 权限

    fork() 创建子进程。

    一个进程读， 另外一个进程写。

无血缘关系进程间 mmap 通信：              【会写】

    两个进程 打开同一个文件，创建映射区。

    指定flags 为 MAP_SHARED。

    一个进程写入，另外一个进程读出。

    【注意】：无血缘关系进程间通信。mmap：数据可以重复读取。

                  fifo：数据只能一次读取。

匿名映射：只能用于 血缘关系进程间通信。

    p = (int *)mmap(NULL, 40, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0);


# 信号

## 信号的概念和机制

信号共性：

    简单、不能携带大量信息、满足条件才发送。

信号的特质：

    信号是软件层面上的“中断”。一旦信号产生，无论程序执行到什么位置，必须立即停止运行，处理信号，处理结束，再继续执行后续指令。

    所有信号的产生及处理全部都是由【内核】完成的。

## 与信号相关的概念

信号相关的概念：

    产生信号：

       1. 按键产生

       2. 系统调用产生

       3. 软件条件产生

       4. 硬件异常产生

       5. 命令产生

    概念：

       未决：产生与递达之间状态。 

       递达：产生并且送达到进程。直接被内核处理掉。

       信号处理方式： 执行默认处理动作、忽略、捕捉（自定义）

       阻塞信号集（信号屏蔽字）： 本质：位图。用来记录信号的屏蔽状态。一旦被屏蔽的信号，在解除屏蔽前，一直处于未决态。

       未决信号集：本质：位图。用来记录信号的处理状态。该信号集中的信号，表示，已经产生，但尚未被处理。

## 信号屏蔽字和未决信号集

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image368.jpg)

概念：

       未决：产生与递达之间状态。 

       递达：产生并且送达到进程。直接被内核处理掉。

       信号处理方式： 执行默认处理动作、忽略、捕捉（自定义）

       阻塞信号集（信号屏蔽字）： 本质：位图。用来记录信号的屏蔽状态。一旦被屏蔽的信号，在解除屏蔽前，一直处于未决态。

       未决信号集：本质：位图。用来记录信号的处理状态。该信号集中的信号，表示，已经产生，但尚未被处理。

## 信号四要素和常规信号一览

kill -l 查看当前系统中常规信号

信号4要素：

    信号使用之前，应先确定其4要素，而后再用！！！

    编号、名称、对应事件、默认处理动作。

## kill函数和kill命令

kill命令 和 kill函数：

    int kill（pid_t pid, int signum）

    参数：

       pid:   > 0:发送信号给指定进程

           = 0：发送信号给跟调用kill函数的那个进程处于同一进程组的进程。

           < -1: 取绝对值，发送信号给该绝对值所对应的进程组的所有组员。

           = -1：发送信号给，有权限发送的所有进程。

       signum：待发送的信号

    返回值：

       成功： 0

       失败： -1 errno

小例子，子进程发送信号kill父进程：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image370.jpg)

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image372.jpg)

这里子进程不发送kill信号，发其他信号也行，比如段错误什么的。

kill -9 -groupname  杀一个进程组

## alarm函数

alarm 函数：使用自然计时法。

    定时发送SIGALRM给当前进程。

    unsigned int alarm(unsigned int seconds);

       seconds：定时秒数

       返回值：上次定时剩余时间。

           无错误现象。

       alarm（0）； 取消闹钟。

    time 命令 ： 查看程序执行时间。   实际时间 = 用户时间 + 内核时间 + 等待时间。  --》 优化瓶颈 IO

小例子就不写了，使用alarm函数计时，打印变量i的值。

## setitimer函数

setitimer函数：

    int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

    参数：

       which：    ITIMER_REAL： 采用自然计时。 ——> SIGALRM

           ITIMER_VIRTUAL: 采用用户空间计时  ---> SIGVTALRM

           ITIMER_PROF: 采用内核+用户空间计时 ---> SIGPROF

       new_value：定时秒数

                  类型：struct itimerval {

                            struct timeval {

                                time_t      tv_sec;         /* seconds */

                                suseconds_t tv_usec;        /* microseconds */

                         }it_interval;---> 周期定时秒数

                             struct timeval {

                                time_t      tv_sec;        

                                suseconds_t tv_usec;       

                         }it_value;  ---> 第一次定时秒数 

                      };

       old_value：传出参数，上次定时剩余时间。

       e.g.

           struct itimerval new_t; 

           struct itimerval old_t; 

           new_t.it_interval.tv_sec = 0;

           new_t.it_interval.tv_usec = 0;

           new_t.it_value.tv_sec = 1;

           new_t.it_value.tv_usec = 0;

           int ret = setitimer(&new_t, &old_t);  定时1秒

    返回值：

       成功： 0

       失败： -1 errno

小例子，使用setitimer定时，向屏幕打印信息：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image374.jpg)

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image376.jpg)

第一次信息打印是两秒间隔，之后都是5秒间隔打印一次。可以理解为第一次是有个定时器，什么时候触发打印，之后就是间隔时间。

## 信号集操作函数

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image378.png)

信号集操作函数：

 
``` c
   sigset_t set;  自定义信号集。

    sigemptyset(sigset_t *set); 清空信号集

    sigfillset(sigset_t *set);  全部置1

    sigaddset(sigset_t *set, int signum);  将一个信号添加到集合中

    sigdelset(sigset_t *set, int signum);  将一个信号从集合中移除

    sigismember（const sigset_t *set，int signum); 判断一个信号是否在集合中。 在--》1， 不在--》0
```


设置信号屏蔽字和解除屏蔽：


``` c
    int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

       how:   SIG_BLOCK: 设置阻塞

           SIG_UNBLOCK:  取消阻塞

           SIG_SETMASK:  用自定义set替换mask。

       set：  自定义set

       oldset：旧有的 mask。

```

查看未决信号集：

   
``` c
 int sigpending(sigset_t *set);

       set： 传出的 未决信号集。
```



![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image379.png)

## 信号集操作函数练习

信号列表：


其中9号和19号信号比较特殊，只能执行默认动作，不能忽略捕捉，不能设置阻塞。

下面这个小例子，利用自定义集合，来设置信号阻塞，我们输入被设置阻塞的信号，可以看到未决信号集发生变化：


```
1.  #include <stdio.h>  

2.  #include <signal.h>  

3.  #include <stdlib.h>  

4.  #include <string.h>  

5.  #include <unistd.h>  

6.  #include <errno.h>  

7.  #include <pthread.h>  

8.    

9.  **void** sys_err(**const** **char** *str)  

10. {  

11.     perror(str);  

12.     exit(1);  

13. }  

14.   

15. **void** print_set(sigset_t *set)  

16. {  

17.     **int** i;  

18.     **for** (i = 1; i<32; i++) {  

19.         **if** (sigismember(set, i))   

20.             putchar('1');  

21.         **else**   

22.             putchar('0');  

23.     }  

24.     printf("\n");  

25. }  

26. **int** main(**int** argc, **char** *argv[])  

27. {  

28.     sigset_t set, oldset, pedset;  

29.     **int** ret = 0;  

30.   

31.     sigemptyset(&set);  

32.     sigaddset(&set, SIGINT);  

33.     sigaddset(&set, SIGQUIT);  

34.     sigaddset(&set, SIGBUS);  

35.     sigaddset(&set, SIGKILL);  

36.   

37.     ret = sigprocmask(SIG_BLOCK, &set, &oldset);  

38.     **if** (ret == -1)  

39.         sys_err("sigprocmask error");  

40.   

41.     **while** (1) {  

42.         ret = sigpending(&pedset);  

43.         print_set(&pedset);  

44.         sleep(1);  

45.     }  

46.   

47.     **return** 0;  

48. }  

```

编译运行，如下图所示：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image382.png)

可以看到，在输入Ctrl+C之后，进程捕捉到信号，但由于设置阻塞，没有处理，未决信号集对应位置变为1.

## signal实现信号捕捉



参数：

signum ：待捕捉信号

handler：捕捉信号后的操纵函数

返回值：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image385.jpg)

信号捕捉特性：

    1. 捕捉函数执行期间，信号屏蔽字 由 mask --> sa_mask , 捕捉函数执行结束。 恢复回mask

    2. 捕捉函数执行期间，本信号自动被屏蔽(sa_flgs = 0).

    3. 捕捉函数执行期间，被屏蔽信号多次发送，解除屏蔽后只处理一次！

## sigaction实现信号捕捉

sigaction也是注册一个信号捕捉函数

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image391.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image393.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image395.jpg)

下面的小例子，使用sigaction捕捉两个信号：


```
1.  #include <stdio.h>  

2.  #include <signal.h>  

3.  #include <stdlib.h>  

4.  #include <string.h>  

5.  #include <unistd.h>  

6.  #include <errno.h>  

7.  #include <pthread.h>  

8.    

9.  **void** sys_err(**const** **char** *str)  

10. {  

11.     perror(str);  

12.     exit(1);  

13. }  

14.   

15. **void** sig_catch(**int** signo)                   // 回调函数  

16. {  

17.     **if** (signo == SIGINT) {  

18.         printf("catch you!! %d\n", signo);  

19.         sleep(10);  

20.     }  

21.       

22.     **else** **if** (signo == SIGQUIT)  

23.         printf("-----------catch you!! %d\n", signo);  

24.       

25.     **return** ;  

26. }  

27.   

28. **int** main(**int** argc, **char** *argv[])  

29. {  

30.     **struct** sigaction act, oldact;  

31.   

32.     act.sa_handler = sig_catch;         // set callback function name       设置回调函数  

33.     sigemptyset(&(act.sa_mask));        // set mask when sig_catch working. 清空sa_mask屏蔽字, 只在sig_catch工作时有效  

34.     //sigaddset(&act.sa_mask, SIGQUIT);  

35.     act.sa_flags = 0;                   // usually use.                     默认值  

36.       

37.     **int** ret = sigaction(SIGINT, &act, &oldact);     //注册信号捕捉函数  

38.     **if** (ret == -1)  

39.         sys_err("sigaction error");  

40.     ret = sigaction(SIGQUIT, &act, &oldact);     //注册信号捕捉函数  

41.   

42.     **while** (1);  

43.   

44.     **return** 0;  

45. }  
```


编译运行，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image397.jpg)

如图，两个信号都捕捉到了，并且输出了对应字符串。

## 信号捕捉的特性

信号捕捉特性：

    1. 捕捉函数执行期间，信号屏蔽字 由 mask --> sa_mask , 捕捉函数执行结束。 恢复回mask

    2. 捕捉函数执行期间，本信号自动被屏蔽(sa_flgs = 0).

3.  捕捉函数执行期间，被屏蔽信号多次发送，解除屏蔽后只处理一次！



。

## 慢速系统调用中断

慢速系统调用：

    可能会使进程永久阻塞的一类。如果在阻塞期间收到一个信号，该系统调用就被中断，不再继续执行(早期)，也可以设定系统调用是否重启。如read, write, pause…

## 总结

信号共性：

    简单、不能携带大量信息、满足条件才发送。

信号的特质：

    信号是软件层面上的“中断”。一旦信号产生，无论程序执行到什么位置，必须立即停止运行，处理信号，处理结束，再继续执行后续指令。

    所有信号的产生及处理全部都是由【内核】完成的。

信号相关的概念：

    产生信号：

       1. 按键产生

       2. 系统调用产生

       3. 软件条件产生

       4. 硬件异常产生

       5. 命令产生

    概念：

       未决：产生与递达之间状态。 

       递达：产生并且送达到进程。直接被内核处理掉。

       信号处理方式： 执行默认处理动作、忽略、捕捉（自定义）

       阻塞信号集（信号屏蔽字）： 本质：位图。用来记录信号的屏蔽状态。一旦被屏蔽的信号，在解除屏蔽前，一直处于未决态。

       未决信号集：本质：位图。用来记录信号的处理状态。该信号集中的信号，表示，已经产生，但尚未被处理。

信号4要素：

    信号使用之前，应先确定其4要素，而后再用！！！

    编号、名称、对应事件、默认处理动作。

kill命令 和 kill函数：

    int kill（pid_t pid, int signum）

    参数：

       pid:   > 0:发送信号给指定进程

           = 0：发送信号给跟调用kill函数的那个进程处于同一进程组的进程。

           < -1: 取绝对值，发送信号给该绝对值所对应的进程组的所有组员。

           = -1：发送信号给，有权限发送的所有进程。

       signum：待发送的信号

    返回值：

       成功： 0

       失败： -1 errno

alarm 函数：使用自然计时法。

    定时发送SIGALRM给当前进程。

    unsigned int alarm(unsigned int seconds);

       seconds：定时秒数

       返回值：上次定时剩余时间。

           无错误现象。

       alarm（0）； 取消闹钟。

    time 命令 ： 查看程序执行时间。   实际时间 = 用户时间 + 内核时间 + 等待时间。  --》 优化瓶颈 IO

setitimer函数：

    int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

    参数：

       which：    ITIMER_REAL： 采用自然计时。 ——> SIGALRM

           ITIMER_VIRTUAL: 采用用户空间计时  ---> SIGVTALRM

           ITIMER_PROF: 采用内核+用户空间计时 ---> SIGPROF

       new_value：定时秒数

                  类型：struct itimerval {

                            struct timeval {

                                time_t      tv_sec;         /* seconds */

                                suseconds_t tv_usec;        /* microseconds */

                         }it_interval;---> 周期定时秒数

                             struct timeval {

                                time_t      tv_sec;        

                                suseconds_t tv_usec;       

                         }it_value;  ---> 第一次定时秒数 

                      };

       old_value：传出参数，上次定时剩余时间。

       e.g.

           struct itimerval new_t; 

           struct itimerval old_t; 

           new_t.it_interval.tv_sec = 0;

           new_t.it_interval.tv_usec = 0;

           new_t.it_value.tv_sec = 1;

           new_t.it_value.tv_usec = 0;

           int ret = setitimer(&new_t, &old_t);  定时1秒

    返回值：

       成功： 0

       失败： -1 errno

其他几个发信号函数：

    int raise(int sig);

    void abort(void);

信号集操作函数：

    sigset_t set;  自定义信号集。

    sigemptyset(sigset_t *set); 清空信号集

    sigfillset(sigset_t *set);  全部置1

    sigaddset(sigset_t *set, int signum);  将一个信号添加到集合中

    sigdelset(sigset_t *set, int signum);  将一个信号从集合中移除

    sigismember（const sigset_t *set，int signum); 判断一个信号是否在集合中。 在--》1， 不在--》0

设置信号屏蔽字和解除屏蔽：

    int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

       how:   SIG_BLOCK: 设置阻塞

           SIG_UNBLOCK:  取消阻塞

           SIG_SETMASK:  用自定义set替换mask。

       set：  自定义set

       oldset：旧有的 mask。

查看未决信号集：

    int sigpending(sigset_t *set);

       set： 传出的 未决信号集。

【信号捕捉】：

    signal();

    【sigaction();】 重点！！！

信号捕捉特性：

    1. 捕捉函数执行期间，信号屏蔽字 由 mask --> sa_mask , 捕捉函数执行结束。 恢复回mask

    2. 捕捉函数执行期间，本信号自动被屏蔽(sa_flgs = 0).

    3. 捕捉函数执行期间，被屏蔽信号多次发送，解除屏蔽后只处理一次！

借助信号完成 子进程回收。

## 会话

会话：多个进程组的集合

创建会话的6点注意事项：

1.  调用进程不能是进程组组长，该进程变成新会话首进程

2.  该进程成为一个新进程组的组长进程

3.  需要root权限（ubuntu不需要）

4.  新会话丢弃原有的控制终端，该会话没有控制终端

5.  该调用进程是组长进程，则出错返回

6.  建立新会话时，先调用fork，父进程终止，子进程调用setsid

getsid函数：

pid_t getsid(pid_t pid)    获取当前进程的会话id

成功返回调用进程会话ID，失败返回-1，设置error

setsid函数：

pid_t setsid(void)    创建一个会话，并以自己的ID设置进程组ID，同时也是新会话的ID

成功返回调用进程的会话ID，失败返回-1，设置error

## 守护进程创建步骤分析

守护进程：

    daemon进程。通常运行于操作系统后台，**脱离控制终端**。一般不与用户直接交互。周期性的等待某个事件发生或周期性执行某一动作。

    不受用户登录注销影响。通常采用以d结尾的命名方式。

    创建守护进程，最关键的一步是调用setsid函数创建一个新的Session，并成为Session Leader

守护进程创建步骤：

    1. fork子进程，让父进程终止。

    2. 子进程调用 setsid() 创建新会话

    3. 通常根据需要，改变工作目录位置 chdir()， 防止目录被卸载。

    4. 通常根据需要，重设umask文件权限掩码，影响新文件的创建权限。  022 -- 755 0345 --- 432   r---wx-w-   422

    5. 通常根据需要，关闭/重定向 文件描述符

    6. 守护进程 业务逻辑。while（）

## 守护进程创建

下面这个例子，创建一个守护进程：

```

1.  #include <stdio.h>  

2.  #include <sys/stat.h>  

3.  #include <fcntl.h>  

4.  #include <stdlib.h>  

5.  #include <string.h>  

6.  #include <unistd.h>  

7.  #include <errno.h>  

8.  #include <pthread.h>  

9.    

10. **void** sys_err(**const** **char** *str)  

11. {  

12.     perror(str);  

13.     exit(1);  

14. }  

15.   

16. **int** main(**int** argc, **char** *argv[])  

17. {  

18.     pid_t pid;  

19.     **int** ret, fd;  

20.   

21.     pid = fork();  

22.     **if** (pid > 0)                // 父进程终止  

23.         exit(0);  

24.   

25.     pid = setsid();           //创建新会话  

26.     **if** (pid == -1)  

27.         sys_err("setsid error");  

28.   

29.     ret = chdir("/home/zhcode/Code/code146");       // 改变工作目录位置  

30.     **if** (ret == -1)  

31.         sys_err("chdir error");  

32.   

33.     umask(0022);            // 改变文件访问权限掩码  

34.   

35.     close(STDIN_FILENO);    // 关闭文件描述符 0  

36.   

37.     fd = open("/dev/null", O_RDWR);  //  fd --> 0  

38.     **if** (fd == -1)  

39.         sys_err("open error");  

40.   

41.     dup2(fd, STDOUT_FILENO); // 重定向 stdout和stderr  

42.     dup2(fd, STDERR_FILENO);  

43.   

44.     **while** (1);              // 模拟 守护进程业务.  

45.   

46.     **return** 0;  

47. }  
```


编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image406.jpg)

查看进程列表，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image408.jpg)

这个daemon进程就不会受到用户登录注销影响。

要想终止，就必须用kill命令

# 线程
## 线程概念

线程概念：

    进程：有独立的 进程地址空间。有独立的pcb。   分配资源的最小单位。

    线程：有独立的pcb。没有独立的进程地址空间。  最小单位的执行。

    ps -Lf 进程id    ---> 线程号。LWP  --》cpu 执行的最小单位。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image410.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image412.jpg)

ps -Lf 进程号     查看进程的线程

![](Pasted%20image%2020221012213308.png)
clone复制了资源就是创建了子进程 如果共享了资源就是创建了线程



## 线程共享和非共享
1.1 线程共享资源

1）文件描述符表。由于线程间共享进程间的内容，而文件描述符表在主线程的PCB当中，各个线程可以直接去请求访问，所以线程间通信就不需要像进程那样通过管道这些方式通信。
2）每种信号的处理方式。即当某个信号发送到该进程时，该进程有多个线程，那么哪个线程捕捉到就谁处理。但mask是不共享的，所以可以通过这个指定某个线程接收。但不建议信号与线程混用，难度很大。
3）当前工作目录。物理内存地址相同，所以工作目录相同。因为工作目录的物理地址是唯一的。只有物理内存地址相同才能保证工作目录相同。
4）用户ID和组ID。很简单，因为都在0-3G的用户空间。
5）内存地址空间 (.text/.data(全局区)/.bss(静态区)/heap/共享库)。(线程间栈独立，堆共享)
1.2 线程非共享资源

1）线程id。
2）处理器现场和栈指针(内核栈)。(上面说了，线程间栈独立，堆共享)
3）独立的栈空间(用户空间栈)。
4）errno变量。注：errno是全局变量，本来应该是共享的，但是就这么巧，它是唯一一个全局变量不共享的。
5）信号屏蔽字。不共享因为需要它去拦截对应信号，实现某个信号给指定线程处理。
6）调度优先级。


## 创建线程


```
pthread_t pthread_self(void);   获取线程id。 线程id是在进程地址空间内部，用来标识线程身份的id号。

       返回值：本线程id

    检查出错返回：  线程中。

       fprintf(stderr, "xxx error: %s\n", strerror(ret));
```



```
int pthread_create(pthread_t *tid, const pthread_attr_t *attr, void *(*start_rountn)(void *), void *arg); 创建子线程。

       参1：传出参数，表新创建的子线程 id

       参2：线程属性。传NULL表使用默认属性。

       参3：子线程回调函数。创建成功，ptherad_create函数返回时，该函数会被自动调用。

       参4：参3的参数。没有的话，传NULL

       返回值：成功：0

           失败：errno
```


下面这个例子，创建一个子线程去执行其他任务：


```c
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.  **void** sys_err(**const** **char** *str){  

9.      perror(str);  

10.     exit(1);  

11. }  

12.   

13. **void** *tfn(**void** *arg){  

14.     printf("thread: pid = %d, tid = %lu\n", getpid(), pthread_self());  

15.   

16.     **return** NULL;  

17. }  

18.   

19. **int** main(**int** argc, **char** *argv[]){  

20.     pthread_t tid;  

21.       

22.     printf("main: pid = %d, tid = %lu\n", getpid(), pthread_self());  

23.   

24.     **int** ret = pthread_create(&tid, NULL, tfn, NULL);  

25.     **if** (ret != 0) {  

26.         perror("pthread_create error");  

27.     }  

28.   

29.     **return** 0;  

30. }  
```


编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image418.jpg)

    可以看到，子线程的打印信息并未出现。原因在于，主线程执行完之后，就销毁了整个进程的地址空间，于是子线程就无法打印。简单粗暴的方法就是让主线程睡1秒，等子线程执行。

代码变化如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image420.jpg)

编译执行，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image422.jpg)



## 错误分析

在152P的代码中，如果将i取地址后再传入线程创建函数里，就是说

当前传的是：(void *)i

改成：       (void *)&i

相应的，修改回调函数：int i = *((int *)arg)

运行代码，会出现如下结果：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image426.jpg)

如果多次运行都只有主线程的输出，将主线程等待时长从i改为大于6的数即可。因为子线程等待时间i是不定的，但都小于等于6秒，由于抢cpu时没抢过主线程，导致没有子线程的输出。

错误原因在于，子线程如果用引用传递i，会去读取主线程里的i值，而主线程里的i是动态变化的，不固定。所以，应该采用值传递，不用引用传递。

## 线程间全局变量共享

在子线程里更改全局变量，主线程里也跟着发生变化。error除外

## pthread_exit退出

void pthread_exit(void *retval);  退出当前线程。

       retval：退出值。 无退出值时，NULL

       exit();    退出当前进程。

       return: 返回到调用者那里去。

       pthread_exit(): 退出当前线程。

如果在回调函数里加一段代码：

if(i == 2)

exit(0);

看起来好像是退出了第三个子线程，然而运行时，发现后续的4,5也没了。这是因为，exit是退出进程。

一、修改一下，换成：

if(i == 2)

return NULL;

这样运行一下，发现后续线程不会凉凉，说明return是可以达到退出线程的目的。然而真正意义上，return是返回到函数调用者那里去，线程并没有退出。

二、再修改一下，再定义一个函数func，直接返回那种

void *func(void){

    return NULL;

}

if(i == 2)

func();

运行，发现1,2,3,4,5线程都还在，说明没有达到退出目的。

三、再次修改：

void *func(void){

    pthread_exit(NULL);

    return NULL;

}

if(i == 2)

func();

编译运行，发现3没了，看起来很科学的样子。pthread_exit表示将当前线程退出。放在函数里，还是直接调用，都可以。

    回到之前说的一个问题，由于主线程可能先于子线程结束，所以子线程的输出可能不会打印出来，当时是用主线程sleep等待子线程结束来解决的。现在就可以使用pthread_exit来解决了。方法就是将return 0替换为pthread_exit，只退出当先线程，不会对其他线程造成影响。

void pthread_exit(void *retval);  退出当前线程。

       retval：退出值。 无退出值时，NULL

       exit();    退出当前进程。

       return: 返回到调用者那里去。

       pthread_exit(): 退出当前线程。

## pthread_join回收线程

int pthread_join(pthread_t thread, void **retval);   阻塞 回收线程。

       thread: 待回收的线程id

       retval：传出参数。 回收的那个线程的退出值。

           线程异常借助，值为 -1。

       返回值：成功：0

           失败：errno

下面这个是回收线程并获取子线程返回值的小例子：


```
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.  **struct** thrd {  

9.      **int** var;  

10.     **char** str[256];  

11. };  

12.   

13. **void** sys_err(**const** **char** *str)  

14. {  

15.     perror(str);  

16.     exit(1);  

17. }  

18.   

19. **void** *tfn(**void** *arg)  

20. {  

21.     **struct** thrd *tval;  

22.   

23.     tval = malloc(**sizeof**(tval));  

24.     tval->var = 100;  

25.     strcpy(tval->str, "hello thread");  

26.   

27.     **return** (**void** *)tval;  

28. }  

29.   

30. **int** main(**int** argc, **char** *argv[])  

31. {  

32.     pthread_t tid;  

33.   

34.     **struct** thrd *retval;  

35.   

36.     **int** ret = pthread_create(&tid, NULL, tfn, NULL);  

37.     **if** (ret != 0)  

38.         sys_err("pthread_create error");  

39.   

40.     //int pthread_join(pthread_t thread, void **retval);  

41.     ret = pthread_join(tid, (**void** **)&retval);  

42.     **if** (ret != 0)  

43.         sys_err("pthread_join error");  

44.   

45.     printf("child thread exit with var= %d, str= %s\n", retval->var, retval->str);  

46.       

47.     pthread_exit(NULL);  

48.   

49. }  

```



## pthread_cancel杀死线程

int pthread_cancel(pthread_t thread);     杀死一个线程。  需要到达取消点（保存点）

       thread: 待杀死的线程id

       返回值：成功：0

           失败：errno

       如果，子线程没有到达取消点， 那么 pthread_cancel 无效。

       我们可以在程序中，手动添加一个取消点。使用 pthread_testcancel();

       成功被 pthread_cancel() 杀死的线程，返回 -1.使用pthead_join 回收。

小例子，主线程调用pthread_cancel杀死子线程


```
1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.    

9.  **void** *tfn(**void** *arg){  

10.     **while** (1) {  

11.         printf("thread: pid = %d, tid = %lu\n", getpid(), pthread_self());  

12.         sleep(1);  

13.     }  

14.   

15.     **return** NULL;  

16. }  

17.   

18. **int** main(**int** argc, **char** *argv[]){  

19.     pthread_t tid;  

20.   

21.     **int** ret = pthread_create(&tid, NULL, tfn, NULL);  

22.     **if** (ret != 0) {  

23.         fprintf(stderr, "pthread_create error:%s\n", strerror(ret));  

24.         exit(1);  

25.     }  

26.   

27.     printf("main: pid = %d, tid = %lu\n", getpid(), pthread_self());  

28.   

29.     sleep(5);  

30.   

31.     ret = pthread_cancel(tid);          // 终止线程  

32.     **if** (ret != 0) {  

33.         fprintf(stderr, "pthread_cancel error:%s\n", strerror(ret));  

34.         exit(1);  

35.     }  

36.   

37.     **while** (1);  

38.   

39.     pthread_exit((**void** *)0);  

40. }  

```

编译运行，如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image434.jpg)

    可以看到，主线程确实kill了子线程。

这里要注意一点，pthread_cancel工作的必要条件是进入内核，如果tfn真的奇葩到没有进入内核，则pthread_cancel不能杀死线程，此时需要手动设置取消点，就是pthread_testcancel()

## pthread_detach 线程分离

int pthread_detach(pthread_t thread);     设置线程分离

       thread: 待分离的线程id

       返回值：成功：0

           失败：errno  

下面这个例子，使用detach分离线程，照理来说，分离后的线程会自动回收：

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.    

9.  **void** *tfn(**void** *arg)  

10. {  

11.     printf("thread: pid = %d, tid = %lu\n", getpid(), pthread_self());  

12.   

13.     **return** NULL;  

14. }  

15.   

16. **int** main(**int** argc, **char** *argv[])  

17. {  

18.     pthread_t tid;  

19.   

20.     **int** ret = pthread_create(&tid, NULL, tfn, NULL);  

21.     **if** (ret != 0) {  

22.         perror("pthread_create error");  

23.     }  

24.     ret = pthread_detach(tid);              // 设置线程分离` 线程终止,会自动清理pcb,无需回收  

25.     **if** (ret != 0) {  

26.         perror("pthread_detach error");  

27.     }  

28.   

29.     sleep(1);  

30.   

31.     ret = pthread_join(tid, NULL);  

32.     printf("join ret = %d\n", ret);  

33.     **if** (ret != 0) {  

34.         perror("pthread_join error");  

35.     }  

36.   

37.     printf("main: pid = %d, tid = %lu\n", getpid(), pthread_self());  

38.   

39.     pthread_exit((**void** *)0);  

40. }  

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image436.jpg)

    这里，问题出现了，join出错了，但是没打印错误原因。之前的perror方法检查线程错误是有问题的。应该使用strerror，修改代码如下：

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.    

9.  **void** *tfn(**void** *arg)  

10. {  

11.     printf("thread: pid = %d, tid = %lu\n", getpid(), pthread_self());  

12.   

13.     **return** NULL;  

14. }  

15.   

16. **int** main(**int** argc, **char** *argv[])  

17. {  

18.     pthread_t tid;  

19.   

20.     **int** ret = pthread_create(&tid, NULL, tfn, NULL);  

21.     **if** (ret != 0) {  

22.         fprintf(stderr, "pthread_create error: %s\n", strerror(ret));  

23.         exit(1);  

24.     }  

25.     ret = pthread_detach(tid);              // 设置线程分离` 线程终止,会自动清理pcb,无需回收  

26.     **if** (ret != 0) {  

27.         fprintf(stderr, "pthread_detach error: %s\n", strerror(ret));  

28.         exit(1);  

29.     }  

30.   

31.     sleep(1);  

32.   

33.     ret = pthread_join(tid, NULL);  

34.     printf("join ret = %d\n", ret);  

35.     **if** (ret != 0) {  

36.         fprintf(stderr, "pthread_join error: %s\n", strerror(ret));  

37.         exit(1);  

38.     }  

39.   

40.     printf("main: pid = %d, tid = %lu\n", getpid(), pthread_self());  

41.   

42.     pthread_exit((**void** *)0);  

43. }  

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image438.jpg)

出错，是因为线程分离后，系统会自动回收资源，用pthread_join去回收已经被系统回收的线程，那个线程号就是无效参数。

## 进程和线程控制原语对比

线程控制原语                进程控制原语

    pthread_create()            fork();

    pthread_self()                  getpid();

    pthread_exit()                  exit();       / return

    pthread_join()                  wait()/waitpid()

    pthread_cancel()            kill()

    pthread_detach()

## 线程属性设置分离线程

线程属性：

    设置分离属性。

    pthread_attr_t attr      创建一个线程属性结构体变量

    pthread_attr_init(&attr);   初始化线程属性

    pthread_attr_setdetachstate(&attr,  PTHREAD_CREATE_DETACHED);      设置线程属性为 分离态

    pthread_create(&tid, &attr, tfn, NULL); 借助修改后的 设置线程属性 创建为分离态的新线程

    pthread_attr_destroy(&attr);    销毁线程属性

调整线程状态，使线程创建出来就是分离态，代码如下：

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.    

9.  **void** *tfn(**void** *arg)  

10. {  

11.     printf("thread: pid = %d, tid = %lu\n", getpid(), pthread_self());  

12.   

13.     **return** NULL;  

14. }  

15.   

16. **int** main(**int** argc, **char** *argv[])  

17. {  

18.     pthread_t tid;  

19.   

20.     pthread_attr_t attr;  

21.   

22.     **int** ret = pthread_attr_init(&attr);  

23.     **if** (ret != 0) {  

24.         fprintf(stderr, "attr_init error:%s\n", strerror(ret));  

25.         exit(1);  

26.     }  

27.   

28.     ret = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);      // 设置线程属性为 分离属性  

29.     **if** (ret != 0) {  

30.         fprintf(stderr, "attr_setdetachstate error:%s\n", strerror(ret));  

31.         exit(1);  

32.     }  

33.   

34.     ret = pthread_create(&tid, &attr, tfn, NULL);  

35.     **if** (ret != 0) {  

36.         perror("pthread_create error");  

37.     }  

38.   

39.     ret = pthread_attr_destroy(&attr);  

40.     **if** (ret != 0) {  

41.         fprintf(stderr, "attr_destroy error:%s\n", strerror(ret));  

42.         exit(1);  

43.     }  

44.   

45.     ret = pthread_join(tid, NULL);  

46.     **if** (ret != 0) {  

47.         fprintf(stderr, "pthread_join error:%s\n", strerror(ret));  

48.         exit(1);  

49.     }  

50.   

51.     printf("main: pid = %d, tid = %lu\n", getpid(), pthread_self());  

52.   

53.     pthread_exit((**void** *)0);  

54. }  

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image440.jpg)

如图，pthread_join报错，说明线程已经自动回收，设置分离成功。

## 线程使用注意事项

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image442.jpg)

## 总结

守护进程：

    daemon进程。通常运行与操作系统后台，脱离控制终端。一般不与用户直接交互。周期性的等待某个事件发生或周期性执行某一动作。

    不受用户登录注销影响。通常采用以d结尾的命名方式。

守护进程创建步骤：

    1. fork子进程，让父进程终止。

    2. 子进程调用 setsid() 创建新会话

    3. 通常根据需要，改变工作目录位置 chdir()， 防止目录被卸载。

    4. 通常根据需要，重设umask文件权限掩码，影响新文件的创建权限。  022 -- 755 0345 --- 432   r---wx-w-   422

    5. 通常根据需要，关闭/重定向 文件描述符

    6. 守护进程 业务逻辑。while（）

=============================================================

线程概念：

    进程：有独立的 进程地址空间。有独立的pcb。   分配资源的最小单位。

    线程：有独立的pcb。没有独立的进程地址空间。  最小单位的执行。

    ps -Lf 进程id    ---> 线程号。LWP  --》cpu 执行的最小单位。

线程共享：

    独享 栈空间（内核栈、用户栈）

   共享 ./text./data ./rodataa ./bsss heap  ---> 共享【全局变量】（errno）

线程控制原语：

    pthread_t pthread_self(void);   获取线程id。 线程id是在进程地址空间内部，用来标识线程身份的id号。

       返回值：本线程id

    检查出错返回：  线程中。

       fprintf(stderr, "xxx error: %s\n", strerror(ret));

    int pthread_create(pthread_t *tid, const pthread_attr_t *attr, void *(*start_rountn)(void *), void *arg); 创建子线程。

       参1：传出参数，表新创建的子线程 id

       参2：线程属性。传NULL表使用默认属性。

       参3：子线程回调函数。创建成功，ptherad_create函数返回时，该函数会被自动调用。

       参4：参3的参数。没有的话，传NULL

       返回值：成功：0

           失败：errno

    循环创建N个子线程：

       for （i = 0； i < 5; i++）

           pthread_create(&tid, NULL, tfn, (void *)i);   // 将 int 类型 i， 强转成 void *， 传参。  

    void pthread_exit(void *retval);  退出当前线程。

       retval：退出值。 无退出值时，NULL

       exit();    退出当前进程。

       return: 返回到调用者那里去。

       pthread_exit(): 退出当前线程。

    int pthread_join(pthread_t thread, void **retval);  阻塞 回收线程。

       thread: 待回收的线程id

       retval：传出参数。 回收的那个线程的退出值。

           线程异常借助，值为 -1。

       返回值：成功：0

           失败：errno

    int pthread_detach(pthread_t thread);     设置线程分离

       thread: 待分离的线程id

       返回值：成功：0

           失败：errno  

    int pthread_cancel(pthread_t thread);     杀死一个线程。  需要到达取消点（保存点）

       thread: 待杀死的线程id

       返回值：成功：0

           失败：errno

       如果，子线程没有到达取消点， 那么 pthread_cancel 无效。

       我们可以在程序中，手动添加一个取消点。使用 pthread_testcancel();

       成功被 pthread_cancel() 杀死的线程，返回 -1.使用pthead_join 回收。

    线程控制原语                进程控制原语

    pthread_create()            fork();

    pthread_self()                  getpid();

    pthread_exit()                  exit();       / return

    pthread_join()                  wait()/waitpid()

    pthread_cancel()            kill()

    pthread_detach()

线程属性：

    设置分离属性。

    pthread_attr_t attr      创建一个线程属性结构体变量

    pthread_attr_init(&attr);   初始化线程属性

    pthread_attr_setdetachstate(&attr,  PTHREAD_CREATE_DETACHED);      设置线程属性为 分离态

    pthread_create(&tid, &attr, tfn, NULL); 借助修改后的 设置线程属性 创建为分离态的新线程

    pthread_attr_destroy(&attr);    销毁线程属性

## 线程同步概念

互斥：是指散步在不同进程之间的若干程序片断，当某个进程运行其中一个程序片段时，**其它进程就不能运行它们之中的任一程序片段**，只能等到该进程运行完这个程序片段后才可以运行。

同步：是指散步在不同进程之间的若干程序片断，它们的**运行必须严格按照规定的 某种先后次序来运行**，这种先后次序依赖于要完成的特定的任务。
生产者消费者 生成者必须生产了产品，消费者才能运行其功能

联系：

同步是一种更为复杂的互斥，而互斥是一种特殊的同步。也就是说互斥是两个线程之间不可以同时运行，他们会相互排斥，必须等待一个线程运行完毕，另一个才能运行，而同步也是不能同时运行，但他是必须要安照某种次序来运行相应的线程（也是一种互斥）。

互斥：是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。但互斥无法限制访问者对资源的访问顺序，即访问是无序的。　　

同步：是指在互斥的基础上（大多数情况），通过其它机制实现访问者对资源的有序访问。在大多数情况下，同步已经实现了互斥，特别是所有写入资源的情况必定是互斥的。少数情况是指可以允许多个访问者同时访问资源。

线程同步：

    协同步调，对公共区域数据按序访问。防止数据混乱，产生与时间有关的错误。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image444.jpg)

数据混乱的原因：

1.  资源共享(独享资源则不会)

2.  调度随机(意味着数据访问会出现竞争)

3.  线程间缺乏必要同步机制

## 锁使用的注意事项

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image446.jpg)

锁的使用：

    建议锁！对公共数据进行保护。所有线程【应该】在访问公共数据前先拿锁再访问。但，锁本身不具备强制性。

## 借助互斥锁管理共享数据实现同步

下面一个小例子，数据共享导致的混乱：


```
1.  #include <stdio.h>  

2.  #include <string.h>  

3.  #include <pthread.h>  

4.  #include <stdlib.h>  

5.  #include <unistd.h>  

6.    

7.  **void** *tfn(**void** *arg)  

8.  {  

9.      srand(time(NULL));  

10.   

11.     **while** (1) {  

12.   

13.         printf("hello ");  

14.         sleep(rand() % 3);  /*模拟长时间操作共享资源，导致cpu易主，产生与时间有关的错误*/  

15.         printf("world\n");  

16.         sleep(rand() % 3);  

17.     }  

18.   

19.     **return** NULL;  

20. }  

21.   

22. **int** main(**void**)  

23. {  

24.     pthread_t tid;  

25.     srand(time(NULL));  

26.   

27.     pthread_create(&tid, NULL, tfn, NULL);  

28.     **while** (1) {  

29.   

30.         printf("HELLO ");  

31.         sleep(rand() % 3);  

32.         printf("WORLD\n");  

33.         sleep(rand() % 3);  

34.   

35.     }  

36.     pthread_join(tid, NULL);  

37.   

38.     **return** 0;  

39. }  

```


主要应用函数：


```c
    pthread_mutex_init       函数 //初始化锁
 
    pthread_mutex_destory    函数//销毁锁

    pthread_mutex_lock       函数//上锁

    pthread_mutex_trylock    函数//尝试上锁

    pthread_mutex_unlock     函数//解锁
```


以上5个函数的返回值都是：成功返回0，失败返回错误号

pthread_mutex_t 类型，其本质是一个结构体。为简化理解，应用时可忽略其实现细节，简单当成整数看待

pthread_mutex_t mutex；变量mutex只有两种取值：0,1

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image450.jpg)

使用mutex(互斥量、互斥锁)一般步骤：

    pthread_mutex_t 类型。

    1. pthread_mutex_t lock;  创建锁

    2  pthread_mutex_init; 初始化      1

    3. pthread_mutex_lock;加锁      1-- --> 0

    4. 访问共享数据（stdout)   //临界区

    5. pthrad_mutext_unlock();解锁     0++ --> 1

    6. pthead_mutex_destroy；销毁锁

int pthread_mutex_init(pthread_mutex_t *restrict mutex,

const pthread_mutexattr_t *restrict attr)

这里的restrict关键字，表示指针指向的内容只能通过这个指针进行修改

restrict关键字：

    用来限定指针变量。被该关键字限定的指针变量所指向的内存操作，必须由本指针完成。

初始化互斥量：

       pthread_mutex_t mutex;

       1. pthread_mutex_init(&mutex, NULL);             动态初始化。

       2. pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;   静态初始化。

修改上面的代码，使用锁实现互斥访问共享区：


```
1.  #include <stdio.h>  

2.  #include <string.h>  

3.  #include <pthread.h>  

4.  #include <stdlib.h>  

5.  #include <unistd.h>  

6.    

7.  pthread_mutex_t mutex;      // 定义一把互斥锁  

8.    

9.  **void** *tfn(**void** *arg)  

10. {  

11.     srand(time(NULL));  

12.   

13.     **while** (1) {  

14.         pthread_mutex_lock(&mutex);     // 加锁  

15.         printf("hello ");  

16.         sleep(rand() % 3);  // 模拟长时间操作共享资源，导致cpu易主，产生与时间有关的错误  

17.         printf("world\n");  

18.         pthread_mutex_unlock(&mutex);   // 解锁  

19.         sleep(rand() % 3);  

20.     }  

21.   

22.     **return** NULL;  

23. }  

24.   

25. **int** main(**void**)  

26. {  

27.     pthread_t tid;  

28.     srand(time(NULL));  

29.     **int** ret = pthread_mutex_init(&mutex, NULL);    // 初始化互斥锁  

30.     **if**(ret != 0){  

31.         fprintf(stderr, "mutex init error:%s\n", strerror(ret));  

32.         exit(1);  

33.     }  

34.   

35.     pthread_create(&tid, NULL, tfn, NULL);  

36.     **while** (1) {  

37.         pthread_mutex_lock(&mutex);     // 加锁  

38.         printf("HELLO ");  

39.         sleep(rand() % 3);  

40.         printf("WORLD\n");  

41.         pthread_mutex_unlock(&mutex);   // 解锁  

42.         sleep(rand() % 3);  

43.     }  

44.     pthread_join(tid, NULL);  

45.       

46.     pthread_mutex_destory(&mutex);     // 销毁互斥锁  

47.   

48.     **return** 0;  

49. }  

```
这样主线程和子线程在访问共享区时就没有交叉输出的情况了。

## 互斥锁使用技巧

注意事项：

       尽量保证锁的粒度， 越小越好。（访问共享数据前，加锁。访问结束【立即】解锁。）

       互斥锁，本质是结构体。 我们可以看成整数。 初值为 1。（pthread_mutex_init() 函数调用成功。）

       加锁： --操作， 阻塞线程。

       解锁： ++操作， 唤醒阻塞在锁上的线程。

       try锁：尝试加锁，成功--。失败，返回。同时设置错误号 EBUSY

## try锁

try锁：尝试加锁，成功--，加锁失败直接返回错误号(如EBUSY)，不阻塞

## 读写锁操作函数原型

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image454.jpg)

读写锁：

    锁只有一把。以读方式给数据加锁——读锁。以写方式给数据加锁——写锁。

    读共享，写独占。

    写锁优先级高。

    相较于互斥量而言，当读线程多的时候，提高访问效率

    pthread_rwlock_t  rwlock;

    pthread_rwlock_init(&rwlock, NULL);

    pthread_rwlock_rdlock(&rwlock);    try

    pthread_rwlock_wrlock(&rwlock);    try

    pthread_rwlock_unlock(&rwlock);

    pthread_rwlock_destroy(&rwlock);

以上函数都是成功返回0，失败返回错误号。

pthread_rwlock_t 类型    用于定义一个读写锁变量

pthread_rwlock_t  rwlock

## 两种死锁

【死锁】：

    是使用锁不恰当导致的现象：

       1. 对一个锁反复lock。

       2. 两个线程，各自持有一把锁，请求另一把。

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image456.jpg)

## 读写锁原理

读写锁：

    锁只有一把。以读方式给数据加锁——读锁。以写方式给数据加锁——写锁。

    读共享，写独占。

    写锁优先级高。

    相较于互斥量而言，当读线程多的时候，提高访问效率

    pthread_rwlock_t  rwlock;

    pthread_rwlock_init(&rwlock, NULL);  //初始化

    pthread_rwlock_rdlock(&rwlock);    try  //读

    pthread_rwlock_wrlock(&rwlock);    try  //写

    pthread_rwlock_unlock(&rwlock);   //解锁

    pthread_rwlock_destroy(&rwlock);  //删除

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image458.jpg)

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image460.jpg)

## rwlock

一个读写锁的例子，核心还是

读共享，写独占。

    写锁优先级高。


```
1.  /* 3个线程不定时 "写" 全局资源，5个线程不定时 "读" 同一全局资源 */  

2.    

3.  #include <stdio.h>  

4.  #include <unistd.h>  

5.  #include <pthread.h>  

6.    

7.  **int** counter;                          //全局资源  

8.  pthread_rwlock_t rwlock;  

9.    

10. **void** *th_write(**void** *arg)  

11. {  

12.     **int** t;  

13.     **int** i = (**int**)arg;  

14.   

15.     **while** (1) {  

16.         t = counter;                    // 保存写之前的值  

17.         usleep(1000);  

18.   

19.         pthread_rwlock_wrlock(&rwlock);  

20.         printf("=======write %d: %lu: counter=%d ++counter=%d\n", i, pthread_self(), t, ++counter);  

21.         pthread_rwlock_unlock(&rwlock);  

22.   

23.         usleep(9000);               // 给 r 锁提供机会  

24.     }  

25.     **return** NULL;  

26. }  

27.   

28. **void** *th_read(**void** *arg)  

29. {  

30.     **int** i = (**int**)arg;  

31.   

32.     **while** (1) {  

33.         pthread_rwlock_rdlock(&rwlock);  

34.         printf("----------------------------read %d: %lu: %d\n", i, pthread_self(), counter);  

35.         pthread_rwlock_unlock(&rwlock);  

36.   

37.         usleep(2000);                // 给写锁提供机会  

38.     }  

39.     **return** NULL;  

40. }  

41.   

42. **int** main(**void**)  

43. {  

44.     **int** i;  

45.     pthread_t tid[8];  

46.   

47.     pthread_rwlock_init(&rwlock, NULL);  

48.   

49.     **for** (i = 0; i < 3; i++)  

50.         pthread_create(&tid[i], NULL, th_write, (**void** *)i);  

51.   

52.     **for** (i = 0; i < 5; i++)  

53.         pthread_create(&tid[i+3], NULL, th_read, (**void** *)i);  

54.   

55.     **for** (i = 0; i < 8; i++)  

56.         pthread_join(tid[i], NULL);  

57.   

58.     pthread_rwlock_destroy(&rwlock);            //释放读写琐  

59.   

60.     **return** 0;  

61. }  

```



由于读共享，写独占。写锁优先级高。前面5个read一定先于后面的write到达的，不然write会抢到锁先进行写操作。

## 静态初始化条件变量和互斥量

条件变量：

    本身不是锁！  但是通常结合锁来使用。 mutex

主要应用函数：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image462.png)

pthread_cond_t cond;

    初始化条件变量：

       1. pthread_cond_init(&cond, NULL);            动态初始化。

       2. pthread_cond_t cond = PTHREAD_COND_INITIALIZER;   静态初始化。

## 条件变量和相关函数wait

阻塞等待条件：

       pthread_cond_wait(&cond, &mutex);

       作用：

1） 阻塞等待条件变量满足

           2） 解锁已经加锁成功的信号量 （相当于 pthread_mutex_unlock(&mutex)），12两步为一个原子操作

3)  当条件满足，函数返回时，解除阻塞并重新申请获取互斥锁。重新加锁信号量 （相当于， pthread_mutex_lock(&mutex);）

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image464.jpg)

## 条件变量的生产者消费者模型分析

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image466.jpg)

## 条件变量生产者消费者代码预览


```
代码如下：

1.  /*借助条件变量模拟 生产者-消费者 问题*/  

2.  #include <stdlib.h>  

3.  #include <unistd.h>  

4.  #include <pthread.h>  

5.  #include <stdio.h>  

6.    

7.  /*链表作为公享数据,需被互斥量保护*/  

8.  **struct** msg {  

9.      **struct** msg *next;  

10.     **int** num;  

11. };  

12.   

13. **struct** msg *head;  

14.   

15. /* 静态初始化 一个条件变量 和 一个互斥量*/  

16. pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;  

17. pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;  

18.   

19. **void** *consumer(**void** *p)  

20. {  

21.     **struct** msg *mp;  

22.   

23.     **for** (;;) {  

24.         pthread_mutex_lock(&lock);  

25.         **while** (head == NULL) {           //头指针为空,说明没有节点    可以为if吗  

26.             pthread_cond_wait(&has_product, &lock);   // 解锁，并阻塞等待  

27.         }  

28.         mp = head;        

29.         head = mp->next;                 //模拟消费掉一个产品  

30.         pthread_mutex_unlock(&lock);  

31.   

32.         printf("-Consume %lu---%d\n", pthread_self(), mp->num);  

33.         free(mp);  

34.         sleep(rand() % 5);  

35.     }  

36. }  

37.   

38. **void** *producer(**void** *p)  

39. {  

40.     **struct** msg *mp;  

41.   

42.     **for** (;;) {  

43.         mp = malloc(**sizeof**(**struct** msg));  

44.         mp->num = rand() % 1000 + 1;        //模拟生产一个产品  

45.         printf("-Produce ---------------------%d\n", mp->num);  

46.   

47.         pthread_mutex_lock(&lock);  

48.         mp->next = head;  

49.         head = mp;  

50.         pthread_mutex_unlock(&lock);  

51.   

52.         pthread_cond_signal(&has_product);  //将等待在该条件变量上的一个线程唤醒  

53.         sleep(rand() % 5);  

54.     }  

55. }  

56.   

57. **int** main(**int** argc, **char** *argv[])  

58. {  

59.     pthread_t pid, cid;  

60.     srand(time(NULL));  

61.   

62.     pthread_create(&pid, NULL, producer, NULL);  

63.     pthread_create(&cid, NULL, consumer, NULL);  

64.   

65.     pthread_join(pid, NULL);  

66.     pthread_join(cid, NULL);  

67.   

68.     **return** 0;  

69. }  

```

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image468.jpg)

可以看到，消费者按序消费，毕竟是链表。

## 条件变量实现生产者消费者代码

```

1.  #include <stdio.h>  

2.  #include <stdlib.h>  

3.  #include <string.h>  

4.  #include <unistd.h>  

5.  #include <errno.h>  

6.  #include <pthread.h>  

7.    

8.  **void** err_thread(**int** ret, **char** *str)  

9.  {  

10.     **if** (ret != 0) {  

11.         fprintf(stderr, "%s:%s\n", str, strerror(ret));  

12.         pthread_exit(NULL);  

13.     }  

14. }  

15.   

16. **struct** msg {  

17.     **int** num;  

18.     **struct** msg *next;  

19. };  

20.   

21. **struct** msg *head;  

22.   

23. pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;      // 定义/初始化一个互斥量  

24. pthread_cond_t has_data = PTHREAD_COND_INITIALIZER;      // 定义/初始化一个条件变量  

25.   

26. **void** *produser(**void** *arg)  

27. {  

28.     **while** (1) {  

29.         **struct** msg *mp = malloc(**sizeof**(**struct** msg));  

30.   

31.         mp->num = rand() % 1000 + 1;                        // 模拟生产一个数据`  

32.         printf("--produce %d\n", mp->num);  

33.   

34.         pthread_mutex_lock(&mutex);                         // 加锁 互斥量  

35.         mp->next = head;                                    // 写公共区域  

36.         head = mp;  

37.         pthread_mutex_unlock(&mutex);                       // 解锁 互斥量  

38.   

39.         pthread_cond_signal(&has_data);                     // 唤醒阻塞在条件变量 has_data上的线程.  

40.   

41.         sleep(rand() % 3);  

42.     }  

43.   

44.     **return** NULL;  

45. }  

46.   

47. **void** *consumer(**void** *arg)  

48. {  

49.     **while** (1) {  

50.         **struct** msg *mp;  

51.   

52.         pthread_mutex_lock(&mutex);                         // 加锁 互斥量  

53.         **if** (head == NULL) {  

54.             pthread_cond_wait(&has_data, &mutex);           // 阻塞等待条件变量, 解锁  

55.         }                                                   // pthread_cond_wait 返回时, 重写加锁 mutex  

56.   

57.         mp = head;  

58.         head = mp->next;  

59.   

60.         pthread_mutex_unlock(&mutex);                       // 解锁 互斥量  

61.         printf("---------consumer:%d\n", mp->num);  

62.   

63.         free(mp);  

64.         sleep(rand()%3);  

65.     }  

66.   

67.     **return** NULL;  

68. }  

69.   

70. **int** main(**int** argc, **char** *argv[])  

71. {  

72.     **int** ret;  

73.     pthread_t pid, cid;  

74.   

75.     srand(time(NULL));  

76.   

77.     ret = pthread_create(&pid, NULL, produser, NULL);           // 生产者  

78.     **if** (ret != 0)   

79.         err_thread(ret, "pthread_create produser error");  

80.   

81.     ret = pthread_create(&cid, NULL, consumer, NULL);           // 消费者  

82.     **if** (ret != 0)   

83.         err_thread(ret, "pthread_create consuer error");  

84.   

85.     pthread_join(pid, NULL);  

86.     pthread_join(cid, NULL);  

87.   

88.     **return** 0;  

89. }  

```

编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image470.jpg)



## 条件变量signal注意事项

pthread_cond_signal(): 唤醒阻塞在条件变量上的 (至少)一个线程。

pthread_cond_broadcast()： 唤醒阻塞在条件变量上的 所有线程。

## 信号量概念及其相关操作函数

信号量：

    应用于线程、进程间同步。

    相当于 初始化值为 N 的互斥量。  N值，表示可以同时访问共享数据区的线程数。

    函数：

       sem_t sem; 定义类型。

       int sem_init(sem_t *sem, int pshared, unsigned int value);

       参数：

           sem： 信号量

           pshared：  0： 用于线程间同步

                  1： 用于进程间同步

           value：N值。（指定同时访问的线程数）

       sem_destroy();

       sem_wait();       一次调用，做一次-- 操作， 当信号量的值为 0 时，再次 -- 就会阻塞。 （对比 pthread_mutex_lock）

       sem_post();       一次调用，做一次++ 操作. 当信号量的值为 N 时, 再次 ++ 就会阻塞。（对比 pthread_mutex_unlock）

## 信号量实现的生产者消费者

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image474.jpg)


```
代码如下：

1.  /*信号量实现 生产者 消费者问题*/  

2.    

3.  #include <stdlib.h>  

4.  #include <unistd.h>  

5.  #include <pthread.h>  

6.  #include <stdio.h>  

7.  #include <semaphore.h>  

8.    

9.  #define NUM 5                 

10.   

11. **int** queue[NUM];                                     //全局数组实现环形队列  

12. sem_t blank_number, product_number;                 //空格子信号量, 产品信号量  

13.   

14. **void** *producer(**void** *arg)  

15. {  

16.     **int** i = 0;  

17.   

18.     **while** (1) {  

19.         sem_wait(&blank_number);                    //生产者将空格子数--,为0则阻塞等待  

20.         queue[i] = rand() % 1000 + 1;               //生产一个产品  

21.         printf("----Produce---%d\n", queue[i]);          

22.         sem_post(&product_number);                  //将产品数++  

23.   

24.         i = (i+1) % NUM;                            //借助下标实现环形  

25.         sleep(rand()%1);  

26.     }  

27. }  

28.   

29. **void** *consumer(**void** *arg)  

30. {  

31.     **int** i = 0;  

32.   

33.     **while** (1) {  

34.         sem_wait(&product_number);                  //消费者将产品数--,为0则阻塞等待  

35.         printf("-Consume---%d\n", queue[i]);  

36.         queue[i] = 0;                               //消费一个产品   

37.         sem_post(&blank_number);                    //消费掉以后,将空格子数++  

38.   

39.         i = (i+1) % NUM;  

40.         sleep(rand()%3);  

41.     }  

42. }  

43.   

44. **int** main(**int** argc, **char** *argv[])  

45. {  

46.     pthread_t pid, cid;  

47.   

48.     sem_init(&blank_number, 0, NUM);                //初始化空格子信号量为5, 线程间共享 -- 0  

49.     sem_init(&product_number, 0, 0);                //产品数为0  

50.   

51.     pthread_create(&pid, NULL, producer, NULL);  

52.     pthread_create(&cid, NULL, consumer, NULL);  

53.   

54.     pthread_join(pid, NULL);  

55.     pthread_join(cid, NULL);  

56.   

57.     sem_destroy(&blank_number);  

58.     sem_destroy(&product_number);  

59.   

60.     **return** 0;  

61. }  
```


编译运行，结果如下：

![](file:///C:/Users/Administrator/AppData/Local/Temp/msohtmlclip1/01/clip_image476.jpg)

## 总结

线程同步：

    协同步调，对公共区域数据按序访问。防止数据混乱，产生与时间有关的错误。

锁的使用：

    建议锁！对公共数据进行保护。所有线程【应该】在访问公共数据前先拿锁再访问。但，锁本身不具备强制性。

使用mutex(互斥量、互斥锁)一般步骤：

    pthread_mutex_t 类型。

    1. pthread_mutex_t lock;  创建锁

    2  pthread_mutex_init; 初始化      1

    3. pthread_mutex_lock;加锁      1-- --> 0

    4. 访问共享数据（stdout)   

    5. pthrad_mutext_unlock();解锁     0++ --> 1

    6. pthead_mutex_destroy；销毁锁

    初始化互斥量：

       pthread_mutex_t mutex;

       1. pthread_mutex_init(&mutex, NULL);             动态初始化。

       2. pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;   静态初始化。

    注意事项：

       尽量保证锁的粒度， 越小越好。（访问共享数据前，加锁。访问结束【立即】解锁。）

       互斥锁，本质是结构体。 我们可以看成整数。 初值为 1。（pthread_mutex_init() 函数调用成功。）

       加锁： --操作， 阻塞线程。

       解锁： ++操作， 换醒阻塞在锁上的线程。

       try锁：尝试加锁，成功--。失败，返回。同时设置错误号 EBUSY

restrict关键字：

    用来限定指针变量。被该关键字限定的指针变量所指向的内存操作，必须由本指针完成。

【死锁】：

    是使用锁不恰当导致的现象：

       1. 对一个锁反复lock。

       2. 两个线程，各自持有一把锁，请求另一把。

读写锁：

    锁只有一把。以读方式给数据加锁——读锁。以写方式给数据加锁——写锁。

    读共享，写独占。

    写锁优先级高。

    相较于互斥量而言，当读线程多的时候，提高访问效率

    pthread_rwlock_t  rwlock;

    pthread_rwlock_init(&rwlock, NULL);

    pthread_rwlock_rdlock(&rwlock);    try

    pthread_rwlock_wrlock(&rwlock);    try

    pthread_rwlock_unlock(&rwlock);

    pthread_rwlock_destroy(&rwlock);

条件变量：

    本身不是锁！  但是通常结合锁来使用。 mutex

    pthread_cond_t cond;

    初始化条件变量：

       1. pthread_cond_init(&cond, NULL);            动态初始化。

       2. pthread_cond_t cond = PTHREAD_COND_INITIALIZER;   静态初始化。

    阻塞等待条件：

       pthread_cond_wait(&cond, &mutex);

       作用： 1） 阻塞等待条件变量满足

           2） 解锁已经加锁成功的信号量 （相当于 pthread_mutex_unlock(&mutex)）

           3)  当条件满足，函数返回时，重新加锁信号量 （相当于， pthread_mutex_lock(&mutex);）

    pthread_cond_signal(): 唤醒阻塞在条件变量上的 (至少)一个线程。

    pthread_cond_broadcast()： 唤醒阻塞在条件变量上的 所有线程。

    【要求，能够借助条件变量，完成生成者消费者】

信号量：

    应用于线程、进程间同步。

    相当于 初始化值为 N 的互斥量。  N值，表示可以同时访问共享数据区的线程数。

    函数：

       sem_t sem; 定义类型。

       int sem_init(sem_t *sem, int pshared, unsigned int value);

       参数：

           sem： 信号量

           pshared：  0： 用于线程间同步

                  1： 用于进程间同步

           value：N值。（指定同时访问的线程数）

       sem_destroy();

       sem_wait();       一次调用，做一次-- 操作， 当信号量的值为 0 时，再次 -- 就会阻塞。 （对比 pthread_mutex_lock）

       sem_post();       一次调用，做一次++ 操作. 当信号量的值为 N 时, 再次 ++ 就会阻塞。（对比 pthread_mutex_unlock）