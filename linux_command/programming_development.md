# 编程开发

## readelf

## objdump

## size

## objcopy

## nm

## [make](http://www.ruanyifeng.com/blog/2015/02/make.html)

`make`是一个工具程序，经过读取`Makefile`文件，自动化构建软件。

例如：

    > make
    or
    > make -f test.mk

在不指定目标文件的情况下， 默认`make`命令会自动读取当前路径下的`Makefile`文件并构建。

##### 规则

`Makefile`的格式是：

    # 用 “#”代表注释
    <target> : <dependencies>
    [tab] <commands>
    [tab] <commands>
    [tab] <commands>

解释：

    在Makefile文件中，采用以 “#” 开头的注释方式。
    接下来， 
    冒号左边的部分称为 “目标（target）”，
    冒号后边的部分称为 “前置条件（prerequisites）”，
    [tab]表示此处应该用 “tab键位”进行缩进，否则会报以下错误：
    “*** missing separator。”

> 特别提示：每个`<commands>`之间不存在任何上下文关系。

###### 目标（target）

一个`<target>`就构成了一条规则。`<target>`通常是文件名，指明`make`命令想要构建的对象。
也可以是多个文件名，之间用逗号分开。`<target>`还可以是某个操作的名字，这称为`伪目标`（phony target）。

例如:

    > cat Makefile
    create:
        touch hello
        touch world

    hello:
        echo "hello"

    .PHONY: world
    world:
        echo "world"

    .PHONY echoing
    echoing:
        echo "this is echoing order"
        @echo "no echoing"

运行：

    > make create
    touch hello
    touch world

    > make hello
    make: 'hello' is up to date.
    
    > make world
    echo "world"
    world

    > make echoing
    echo "this is echoing order"
    this is echoing order
    no echoing

解释：

    上述代码的 “make create”执行后，会在当前目录下创建两个文件：hello文件与world文件。
    再次执行“make hello”，在没有声明“hello目标”是一个伪目标的情况下，
    make命令会去检查当前目录下是否已经存在名字为“hello”的文件，如果存在，则不会继续执行。
    所以，为了避免这种问题，我们会在声明一个“伪目标”时，显示指定出来：
    .PHONY: <target>

    正常情况下，make会打印每条命令，然后再执行，这叫做“回声”（echoing）,
    关闭回声的办法是：在命令之前加上“@”，参考上述代码中的“echoing”目标。

> 注意，如果`make`命令在运行时没有指定目标，默认会执行Makefile文件的第一个目标。
> 可以将上述的执行指令 “make create” 更改为 “make”

###### 前置条件（prerequisites）

前置条件通常是一组文件名，之间用空格分割，它是当前`<target>`是否能被执行的重要条件：
只要有一个文件不存在，或者被更新，就需要重新构建`<target>`。

例如：

    > cat Makefile
    file1.txt:
            touch file1.txt
            echo "1" >> file1.txt

    file2.txt:
            touch file2.txt
            echo "2" >> file2.txt

    file3.txt: file1.txt file2.txt
            cat file1.txt >> file3.txt
            cat file2.txt >> file3.txt

    .PHONY: echo1
    echo1:
            echo "1"

    .PHONY: echo2
    echo2:
            echo "2"

    .PHONY: echo3
    echo3: echo1 echo2
            echo "3"

运行：

    > make file3.txt
    touch file1.txt
    echo "1" >> file1.txt
    touch file2.txt
    echo "2" >> file2.txt
    cat file1.txt >> file3.txt
    cat file2.txt >> file3.txt

    > cat file3.txt
    1
    2

    > make echo3
    echo "1"
    1
    echo "2"
    2
    echo "3"
    3

> 对于`伪依赖`，一定会执行.

###### 命令（commands）

命令由一行或者多行Shell组成，是构建`目标`的具体指令。
需要注意的是，每行命令在一个单独的shell中执行。这些Shell之间没有继承关系。

例如：

    > cat Makefile
    test1:
            @name="wings"
            @echo "my name is" $$name

    test2:
            @name="wings";\
            echo "my name is" $$name

    .ONESHELL:
    test3:
            @name="wings"
            @echo "my name is" $$name

运行：

    > make test1
    my name is
    > make test2
    my name is wings
    > make test3
    my name is wings

解释：

    利用反斜杠 "\" 与 分号 ":" 来指明这些命令在一行。
    或者加上“.ONESHELL:”标记。

##### 语法

例如：

    > cat Makefile
    %.txt:
	    touch $@

    txt = Hello World
    test:
            @echo $(txt)

运行：

    > make test1.txt
    touch test1.txt
    
    > make test2.txt
    touch test2.txt

    > ls
    Makefile  test1.txt  test2.txt

    > make test
    Hello World

由上述代码可知，Makefile中有一些匹配规则、自定义变量、自动变量：

匹配规则：

    匹配符是 %
    例如 target 为 "%.txt"
    那么我们命令 “make test1.txt” 或者 "make test2.txt"都能命中此规则。

自定义变量：

    Makefile 允许使用等号自定义变量。
    调用时，变量需要放在 $( ) 之中。
    调用Shell变量，需要在美元符号前，再加一个美元符号，这是因为Make命令会对美元符号转义。

自动变量：

* $@：代指当前目标

    比如，make create的 `$@` 就指代create。

* $<：指代第一个前置条件

    比如，规则为 t: p1 p2，那么`$<`就指代p1。

* $?：指代比目标更新的所有前置条件，之间以空格分隔

    比如，规则为 t: p1 p2，其中 p2 的时间戳比 t 新，`$?`就指代p2

* $^：指代所有前置条件，之间以空格分隔

    比如，规则为 t: p1 p2，那么 `$^` 就指代 p1 p2 。

* $*：指代匹配符 % 匹配的部分

    比如目标`%.txt` 匹配 f1.txt 中的f1 ，`$*` 就表示 f1。

##### 函数

`Makefile`可以使用函数，格式为：

    $(function arguments)
    或者
    ${function arguments}

