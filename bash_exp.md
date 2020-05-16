# Bash教程

## 1 模式扩展
前言：  
Shell 接收到用户输入的命令以后，会根据空格将用户的输入，拆分成一个个词元（token）。然后，Shell 会扩展词元里面的特殊字符，扩展完成后才会调用相应的命令。  
这种特殊字符的扩展，称为模式扩展（globbing）。其中有些用到通配符，又称为通配符扩展（wildcard expansion）。Bash 一共提供八种扩展。

+ 波浪线扩展
+ ? 字符扩展
+ *字符扩展
+ 方括号扩展
+ 大括号扩展
+ 变量扩展
+ 子命令扩展
+ 算术扩展

打开扩展与关闭扩展：

    关闭：
    $ set -o noglob
    # 或者
    $ set -f
    打开
    $ set +o noglob
    # 或者
    $ set +f

### 1.1 波浪线扩展
波浪线~会自动扩展成当前用户的主目录。

    $ echo ~
    >> /home/me


    $ echo ~foo
    >> /home/foo

    $ echo ~root
    >> /root

Bash 会根据波浪号后面的用户名，返回该用户的主目录。

~+会扩展成当前所在的目录，等同于pwd命令。

    $ cd ~/foo
    $ echo ~+
    >> /home/me/foo    

### 1.2 ?扩展
?字符代表文件路径里面的任意单个字符，不包括空字符。比如，Data???匹配所有Data后面跟着三个字符的文件名。

    $ ls ?.txt
    >> a.txt b.txt

上面命令中，?表示单个字符，所以会同时匹配a.txt和b.txt。
如果匹配多个字符，就需要多个?连用。

    $ ls ??.txt
    >> ab.txt

### 1.3 *扩展

*字符代表文件路径里面的任意数量的任意字符，包括零个字符。

    $ ls *.txt
    >> a.txt b.txt ab.txt

上面例子中，*.txt代表后缀名为.txt的所有文件。

如果想输出当前目录的所有文件，直接用*即可。

    $ ls *

如果要匹配隐藏文件，需要写成.*。

    $ echo .*

如果要匹配隐藏文件，同时要排除.和..这两个特殊的隐藏文件，可以与方括号扩展结合使用，写成.[!.]*。

    $ echo .[!.]*

*只匹配当前目录，不会匹配子目录。
    
    # 子目录有一个 a.txt
    # 无效的写法
    $ ls *.txt

    # 有效的写法
    $ ls */*.txt

上面的例子，文本文件在子目录，*.txt不会产生匹配，必须写成*/*.txt。有几层子目录，就必须写几层星号。

### 1.4 方括号扩展
方括号扩展的形式是[...]，只有文件确实存在的前提下才会扩展。如果文件不存在，就会原样输出。

    # 存在文件 a.txt 和 b.txt
    $ ls [ab].txt
    a.txt b.txt

    # 只存在文件 a.txt
    $ ls [ab].txt
    a.txt

方括号扩展还有两种变体：[^...]和[!...]。它们表示匹配不在方括号里面的字符，这两种写法是等价的。比如，[^abc]或[!abc]表示匹配除了a、b、c以外的字符。

    $ ls ?[!a]?
    aba bbb

### 1.5 [start-end] 扩展
方括号扩展有一个简写形式[start-end]，表示匹配一个连续的范围。比如，[a-c]等同于[abc]，[0-9]匹配[0123456789]。


    $ ls report[0-9].txt
    report1.txt
    report2.txt
    report3.txt

常用简写的例子：
+ [a-z]：所有小写字母。
+ [a-zA-Z]：所有小写字母与大写字母。
+ [a-zA-Z0-9]：所有小写字母、大写字母与数字。
+ [abc]*：所有以a、b、c字符之一开头的文件名。
+ program.[co]：文件program.c与文件program.o。
+ BACKUP.[0-9][0-9][0-9]：所有以BACKUP.开头，后面是三个数字的文件名。


这种简写形式有一个否定形式[!start-end]，表示匹配不属于这个范围的字符。比如，[!a-zA-Z]表示匹配非英文字母的字符。


### 1.6 大括号扩展
大括号扩展{...}表示分别扩展成大括号里面的所有值，各个值之间使用逗号分隔。比如，{1,2,3}扩展成1 2 3。

    $ echo Front-{A,B,C}-Back
    Front-A-Back Front-B-Back Front-C-Back

注意，大括号扩展不是文件名扩展。它会扩展成所有给定的值，而不管是否有对应的文件存在。  
逗号前面可以没有值，表示扩展的第一项为空。

    $ cp a.log{,.bak}

    # 等同于
    # cp a.log a.log.bak

大括号可以嵌套。

    $ echo {j{p,pe}g,png}
    jpg jpeg png

    $ echo a{A{1,2},B{3,4}}b
    aA1b aA2b aB3b aB4b

    $ echo {cat,d*}
    cat dawg dg dig dog doug dug

大括号扩展的常见用途为新建一系列目录。

    $ mkdir {2007..2009}-{01..12}

上面命令会新建36个子目录，每个子目录的名字都是”年份-月份“。

这个写法的另一个常见用途，是直接用于for循环。

    for i in {1..4}
        do
    echo $i
        done

简写形式还可以使用第二个双点号（start..end..step），用来指定扩展的步长。

    $ echo {0..8..2}
    >> 0 2 4 6 8

### 1.7 子命令扩展
$(...)可以扩展成另一个命令的运行结果，该命令的所有输出都会作为返回值。
    
    $ echo $(date)
    >> Tue Jan 28 00:01:13 CST 2020

### 1.8 字符类
[[:class:]]表示一个字符类，扩展成某一类特定字符之中的一个。常用的字符类如下。 
+ [[:alnum:]]：匹配任意英文字母与数字
+ [[:alpha:]]：匹配任意英文字母
+ [[:blank:]]：空格和 Tab 键。
+ [[:cntrl:]]：ASCII 码 0-31 的不可打印字符。
+ [[:digit:]]：匹配任意数字 0-9。
+ [[:graph:]]：A-Z、a-z、0-9 和标点符号。
+ [[:lower:]]：匹配任意小写字母 a-z。
+ [[:print:]]：ASCII 码 32-127 的可打印字符。
+ [[:punct:]]：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
+ [[:space:]]：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
+ [[:upper:]]：匹配任意大写字母 A-Z。
+ [[:xdigit:]]：16进制字符（A-F、a-f、0-9）。

输出所有大写字母开头的文件名。
  
    $ echo [[:upper:]]*

### 1.9 扩展使用注意点

(1) 通配符是先解释，再执行。

Bash 接收到命令以后，发现里面有通配符，会进行通配符扩展，然后再执行命令。

    $ ls a*.txt
    >> ab.txt

(2) 只适用于单层路径。

所有文件名扩展只匹配单层路径，不能跨目录匹配，即无法匹配子目录里面的文件。或者说，?或*这样的通配符，不能匹配路径分隔符（/）。
只能写成这样：

    $ ls */*.txt

### 1.10 量词语法
量词语法用来控制模式匹配的次数。它只有在 Bash 的extglob参数打开的情况下才能使用，不过一般是默认打开的。下面的命令可以查询。

    $ shopt extglob
    extglob        	on

量词语法有下面几个。

+ ?(pattern-list)：匹配零个或一个模式。
+ *(pattern-list)：匹配零个或多个模式。
+ +(pattern-list)：匹配一个或多个模式。
+ @(pattern-list)：只匹配一个模式。
+ !(pattern-list)：匹配零个或一个以上的模式，但不匹配单独一个的模式。

下面例子中，+(.txt|.php)匹配文件有一个.txt或.php后缀名。
    
    $ ls abc+(.txt|.php)
    >> abc.php abc.txt

下面例子中，+(.txt)匹配文件有一个或多个.txt后缀名。

    $ ls abc+(.txt)
    abc.txt abc.txt.txt

### 1.11 shopt命令
shopt命令可以调整 Bash 的行为。它有好几个参数跟通配符扩展有关。

shopt命令的使用方法如下。

    # 打开某个参数
    $ shopt -s [optionname]
    
    # 关闭某个参数
    $ shopt -u [optionname]

    # 查询某个参数关闭还是打开
    $ shopt [optionname]

1）dotglob 参数

dotglob参数可以让扩展结果包括隐藏文件（即点开头的文件）。

正常情况下，扩展结果不包括隐藏文件。
打开dotglob，就会包括隐藏文件。

    $ shopt -s dotglob
    $ ls *
    abc.txt .config

2）nullglob 参数

nullglob参数可以让通配符不匹配任何文件名时，返回空字符。

默认情况下，通配符不匹配任何文件名时，会保持不变。
打开nullglob参数，就可以让不匹配的通配符返回空字符串。
   
    $ shopt -s nullglob
    $ rm b*
    >> rm: 缺少操作数

上面例子中，由于没有 b* 匹配的文件名，所以rm b*扩展成了rm，导致报错变成了”缺少操作数“。

3）failglob 参数

failglob参数使得通配符不匹配任何文件名时，Bash 会直接报错，而不是让各个命令去处理。

    $ shopt -s failglob
    $ rm b*
    bash: 无匹配: b*

上面例子中，打开failglob以后，由于b*不匹配任何文件名，Bash 直接报错了，不再让rm命令去处理。

4）extglob 参数

extglob参数使得 Bash 支持 ksh 的一些扩展语法。它默认应该是打开的。

    $ shopt extglob
    >> extglob        	on

它的主要应用是支持量词语法。如果不希望支持量词语法，可以用下面的命令关闭。

    $ shopt -u extglob

5）nocaseglob 参数

nocaseglob参数可以让通配符扩展不区分大小写。

    $ shopt -s nocaseglob
    $ ls /windows/program*
    /windows/ProgramData
    /windows/Program Files
    /windows/Program Files (x86)

上面例子中，打开nocaseglob以后，program*就不区分大小写了，可以匹配ProgramData等。

6）globstar 参数

globstar参数可以使得**匹配零个或多个子目录。该参数默认是关闭的。

假设有下面的文件结构。

a.txt
sub1/b.txt
sub1/sub2/c.txt
上面的文件结构中，顶层目录、第一级子目录sub1、第二级子目录sub1\sub2里面各有一个文本文件。请问怎样才能使用通配符，将它们显示出来？

默认情况下，只能写成下面这样。

    $ ls *.txt */*.txt */*/*.txt
    a.txt  sub1/b.txt  sub1/sub2/c.txt
这是因为*只匹配当前目录，如果要匹配子目录，只能一层层写出来。

打开globstar参数以后，**匹配零个或多个子目录。因此，**/*.txt就可以得到想要的结果。

    $ shopt -s globstar
    $ ls **/*.txt
    a.txt  sub1/b.txt  sub1/sub2/c.txt


## 2 转义和引号

### 2.1 转义

某些字符在 Bash 里面有特殊含义（比如$、&、*）。
上面例子中，输出 $date不会有任何结果，因为 $ 是一个特殊字符。

如果想要原样输出这些特殊字符，就必须在它们前面加上反斜杠，使其变成普通字符。这就叫做“转义”（escape）。

    $ echo \$date
    $date
上面命令中，只有在特殊字符$前面加反斜杠，才能原样输出。

反斜杠本身也是特殊字符，如果想要原样输出反斜杠，就需要对它自身转义，连续使用两个反斜线（\\）。

    $ echo \\
    \

反斜杠除了用于转义，还可以表示一些不可打印的字符。


+ \a：响铃
+ \b：退格
+ \n：换行
+ \r：回车
+ \t：制表符


如果想要在命令行使用这些不可打印的字符，可以把它们放在引号里面，然后使用echo命令的-e参数。

    $ echo a\tb
    atb

    $ echo -e "a\tb"
    a        b

由于反斜杠可以对换行符转义，使得 Bash 认为换行符是一个普通字符，从而可以将一行命令写成多行。

    $ mv \
    /path/to/foo \
    /path/to/bar

    # 等同于
    $ mv /path/to/foo /path/to/bar

如果一条命令过长，就可以在行尾使用反斜杠，将其改写成多行。这是常见的多行命令的写法。

### 2.2 引号

单引号用于保留字符的字面含义，各种特殊字符在单引号里面，都会变为普通字符，比如星号（*）、美元符号（$）、反斜杠（\）等。

    # 不正确
    $ echo it's

    # 不正确
    $ echo 'it\'s'

    # 正确
    $ echo $'it\'s'

双引号比单引号宽松，可以保留大部分特殊字符的本来含义，但是三个字符除外：美元符号（$）、反引号（`）和反斜杠（\）。也就是说，这三个字符在双引号之中，会被 Bash 自动扩展。

通配符*放在双引号之中，就变成了普通字符，会原样输出。这一点需要特别留意，双引号里面不会进行文件名扩展。

    $ echo "$SHELL"
    /bin/bash

    $ echo "`date`"
    Mon Jan 27 13:33:18 CST 2020

美元符号和反引号在双引号中，都保持特殊含义。美元符号用来引用变量，反引号则是执行子命令

双引号的另一个常见的使用场合是，文件名包含空格。这时就必须使用双引号，将文件名放在里面,
双引号会原样保存多余的空格。

    $ echo "this is a     test"
    >> this is a     test


## 3 bash变量
环境变量是 Bash 环境自带的变量，进入 Shell 时已经定义好了，可以直接使用。它们通常是系统定义好的，也可以由用户从父 Shell 传入子 Shell。
env命令或printenv命令，可以显示所有环境变量。

+ BASHPID：Bash 进程的进程 ID。
+ BASHOPTS：当前 Shell 的参数，可以用shopt命令修改。
+ DISPLAY：图形环境的显示器名字，通常是:0，表示 X Server 的第一个显示器。
+ EDITOR：默认的文本编辑器。
+ HOME：用户的主目录。
+ HOST：当前主机的名称。
+ IFS：词与词之间的分隔符，默认为空格。
+ LANG：字符集以及语言编码，比如zh_CN.UTF-8。
+ PATH：由冒号分开的目录列表，当输入可执行程序名后，会搜索这个目录列表。
+ PS1：Shell 提示符。
+ PS2： 输入多行命令时，次要的 Shell 提示符。
+ PWD：当前工作目录。
+ RANDOM：返回一个0到32767之间的随机数。
+ SHELL：Shell 的名字。
+ SHELLOPTS：启动当前 Shell 的set命令的参数，参见《set 命令》一章。
+ TERM：终端类型名，即终端仿真器所用的协议。
+ UID：当前用户的 ID 编号。
+ USER：当前用户的用户名。

set命令可以显示所有变量（包括环境变量和自定义变量），以及所有的 Bash 函数。

考虑到 Bash 将 $1解释成了变量，该变量为空，因此输入就变成了00.00。所以，如果要使用 $ 的原义，需要在 $ 前面放上反斜杠，进行转义。

    $ echo The total is \$100.00
    The total is $100.00

读取变量的时候，变量名也可以使用花括号{}包围，比如$a也可以写成 ${a}。这种写法可以用于变量名与其他字符连用的情况。

    $ a=foo
    $ echo $a_file

    $ echo ${a}_file
    foo_file

上面代码中，变量名a_file不会有任何输出，因为 Bash 将其整个解释为变量，而这个变量是不存在的。只有用花括号区分$a，Bash 才能正确解读。


用户创建的变量仅可用于当前 Shell，子 Shell 默认读取不到父 Shell 定义的变量。为了把变量传递给子 Shell，需要使用export命令。这样输出的变量，对于子 Shell 来说就是环境变量。

export命令用来向子 Shell 输出变量:

    NAME=foo
    export NAME

上面命令输出了变量NAME。变量的赋值和输出也可以在一个步骤中完成。

    # 输出变量 $foo
    $ export foo=bar

    # 新建子 Shell
    $ bash

    # 读取 $foo
    $ echo $foo
    bar

    # 修改继承的变量
    $ foo=baz

    # 退出子 Shell
    $ exit

    # 读取 $foo
    $ echo $foo
    bar

一些特殊变量：
1）$?

$?为上一个命令的退出码，用来判断上一个命令是否执行成功。返回值是0，表示上一个命令执行成功；如果是非零，上一个命令执行失败。

    $ ls doesnotexist
    ls: doesnotexist: No such file or directory

    $ echo $?
    1
上面例子中，ls命令查看一个不存在的文件，导致报错。$?为1，表示上一个命令执行失败。


2）$$

'$$' 为当前 Shell 的进程 ID。

    $ echo $$
    >> 10662

3）$_

$_为上一个命令的最后一个参数。

    $ grep dictionary /usr/share/dict/words
    dictionary

    $ echo $_
    /usr/share/dict/words

4）$!

$!为最近一个后台执行的异步命令的进程 ID。

    $ firefox &
    [1] 11064

    $ echo $!
    11064


上面例子中，firefox是后台运行的命令，$!返回该命令的进程 ID。

5）$0

$0为当前 Shell 的名称（在命令行直接执行时）或者脚本名（在脚本中执行时）。

    $ echo $0
    >> bash
上面例子中，$0返回当前运行的是 Bash。

### bash的默认值设置：
Bash 提供四个特殊语法，跟变量的默认值有关，目的是保证变量不为空。

    ${varname:-word}
上面语法的含义是，如果变量varname存在且不为空，则返回它的值，否则返回word。它的目的是返回一个默认值，比如${count:-0}表示变量count不存在时返回0。

${varname:=word}
上面语法的含义是，如果变量varname存在且不为空，则返回它的值，否则将它设为word，并且返回word。它的目的是设置变量的默认值，比如${count:=0}表示变量count不存在时返回0，且将count设为0。

    ${varname:+word}
上面语法的含义是，如果变量名存在且不为空，则返回word，否则返回空值。它的目的是测试变量是否存在，比如${count:+1}表示变量count存在时返回1（表示true），否则返回空值。

    ${varname:?message}
上面语法的含义是，如果变量varname存在且不为空，则返回它的值，否则打印出varname: message，并中断脚本的执行。如果省略了message，则输出默认的信息“parameter null or not set.”。它的目的是防止变量未定义，比如${count:?"undefined!"}表示变量count未定义时就中断执行，抛出错误，返回给定的报错信息undefined!。

上面四种语法如果用在脚本中，变量名的部分可以用到数字1到9，表示脚本的参数。

    filename=${1:?"filename missing."}
上面代码出现在脚本中，1表示脚本的第一个参数。如果该参数不存在，就退出脚本并报错。

### declare命令

declare命令可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。

    declare OPTION VARIABLE=value

declare命令的主要参数（OPTION）如下。

+ -a：声明数组变量。
+ -f：输出所有函数定义。
+ -F：输出所有函数名。
+ -i：声明整数变量。
+ -l：声明变量为小写字母。
+ -p：查看变量信息。
+ -r：声明只读变量。
+ -u：声明变量为大写字母。
+ -x：该变量输出为环境变量。

-i:

    $ declare -i val1=12 val2=5
    $ declare -i result
    $ result=val1*val2
    $ echo $result
    >> 60

-x:

    $ declare -x foo
    # 等同于
    $ export foo

-r:

    $ declare -r bar=1

    $ bar=2
    bash: bar：只读变量
    $ echo $?
    1

    $ unset bar
    bash: bar：只读变量
    $ echo $?
    1

-u:

    $ declare -u foo
    $ foo=upper
    $ echo $foo
    UPPER

-l:

    $ declare -l bar
    $ bar=LOWER
    $ echo $bar
    lower

-p:

    $ foo=hello
    $ declare -p foo
    declare -- foo="hello"
    $ declare -p bar
    bar：未找到

## 4 字符串操作
### 4.1 字符串长度
获取字符串长度的语法如下以及例子：

    $ myPath=/home/cam/book/long.file.name
    $ echo ${#myPath}
    >> 29

$注意$：大括号{}是必需的，否则 Bash 会将$#理解成脚本的参数个数，将变量名理解成文本。

### 4.2 子字符串获取
返回变量$varname的子字符串，从位置offset开始（从0开始计算），长度为length。

    $ count=frogfootman
    $ echo ${count:4:4}
    foot

上面例子返回字符串frogfootman从4号位置开始的长度为4的子字符串foot。

这种语法不能直接操作字符串，只能通过变量来读取字符串，并且不会改变原始字符串。变量前面的美元符号可以省略。

如果省略length，则从位置offset开始，一直返回到字符串的结尾。

    $ count=frogfootman
    $ echo ${count:4}
    footman
上面例子是返回变量count从4号位置一直到结尾的子字符串。

如果offset为负值，表示从字符串的末尾开始算起。注意，负数前面必须有一个空格， 以防止与${variable:-word}的变量的设置默认值语法混淆。这时，如果还指定length，则length不能小于零。

    $ foo="This string is long."
    $ echo ${foo: -5}
    long.
    $ echo ${foo: -5:2}
    lo
上面例子中，offset为-5，表示从倒数第5个字符开始截取，所以返回long.。如果指定长度为2，则返回lo。

### 4.3 搜索和替换

Bash 提供字符串搜索和替换的多种方法。

（1）字符串头部的模式匹配。

以下两种语法可以检查字符串开头，是否匹配给定的模式。如果匹配成功，就删除匹配的部分，返回剩下的部分。原始变量不会发生变化。

    # 如果 pattern 匹配变量 variable 的开头，
    # 删除最短匹配（非贪婪匹配）的部分，返回剩余部分
    ${variable#pattern}

    # 如果 pattern 匹配变量 variable 的开头，
    # 删除最长匹配（贪婪匹配）的部分，返回剩余部分
    ${variable##pattern}

上面两种语法会删除变量字符串开头的匹配部分（将其替换为空），返回剩下的部分。区别是一个是最短匹配（又称非贪婪匹配），另一个是最长匹配（又称贪婪匹配）。

匹配模式pattern可以使用*、?、[]等通配符。

    $ myPath=/home/cam/book/long.file.name

    $ echo ${myPath#/*/}
    cam/book/long.file.name

    $ echo ${myPath##/*/}
    long.file.name
上面例子中，匹配的模式是/*/，其中*可以匹配任意数量的字符，所以最短匹配是/home/，最长匹配是/home/cam/book/。

下面写法可以删除文件路径的目录部分，只留下文件名。

    $ path=/home/cam/book/long.file.name

    $ echo ${path##*/}
    long.file.name
上面例子中，模式*/匹配目录部分，所以只返回文件名。

下面再看一个例子。

    $ phone="555-456-1414"
    $ echo ${phone#*-}
    456-1414
    $ echo ${phone##*-}
    1414
如果匹配不成功，则返回原始字符串。

    $ phone="555-456-1414"
    $ echo ${phone#444}
    555-456-1414
上面例子中，原始字符串里面无法匹配模式444，所以原样返回。

如果要将头部匹配的部分，替换成其他内容，采用下面的写法。

    # 模式必须出现在字符串的开头
    ${variable/#pattern/string}

    # 示例
    $ foo=JPG.JPG
    $ echo ${foo/#JPG/jpg}
    jpg.JPG
上面例子中，被替换的JPG必须出现在字符串头部，所以返回jpg.JPG。

（2）字符串尾部的模式匹配。

以下两种语法可以检查字符串结尾，是否匹配给定的模式。如果匹配成功，就删除匹配的部分，返回剩下的部分。原始变量不会发生变化。

    # 如果 pattern 匹配变量 variable 的结尾，
    # 删除最短匹配（非贪婪匹配）的部分，返回剩余部分
    ${variable%pattern}

    # 如果 pattern 匹配变量 variable 的结尾，
    # 删除最长匹配（贪婪匹配）的部分，返回剩余部分
    ${variable%%pattern}
上面两种语法会删除变量字符串结尾的匹配部分（将其替换为空），返回剩下的部分。区别是一个是最短匹配（又称非贪婪匹配），另一个是最长匹配（又称贪婪匹配）。

    $ path=/home/cam/book/long.file.name

    $ echo ${path%.*}
    /home/cam/book/long.file

    $ echo ${path%%.*}
    /home/cam/book/long
上面例子中，匹配模式是.*，其中*可以匹配任意数量的字符，所以最短匹配是.name，最长匹配是.file.name。

下面写法可以删除路径的文件名部分，只留下目录部分。

    $ path=/home/cam/book/long.file.name

    $ echo ${path%/*}
    /home/cam/book
上面例子中，模式/*匹配文件名部分，所以只返回目录部分。

下面的写法可以替换文件的后缀名。

    $ file=foo.png
    $ echo ${file%.png}.jpg
    foo.jpg
上面的例子将文件的后缀名，从.png改成了.jpg。

下面再看一个例子。

    $ phone="555-456-1414"
    $ echo ${phone%-*}
    555-456
    $ echo ${phone%%-*}
    555
如果匹配不成功，则返回原始字符串。

如果要将尾部匹配的部分，替换成其他内容，采用下面的写法。

    # 模式必须出现在字符串的结尾
    ${variable/%pattern/string}

    # 示例
    $ foo=JPG.JPG
    $ echo ${foo/%JPG/jpg}
    JPG.jpg
上面例子中，被替换的JPG必须出现在字符串尾部，所以返回JPG.jpg。

（3）任意位置的模式匹配。

以下两种语法可以检查字符串内部，是否匹配给定的模式。如果匹配成功，就删除匹配的部分，换成其他的字符串返回。原始变量不会发生变化。

    # 如果 pattern 匹配变量 variable 的一部分，
    # 最长匹配（贪婪匹配）的那部分被 string 替换，但仅替换第一个匹配
    ${variable/pattern/string}

    # 如果 pattern 匹配变量 variable 的一部分，
    # 最长匹配（贪婪匹配）的那部分被 string 替换，所有匹配都替换
    ${variable//pattern/string}
上面两种语法都是最长匹配（贪婪匹配）下的替换，区别是前一个语法仅仅替换第一个匹配，后一个语法替换所有匹配。

    $ path=/home/cam/foo/foo.name

    $ echo ${path/foo/bar}
    /home/cam/bar/foo.name

    $ echo ${path//foo/bar}
    /home/cam/bar/bar.name
上面例子中，前一个命令只替换了第一个foo，后一个命令将两个foo都替换了。

下面的例子将分隔符从:换成换行符。

    $ echo -e ${PATH//:/'\n'}
    /usr/local/bin
    /usr/bin
    /bin
    ...
上面例子中，echo命令的-e参数，表示将替换后的字符串的\n字符，解释为换行符。

模式部分可以使用通配符。

    $ phone="555-456-1414"
    $ echo ${phone/5?4/-}
    55-56-1414      
上面的例子将5-4替换成-。

如果省略了string部分，那么就相当于匹配的部分替换成空字符串，即删除匹配的部分。

    $ path=/home/cam/foo/foo.name

    $ echo ${path/.*/}
    /home/cam/foo/foo
上面例子中，第二个斜杠后面的string部分省略了，所以模式.*匹配的部分.name被删除后返回。

前面提到过，这个语法还有两种扩展形式。

    # 模式必须出现在字符串的开头
    ${variable/#pattern/string}

    # 模式必须出现在字符串的结尾
    ${variable/%pattern/string}


### 4.4 改变大小写
下面的语法可以改变变量的大小写。

    # 转为大写
    ${varname^^}

    # 转为小写
    ${varname,,}


## 5 算数表达式

### 5.1 表达式
((...))语法可以进行整数的算术运算。

    $ ((foo = 5 + 5))
    $ echo $foo
    10

这个语法不返回值，命令执行的结果根据算术运算的结果而定。只要算术结果不是0，命令就算执行成功。

    $ (( 3 + 2 ))
    $ echo $?
    0
上面例子中，3 + 2的结果是5，命令就算执行成功，环境变量$?为0。

### 5.2 进制
Bash 的数值默认都是十进制，但是在算术表达式中，也可以使用其他进制。

number：没有任何特殊表示法的数字是十进制数（以10为底）。

0number：八进制数。

0xnumber：十六进制数。

### 5.3 位运算操作
$((...))支持以下的二进制位运算符。

<<：位左移运算，把一个数字的所有位向左移动指定的位。

\>>：位右移运算，把一个数字的所有位向右移动指定的位。

&：位的“与”运算，对两个数字的所有位执行一个AND操作。

|：位的“或”运算，对两个数字的所有位执行一个OR操作。

~：位的“否”运算，对一个数字的所有位取反。

!：逻辑“否”运算

^：位的异或运算（exclusive or），对两个数字的所有位执行一个异或操作。

    $ echo $((17&3))
    1
    $ echo $((17|3))
    19
    $ echo $((17^3))
    18

### 5.4 逻辑运算
$((...))支持以下的逻辑运算符。

+ <：小于
+ \>：大于
+ <=：小于或相等
+ \>=：大于或相等
+ ==：相等
+ !=：不相等
+ &&：逻辑与
+ ||：逻辑或
+ expr1?expr2:expr3：三元条件运算符。若表达式expr1的计算结果为非零值（算术真），则执行表达式expr2，否则执行表达式expr3。

如果逻辑表达式为真，返回1，否则返回0。

    $ echo $((3 > 2))
    1
    $ echo $(( (3 > 2) || (4 <= 1) ))
    1

### 5.5 求值运算
逗号,在$((...))内部是求值运算符，执行前后两个表达式，并返回后一个表达式的值。

    $ echo $((foo = 1 + 2, 3 * 4))
    12
    $ echo $foo
    3
上面例子中，逗号前后两个表达式都会执行，然后返回后一个表达式的值12。

### 5.6 expr指令
expr命令支持算术运算，可以不使用((...))语法。

    $ expr 3 + 2
    5
expr命令支持变量替换。

    $ foo=3
    $ expr $foo + 2
    5
expr命令也不支持非整数参数。

    $ expr 3.5 + 2
    expr: 非整数参数
上面例子中，如果有非整数的运算，expr命令就报错了。


## 6 堆栈操作
# 6.1 cd
Bash 可以记忆用户进入过的目录。默认情况下，只记忆前一次所在的目录，cd -命令可以返回前一次的目录。

    # 当前目录是 /path/to/foo
    $ cd bar

    # 重新回到 /path/to/foo
    $ cd -
上面例子中，用户原来所在的目录是/path/to/foo，进入子目录bar以后，使用cd -可以回到原来的目录。

# 6.2 push/pop
如果希望记忆多重目录，可以使用pushd命令和popd命令。它们用来操作目录堆栈。

pushd命令的用法类似cd命令，可以进入指定的目录。

$ pushd dirname
上面命令会进入目录dirname，并将该目录放入堆栈。

第一次使用pushd命令时，会将当前目录先放入堆栈，然后将所要进入的目录也放入堆栈，位置在前一个记录的上方。以后每次使用pushd命令，都会将所要进入的目录，放在堆栈的顶部。

popd命令不带有参数时，会移除堆栈的顶部记录，并进入新的堆栈顶部目录（即原来的第二条目录）。

下面是一个例子。

    # 当前处在主目录，堆栈为空
    $ pwd
    /home/me

    # 进入 /home/me/foo
    # 当前堆栈为 /home/me/foo /home/me
    $ pushd ~/foo

    # 进入 /etc
    # 当前堆栈为 /etc /home/me/foo /home/me
    $ pushd /etc

    # 进入 /home/me/foo
    # 当前堆栈为 /home/me/foo /home/me
    $ popd

    # 进入 /home/me
    # 当前堆栈为 /home/me
    $ popd

    # 目录不变，当前堆栈为空
    $ popd
这两个命令的参数如下。

（1）-n 参数

-n的参数表示仅操作堆栈，不改变目录。

    $ popd -n
上面的命令仅删除堆栈顶部的记录，不改变目录，执行完成后还停留在当前目录。

（2）整数参数

这两个命令还可以接受一个整数作为参数，该整数表示堆栈中指定位置的记录（从0开始），作为操作对象。这时不会切换目录。

    # 从栈顶算起的3号目录（从0开始），移动到栈顶
    $ pushd +3

    # 从栈底算起的3号目录（从0开始），移动到栈顶
    $ pushd -3

    # 删除从栈顶算起的3号目录（从0开始）
    $ popd +3

    # 删除从栈底算起的3号目录（从0开始）
    $ popd -3
上面例子的整数编号都是从0开始计算，popd +0是删除第一个目录，popd +1是删除第二个，popd -0是删除最后一个目录，，popd -1是删除倒数第二个。

（3）目录参数

pushd可以接受一个目录作为参数，表示将该目录放到堆栈顶部，并进入该目录。

    $ pushd dir
popd没有这个参数。

### 6.3 dir命令

dirs命令可以显示目录堆栈的内容，一般用来查看pushd和popd操作后的结果。

    $ dirs
它有以下参数。

+ -c：清空目录栈。
+ -l：用户主目录不显示波浪号前缀，而打印完整的目录。
+ -p：每行一个条目打印目录栈，默认是打印在一行。
+ -v：每行一个条目，每个条目之前显示位置编号（从0开始）。
+ +N：N为整数，表示显示堆顶算起的第 N 个目录，从零开始。
+ -N：N为整数，表示显示堆底算起的第 N 个目录，从零开始。

## 7 脚本文件
### 1 Shebang 行
脚本的第一行通常是指定解释器，即这个脚本必须通过什么解释器执行。这一行以#!字符开头，这个字符称为 Shebang，所以这一行就叫做 Shebang 行。

#!后面就是脚本解释器的位置，Bash 脚本的解释器一般是/bin/sh或/bin/bash。

<!-- #!/bin/sh
# 或者
#!/bin/bash
#!与脚本解释器之间有没有空格，都是可以的。 -->

如果 Bash 解释器不放在目录/bin，脚本就无法执行了。为了保险，可以写成下面这样。

    #!/usr/bin/env bash
    上面命令使用env命令（这个命令总是在/usr/bin目录），返回 Bash 可执行文件的位置。env命令的详细介绍，请看后文。

Shebang 行不是必需的，但是建议加上这行。如果缺少该行，就需要手动将脚本传给解释器。举例来说，脚本是script.sh，有 Shebang 行的时候，可以直接调用执行。

    $ ./script.sh
上面例子中，script.sh是脚本文件名。脚本通常使用.sh后缀名，不过这不是必需的。

如果没有 Shebang 行，就只能手动将脚本传给解释器来执行。

    $ /bin/sh ./script.sh
    # 或者
    $ bash ./script.sh

### 7.2 执行权限

可以使用下面的命令，赋予脚本执行权限。

    # 给所有用户执行权限
    $ chmod +x script.sh

    # 给所有用户读权限和执行权限
    $ chmod +rx script.sh
    # 或者
    $ chmod 755 script.sh

    # 只给脚本拥有者读权限和执行权限
    $ chmod u+rx script.sh
脚本的权限通常设为755（拥有者有所有权限，其他人有读和执行权限）或者700（只有拥有者可以执行）。

除了执行权限，脚本调用时，一般需要指定脚本的路径（比如path/script.sh）。如果将脚本放在环境变量$PATH指定的目录中，就不需要指定路径了。因为 Bash 会自动到这些目录中，寻找是否存在同名的可执行文件。

建议在主目录新建一个~/bin子目录，专门存放可执行脚本，然后把~/bin加入$PATH。

    export PATH=$PATH:~/bin
上面命令改变环境变量 $PATH，将~/bin添加到 $PATH的末尾。可以将这一行加到~/.bashrc文件里面，然后重新加载一次.bashrc，这个配置就可以生效了。

## 7.3 env
env命令总是指向/usr/bin/env文件，或者说，这个二进制文件总是在目录/usr/bin。

#!/usr/bin/env NAME 这个语法的意思是，让 Shell 查找$PATH环境变量里面第一个匹配的NAME。如果你不知道某个命令的具体路径，或者希望兼容其他用户的机器，这样的写法就很有用。

/usr/bin/env bash的意思就是，返回bash可执行文件的位置，前提是bash的路径是在$PATH里面。其他脚本文件也可以使用这个命令。比如 Node.js 脚本的 Shebang 行，可以写成下面这样。

    #!/usr/bin/env node
env命令的参数如下。

-i, --ignore-environment：不带环境变量启动。

-u, --unset=NAME：从环境变量中删除一个变量。

--help：显示帮助。

--version：输出版本信息。
下面是一个例子，新建一个不带任何环境变量的 Shell。
    
    $ env -i /bin/sh

### 7.4

调用脚本的时候，脚本文件名后面可以带有参数。

    $ script.sh word1 word2 word3
上面例子中，script.sh是一个脚本文件，word1、word2和word3是三个参数。

脚本文件内部，可以使用特殊变量，引用这些参数。

+ $0：脚本文件名，即script.sh。
+ $1~$9：对应脚本的第一个参数到第九个参数。
+ $#：参数的总数。
+ $@：全部的参数，参数之间使用空格分隔。
+ $*：全部的参数，参数之间使用变量 $IFS值的第一个字符分隔，默认为空格，但是可以自定义。
  
eg：新建一个脚本script.sh，

    #!/bin/bash
    # script.sh

    echo "全部参数：" $@
    echo "命令行参数数量：" $#
    echo '$0 = ' $0
    echo '$1 = ' $1
    echo '$2 = ' $2
    echo '$3 = ' $3
执行如下命令行：

    $ ./script.sh a b c
    全部参数：a b c
    命令行参数数量：3
    $0 =  script.sh
    $1 =  a
    $2 =  b
    $3 =  c
    用户可以输入任意
且我们可以使用如下命令行查看所有参数

    #!/bin/bash

    for i in "$@"; do
        echo $i
    done

### 7.5 shift命令

shift命令可以改变脚本参数，每次执行都会移除脚本当前的第一个参数（$1），使得后面的参数向前一位，即$2变成$1、$3变成$2、$4变成$3，以此类推。

while循环结合shift命令，也可以读取每一个参数。

    #!/bin/bash

    echo "一共输入了 $# 个参数"

    while [ "$1" != "" ]; do
      echo "剩下 $# 个参数"
      echo "参数：$1"
      shift
    done

### 7.6 getopts命令
getopts命令用在脚本内部，可以解析复杂的脚本命令行参数，通常与while循环一起使用，取出脚本所有的带有前置连词线（-）的参数。

    getopts optstring name
它带有两个参数。第一个参数optstring是字符串，给出脚本所有的连词线参数。比如，某个脚本可以有三个配置项参数-l、-h、-a，其中只有-a可以带有参数值，而-l和-h是开关参数，那么getopts的第一个参数写成lha:，顺序不重要。注意，a后面有一个冒号，表示该参数带有参数值，getopts规定带有参数值的配置项参数，后面必须带有一个冒号（:）。getopts的第二个参数name是一个变量名，用来保存当前取到的配置项参数，即l、h或a。

    while getopts 'lha:' OPTION; do
      case "$OPTION" in
        l)
          echo "linuxconfig"
          ;;

        h)
          echo "h stands for h"
          ;;

        a)
          avalue="$OPTARG"
          echo "The value provided is $OPTARG"
          ;;
        ?)
          echo "script usage: $(basename $0) [-l] [-h] [-a somevalue]" >&2
          exit 1
          ;;
      esac
    done
    shift "$(($OPTIND - 1))"

### 7.7 配置项参数终止符
    $ myPath="~/docs"
    $ ls -- $myPath
例子中，--强制变量$myPath只能当作实体参数（即路径名）解释。

如果变量不是路径名，就会报错。

    $ myPath="-l"
    $ ls -- $myPath
ls: 无法访问'-l': 没有那个文件或目录
上面例子中，变量myPath的值为-l，不是路径。但是，--强制$myPath只能作为路径解释，导致报错“不存在该路径”。

### 7.8 exit命令
exit命令用于终止当前脚本的执行，并向 Shell 返回一个退出值。

    $ exit
退出时，脚本会返回一个退出值。脚本的退出值，0表示正常，1表示发生错误，2表示用法不对，126表示不是可执行脚本，127表示命令没有发现。如果脚本被信号N终止，则退出值为128 + N。简单来说，只要退出值非0，就认为执行出错。

    if [ $(id -u) != "0" ]; then
      echo "根用户才能执行当前脚本"
      exit 1
    fi
id -u命令返回用户的 ID，一旦用户的 ID 不等于0（根用户的 ID），脚本就会退出，并且退出码为1，表示运行失败  
exit与return命令的差别是，return命令是函数的退出，并返回一个值给调用者，脚本依然执行。exit是整个脚本的退出，如果在函数之中调用exit，则退出函数，并终止脚本执行。

### 7.9 命令执行结果
命令执行结束后，会有一个返回值。0表示执行成功，非0（通常是1）表示执行失败。环境变量$?可以读取前一个命令的返回值。

    cd $some_directory
    if [ "$?" = "0" ]; then
      rm *
    else
      echo "无法切换目录！" 1>&2
      exit 1
    fi

可以使用更简单的写法：

    # 第一步执行成功，才会执行第二步
    cd $some_directory && rm *

    # 第一步执行失败，才会执行第二步
    cd $some_directory || exit 1

### 7.10 source

source命令用于执行一个脚本，通常用于重新加载一个配置文件。

    $ source .bashrc    
source命令最大的特点是在当前 Shell 执行脚本，不像直接执行脚本时，会新建一个子 Shell。所以，source命令执行脚本时，不需要export变量。

source命令的另一个用途，是在脚本内部加载外部库。

    #!/bin/bash

    source ./lib.sh

    function_from_lib
上面脚本在内部使用source命令加载了一个外部库，然后就可以在脚本里面，使用这个外部库定义的函数。

source有一个简写形式，可以使用一个点（.）来表示。

    $ . .bashrc

### 7.11 alias
alias命令用来为一个命令指定别名，这样更便于记忆。下面是alias的格式。

    alias NAME=DEFINITION
上面命令中，NAME是别名的名称，DEFINITION是别名对应的原始命令。注意，等号两侧不能有空格，否则会报错。

    $ alias today='date +"%A, %B %-d, %Y"'
    $ today
    >> 星期一, 一月 6, 2020
上面例子中，别名定义了echo命令的前两个参数，等同于修改了echo命令的默认行为。

指定别名以后，就可以像使用其他命令一样使用别名。一般来说，都会把常用的别名写在~/.bashrc的末尾。另外，只能为命令定义别名，为其他部分（比如很长的路径）定义别名是无效的。

直接调用alias命令，可以显示所有别名。
unalias命令可以解除别名。



