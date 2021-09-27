# Linux

## Linux后台高频面试题

### 如何查看进程打开的文件

-  **lsof 查看进程打开那些文件 或者 查看文件给那个进程使用**

```
1、查看进程打开的文件：
  1）pidof programe-name(获得想了解的进程(programe-name)的PID)
 或ps -aux|grep programe-name(获得想了解的进程(programe-name)的PID)
  找出进程的PID
  2）cd /proc/$PID/fd（会看见文件描述符）
  3）ls -l
     得到文件描述符指向的实际文件,即当前进程打开的文件

2、查看进程打开的文件：
 1）获得想了解的进程的PID方法同上
 2）lsof -c programe-name
     或lsof -p $PID

3、查看文件对应的进程：
 lsof file-name

4、lsof命令用法：
 lsof -c abc 显示abc进程现在打开的文件 
  lsof abc 显示开启文件abc的进程 
  lsof -i :22 显示22端口现在运行什么程序 
  lsof -g gid 显示归属gid的进程情况 
  lsof +d /usr/local/ 显示目录下被进程开启的文件 
  lsof +D /usr/local/ 同上，但是会搜索目录下的目录，时间较长 
  lsof -d 4 显示使用fd为4的进程 
  lsof -i 用以显示符合条件的进程情况 
  lsof -s 列出打开文件的大小，如果没有大小，则留下空白
  lsof -u username 以UID，列出打开的文件
  
5、查看网络状态：
lsof -Pnl +M -i4 显示ipv4服务及监听端情况
netstat -anp 所有监听端口及对应的进程
netstat -tlnp 功能同上
```

### 介绍下nm与ldd命令

-  Linux程序分析工具:ldd和nm

```
ldd是用来分析程序运行时需要依赖的动态链接库的工具，nm是用来查看指定程序中的符号表信息的工具。

1 ldd
格式：ldd [options] file   

功能：列出file运行所需的共享库

参数：

      -d    执行重定位并报告所有丢失的函数

      -r    执行对函数和对象的重定位并报告丢失的任何函数或对象

tanghuaming@Thm:~/Documents/sys_programming$ whereis ldd
ldd: /usr/bin/ldd /usr/share/man/man1/ldd.1.gz
tanghuaming@Thm:~/Documents/sys_programming$ ll /usr/bin/ldd
-rwxr-xr-x 1 root root 5420  3月 26 14:59 /usr/bin/ldd*

xiaomanon@xiaomanon-machine:~/Documents/c_code$ ldd tooltest
    linux-gate.so.1 =>  (0xb775b000)
    libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7595000)
    /lib/ld-linux.so.2 (0xb775c000)
我们可以将ldd的输出结果分为3列来看：

■ 第一列：程序需要依赖什么库

■ 第二列：系统提供的与程序需要的库对应的库名称

■ 第三列：依赖库加载的开始地址

通过上面的这些信息，我们可以总结出下面的用途：

(1) 通过对比第一列和第二列，我们可以知道程序需要的动态链接库和系统实际提供的是否相比配。

(2) 通过第三列，我们可以知道当前动态链接库中的符号在进程地址空间中的起始位置。

2 nm
格式：nm [options] file   

功能：列出file中的所有符号

参数：

     -C   将符号转化为用户级的名字

     -s   当用于.a文件即静态库时，输出把符号名映射到定义该符号的模块或成员名的索引

     -u   显示在file外定义的符号或没有定义的符号

     -l   显示每个符号的行号，或为定义符号的重定义项
     
xiaomanon@xiaomanon-machine:~/Documents/c_code$ nm tooltest
0804a038 B __bss_start
0804a038 b completed.6590
0804a02c D __data_start
0804a02c W data_start
08048450 t deregister_tm_clones
....

上面便是tooltest这个程序中所有的符号，首先介绍一下上面输出内容的格式：

■ 第一列：当前符号的地址。

■ 第二列：当前符号的类型（关于类型的说明，可以查看手册页man nm详细阅读）。

■ 第三列：当前符号的名称。

使用nm主要有一下几个方面的帮助：

(1) 判断指定的程序中有没有指定的符号，比较常用的方式为：nm –C program | grep symbol

(2) 解决程序编译时undefined reference的错误，以及multiple definition的错误。

(3) 查看某个符号的地址，以及在进程空间的大概位置（.bss, .data, .text段，具体可以通过第二列的类型来判断）。

A : 该符号的值是绝对的，在以后的链接过程中，不允许进行改变。这样的符号值，常常出现在中断向量表中，例如用符号来表示各个中断向量函数在中断向量表中的位置。

B : 该符号的值出现在非初始化数据段(.bss)中。例如，在一个文件中定义全局static int test。则该符号test的类型为b，位于bss section中。其值表示该符号在bss段中的偏移。一般而言，bss段分配于RAM中 。

C : 该符号为common。common symbol是未初始话数据段。该符号没有包含于一个普通section中。只有在链接过程中才进行分配。符号的值表示该符号需要的字节数。例如在一个c文件中，定义int test，并且该符号在别的地方会被引用，则该符号类型即为C。否则其类型为B。

D : 该符号位于初始化数据段中。一般来说，分配到.data section中。例如定义全局int baud_table[5] = {9600, 19200, 38400, 57600, 115200}，则会分配于初始化数据段中。

G : 该符号也位于初始化数据段中。主要用于small object提高访问small data object的一种方式。

I : 该符号是对另一个符号的间接引用。

N : 该符号是一个debugging符号。

R : 该符号位于只读数据段。例如定义全局const int test[] = {123, 123};则test就是一个只读数据区的符号。注意在cygwin下如果使用gcc直接编译成MZ格式时，源文件中的test对应_test，并且其符号类型为D，即初始化数据段中。但是如果使用m6812-elf-gcc这样的交叉编译工具，源文件中的test对应目标文件的test,即没有添加下划线，并且其符号类型为R。一般而言，位于rodata section。值得注意的是，如果在一个函数中定义const char *test = “abc”, const char test_int = 3。使用nm都不会得到符号信息，但是字符串“abc”分配于只读存储器中，test在rodata section中，大小为4。

S : 符号位于非初始化数据段，用于small object。

T : 该符号位于代码段text section。

U : 该符号在当前文件中是未定义的，即该符号的定义在别的文件中。例如，当前文件调用另一个文件中定义的函数，在这个被调用的函数在当前就是未定义的；但是在定义它的文件中类型是T。但是对于全局变量来说，在定义它的文件中，其符号类型为C，在使用它的文件中，其类型为U。

V : 该符号是一个weak object。

W : The symbol is a weak symbol that has not been specifically tagged as a weak object symbol.

- : 该符号是a.out格式文件中的stabs symbol。

? : 该符号类型没有定义。
```

### shell命令查内存，端口 ，io访问量，读写速率

```

内存：top

磁盘存储： df -lh（文件系统du -h）

端口占用：netstat -tunlp

进程：ps -aux | grep 进程名

io读写：
	查看磁盘读写状态： iostat -x；
	查看哪些进程在读写磁盘io，可以查看哪些进程导致磁盘io很忙：iotop 使用磁盘io越多的排在越前。 
```



### awk grep sed具体应用

```c++
grep：文本搜索
grep ‘w[ea]ll’ file_name

在file_name文件中找到wall 或者是well 所在的所有行并显示

grep ‘w[^e]ll’ file_name

在file_name文件中找到”非well” 所在的所有行并显示

grep ‘^The’ file_name

在file_name文件中找到以The开头的所有行并显示（请与上一条命令进行区别）

sed：数据的替换，删除，增加，选取（以行为单位）
sed ‘2,4d’ file_name

删除file_name文件的2到4行

awk：以字段为单位进行处理(其实是把一行的数据分割，然后进行处理)
0代表一整行的数据   1 代表第一个字段，用人的话来说就是第一列的数据

NR 目前处理的是第几行的数据

命令格式 ：awk ‘条件{命令1} 条件{命令2}…’ file_name
注：print 默认带有换行符，printf 没有

awk ‘NR<6{print 1"\t"2 }’ file_name

把file_name 文件中的前五行的第一列，第二列的数据列出来 （以[tab]或空格键分隔）

总结
三个命令的运用形式
grep ‘字符’ 文件
sed ‘命令’ 文件
awk ‘条件{命令}’ 文件
单引号内就是正则表达式的用法
```



### 硬链接与软连接，目录可不可以用硬链接（不能）

```
1、硬链接
开始之前，先解释一个概念，叫做索引节点（Inode）。

在Linux的文件系统中，保存在磁盘分区中的文件，不管是什么类型，系统都会给它分配一个编号，这个编号被称为索引节点编号（Inode Index），它是该文件或者目录在linux文件系统中的唯一标识。有了这个编号值，就可以查到该文件的详细内容。

同时，Linux系统还规定，可以允许多个文件名同时指向同一个索引节点（Inode），这就是硬链接。这样设计有一个好处就是，只要文件的索引节点还存在一个以上的链接，删除其中一个链接并不影响索引节点本身和其他的链接（也就是说该文件的实体并未删除），而只有当最后一个链接被删除后，且此时有新数据要存储到磁盘上，那么被删除的文件的数据块及目录的链接才会被释放，存储空间才会被新数据所覆盖。因此，该机制可以有效的防止误删操作。

硬链接只能在同一类型的文件系统中进行链接，不能跨文件系统。同时它只能对文件进行链接，不能链接目录。

2、软链接
软链接（也叫符号链接），类似于windows系统中的快捷方式，与硬链接不同，软链接就是一个普通文件，只是数据块内容有点特殊，文件用户数据块中存放的内容是另一文件的路径名的指向，通过这个方式可以快速定位到软连接所指向的源文件实体。

软链接常用来解决空间不足的问题，比如某个文件文件系统空间已经用完了，但是现在必须在该文件系统下创建一个新的目录并存储大量的文件，那么可以把另一个剩余空间较多的文件系统中的目录链接到该文件系统中。

软链接可以跨文件系统而链接，也可以同时对文件或目录进行链接。

3、二者区别
软链接以存放另一个文件的路径的形式存在，硬链接以文件副本的形式存在；
软链接可以跨不同的文件系统而链接，硬链接不可以；
软链接可以对目录进行链接，而硬链接不可以；
软链接可以对一个不存在的文件名进行链接，硬链接必须要有源文件。
删除软链接并不影响被指向的文件，但若被指向的原文件被删除，则相关软连接就变成了死链接。删除硬链接的话，只要索引节点的个数不为零，则不会对原始文件造成任何影响；
注意：不论是硬链接或软链接都不会将原本的目标文件完全复制一份，而只会占用非常少量的存储空间。

二、创建方式（ln命令）
软链接和硬链接都是通过ln命令来创建，只是参数不同。命令格式如下：

ln 参数 源文件或目录 目标文件或目录

注意：源目录和目标目录都必须是绝对路径！

参数：

-i 交互模式，文件存在则提示用户是否覆盖；
-s 软链接（符号链接）；
-d 允许超级用户制作目录的硬链接；
-b 删除，覆盖以前建立的链接；
-f 强制执行；
-n 把符号链接视为一般目录；
-v 显示详细的处理过程；
所以，总结起来就是：

创建软链接 （符号链接）使用：ln -s source target
创建硬链接 （实体链接）使用：ln source target
```



### 常见命令netstat iptables tcpdump top

```
1. netstat
netstat用于显示网络状态，使用netstat可以让你很轻松的获得整个linux的网络情况。

2. top
top命令用来监控linux的系统状况，比如：cpu、内存的使用。

3. lsof
lsof是一个列出当前系统打开文件的工具，包括普通文件、目录、网络连接、硬件等等。

4. tcpdump
root权限下使用的抓包工具，只能抓取流经本机的数据包

5. iptables 是 Linux 中重要的访问控制手段，是俗称的 Linux 防火墙系统的重要组成部分。这里记录了iptables 防火墙规则的一些常用的操作指令。
```



### makefile介绍下(cmake介绍下)

```
但如果源文件太多，一个一个编译那得多麻烦啊？于是人们想到，为啥不设计一种类似批处理的程序，来批处理编译源文件呢？

于是就有了make工具，它是一个自动化编译工具，你可以使用一条命令实现完全编译。但是你需要编写一个规则文件，make依据它来批处理编译，这个文件就是makefile，所以编写makefile文件也是一个程序员所必备的技能。

对于一个大工程，编写makefile实在是件复杂的事，于是人们又想，为什么不设计一个工具，读入所有源文件之后，自动生成makefile呢，于是就出现了cmake工具，它能够输出各种各样的makefile或者project文件,从而帮助程序员减轻负担。但是随之而来也就是编写cmakelist文件，它是cmake所依据的规则。
```



### gdb查看堆栈中所有遍历

```
- 在编译时或在makefile中添加 -g选项（补充调试信息）
- g++ test.c -o test -g
- gdb test
  - list / l : 列出代码
  - run / r : 执行程序
  - break / b : 打断点
  - continue / c : 继续运行
  - next / n : 下一步（单步调试，不进入函数）
  - step / s : 下一步（进入函数）
  - info break / i b : 显示断点信息
  - delete 编号 : 删除某个断点
  - disable / enbale  编号 : 失效/生效某个断点
  - ignore 编号  ：令指令编号失效若干次
  - print / p 表达式 : 打印变量
  - backtrace / bt : 看堆栈
  - display 表达式 ：看执行的信息
  - info display : 获取编号
  - undisplay  编号 : 
  - h 命令名【查命令】
  - x/3xw arr 看内存
```



### gdb查看shared_ptr 指向的内容

```
boost和C++11中的智能指针shared_ptr很好用，但是在linux调试代码时发现，只能指针无法用gdb查看指针指向的变量，下面介绍两个方法查看只能指针指向的变量


1.shared_ptr有一个get方法，返回shared_ptr保存的真正的ptr，显示调用一下get()即可当做正常指针用了

2.有时候调用get方法无非获取到保存的指针，gdb提示init failed，这时候就只能用第二种方法了，如下例子：

class TestClassA

{
public:

  string name;

};

typedef std::shared_ptr<TestClassA> TestClassAPtr;


void testFunc(TestClassAPtr test_ptr)

{
.....调试到这里了

}


想在testFunc中看一下test_ptr->name

但是直接在gdb中输入p test_ptr->name看不到结果

这样输入：

p ((TestClassA*)test_ptr)->name

即可看到name的值

原因如下：shared_ptr保存了真正的指针在此称作real_ptr，在shared_ptr中real_ptr是第一个变量，所以智能指针test_ptr和首地址即为real_ptr，利用强转将test_ptr转为TestClassA*，即可以TestClassA*类型访问到real_ptr了

```



### gdb如何调试多进程多线程

https://blog.csdn.net/zhangye3017/article/details/80382496



### g++和gcc编译出来有什么区别

```
gcc和g++的区别主要是在对cpp文件的编译和链接过程中，因为cpp和c文件中库文件的命名方式不同，那为什么g++既可以编译C又可以编译C++呢，这时因为g++在内部做了处理，默认编译C++程序，但如果遇到C程序，它会直接调用gcc去编译.
```



### 死锁怎么调试

```
编译：

gcc New0001.c -g -lpthread -o a.out

 

这里加上-g是有必要的，加上-g可以产生调试信息，符号信息等。千万不要对生成的a.out文件执行strip命令，strip会导致调试时看不到哪行代码有问题。

 

 

第一种方法：

1.使用gdb a.out(可执行文件)，并输入r命令运行程序

 

gdb a.out

GNU gdb (Ubuntu 7.10-1ubuntu2) 7.10

Copyright (C) 2015 Free Software Foundation, Inc.

License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.  Type "show copying"

and "show warranty" for details.

This GDB was configured as "i686-linux-gnu".

Type "show configuration" for configuration details.

For bug reporting instructions, please see:

<http://www.gnu.org/software/gdb/bugs/>.

Find the GDB manual and other documentation resources online at:

<http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".

Type "apropos word" to search for commands related to "word"...

Reading symbols from a.out...done.

 

 

 

(gdb) r

Starting program: /share/a.out

[Thread debugging using libthread_db enabled]

Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".

[New Thread 0xb7de6b40 (LWP 15436)]

[New Thread 0xb75e5b40 (LWP 15437)]

[New Thread 0xb6de4b40 (LWP 15438)]

[New Thread 0xb65e3b40 (LWP 15439)]

[New Thread 0xb5de2b40 (LWP 15440)]

[Thread 0xb7de6b40 (LWP 15436) exited]

 

2.在运行的过程中按下ctrl + c，

 

^C

Program received signal SIGINT, Interrupt.

0xb7fdbbe8 in __kernel_vsyscall ()


 

3.查看线程栈信息，info stack，这个命令只能查看当前正在运行的某个线程的栈信息

 

(gdb) info stack

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7e9c3e6 in nanosleep () at ../sysdeps/unix/syscall-template.S:81

#2  0xb7e9c1a9 in __sleep (seconds=0) at ../sysdeps/unix/sysv/linux/sleep.c:138

#3  0x08048679 in main () at New0001.c:46

 

4.info threads查看所有线程id，前面有*的，代表正在运行的线程，其他没有*的极有可能是在阻塞或者死锁的。

 

(gdb) info threaads

  Id   Target Id         Frame

  6    Thread 0xb5de2b40 (LWP 15440) "a.out" 0xb7fdbbe8 in __kernel_vsyscall ()

  5    Thread 0xb65e3b40 (LWP 15439) "a.out" 0xb7fdbbe8 in __kernel_vsyscall ()

  4    Thread 0xb6de4b40 (LWP 15438) "a.out" 0xb7fdbbe8 in __kernel_vsyscall ()

  3    Thread 0xb75e5b40 (LWP 15437) "a.out" 0xb7fdbbe8 in __kernel_vsyscall ()

* 1    Thread 0xb7de7700 (LWP 15432) "a.out" 0xb7fdbbe8 in __kernel_vsyscall ()

 

 

5. thread apply all bt （thread apply all  命令，gdb会让所有线程都执行这个命令，比如命令为bt，查看所有线程的具体的栈信息）

 

需要注意的是：如果系统运行着很多线程的时候，不可能使用thread  id（这个id比如上面的1 ,2 ,3, ,4, 5, 6），这样要查到什么时候呢 ，100个线程你还输入100次吗

 

因此最好还是直接使用thread apply all bt

 

 

(gdb)thread apply all bt

 

Thread 6 (Thread 0xb5de2b40 (LWP 15440)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7fb2302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb7fac5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb7faa1aa in start_thread (arg=0xb5de2b40) at pthread_create.c:333

#5  0xb7ed2fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 5 (Thread 0xb65e3b40 (LWP 15439)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7fb2302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb7fac5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb7faa1aa in start_thread (arg=0xb65e3b40) at pthread_create.c:333

#5  0xb7ed2fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 4 (Thread 0xb6de4b40 (LWP 15438)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7fb2302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb7fac5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb7faa1aa in start_thread (arg=0xb6de4b40) at pthread_create.c:333

#5  0xb7ed2fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 3 (Thread 0xb75e5b40 (LWP 15437)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7fb2302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb7fac5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb7faa1aa in start_thread (arg=0xb75e5b40) at pthread_create.c:333

#5  0xb7ed2fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 1 (Thread 0xb7de7700 (LWP 15432)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

---Type <return> to continue, or q <return> to quit---

#1  0xb7e9c3e6 in nanosleep () at ../sysdeps/unix/syscall-template.S:81

#2  0xb7e9c1a9 in __sleep (seconds=0) at ../sysdeps/unix/sysv/linux/sleep.c:138

#3  0x08048679 in main () at New0001.c:46

 

 

6.看到的lock_wait就是被死锁的线程

 

多按照上述步骤运行几次，看到那些线程老是出现lock_wait的，就很明显可能是死锁的线程了。

 

比如线程3吧

Thread 3 (Thread 0xb75e5b40 (LWP 15437)):

#0  0xb7fdbbe8 in __kernel_vsyscall ()

#1  0xb7fb2302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb7fac5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb7faa1aa in start_thread (arg=0xb75e5b40) at pthread_create.c:333

#5  0xb7ed2fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

 

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

就是死锁的位置，可以从这里开始定位代码，看看哪个地方可能没有释放锁。

 


第二种方法

 

先让程序跑起来，打开另外一个会话，通过ps -aux| grep 可执行文件 ，

找到程序的进程号

 

ps -axu | grep a.out

root     15463  0.4  0.1  43320   732 pts/4    Sl+  19:29   0:03 ./a.out

root     15476  0.0  0.3   4540  1864 pts/6    S+   19:44   0:00 grep --color=auto a.out

 

由上可知进程号是 15463

 

 

1.使用gdb  attach  进程号

  或者是进入gdb后， attach 进程号

  或者是 gdb 可执行文件  进程号，此时也会自动attach

 

 

root@ubuntu:/share# gdb

GNU gdb (Ubuntu 7.10-1ubuntu2) 7.10

Copyright (C) 2015 Free Software Foundation, Inc.

License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.  Type "show copying"

and "show warranty" for details.

This GDB was configured as "i686-linux-gnu".

Type "show configuration" for configuration details.

For bug reporting instructions, please see:

<http://www.gnu.org/software/gdb/bugs/>.

Find the GDB manual and other documentation resources online at:

<http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".

Type "apropos word" to search for commands related to "word".

(gdb)

 

attach 进程号

 

(gdb) attach 15463

Attaching to process 15463

Reading symbols from /share/a.out...done.

Reading symbols from /lib/i386-linux-gnu/libpthread.so.0...Reading symbols from /usr/lib/debug//lib/i386-linux-gnu/libpthread-2.21.so...done.

done.

[New LWP 15467]

[New LWP 15466]

[New LWP 15465]

[New LWP 15464]

[Thread debugging using libthread_db enabled]

Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".

Reading symbols from /lib/i386-linux-gnu/libc.so.6...Reading symbols from /usr/lib/debug//lib/i386-linux-gnu/libc-2.21.so...done.

done.

Reading symbols from /lib/ld-linux.so.2...Reading symbols from /usr/lib/debug//lib/i386-linux-gnu/ld-2.21.so...done.

done.

0xb773abe8 in __kernel_vsyscall ()

 

2.查看线程信息

 

(gdb) info threads

  Id   Target Id         Frame

  5    Thread 0xb7545b40 (LWP 15464) "a.out" 0xb773abe8 in __kernel_vsyscall ()

  4    Thread 0xb6d44b40 (LWP 15465) "a.out" 0xb773abe8 in __kernel_vsyscall ()

  3    Thread 0xb6543b40 (LWP 15466) "a.out" 0xb773abe8 in __kernel_vsyscall ()

  2    Thread 0xb5d42b40 (LWP 15467) "a.out" 0xb773abe8 in __kernel_vsyscall ()

* 1    Thread 0xb7546700 (LWP 15463) "a.out" 0xb773abe8 in __kernel_vsyscall ()

 

 

3.查看所有线程信息并执行bt

 

(gdb) thread apply all bt

 

Thread 5 (Thread 0xb7545b40 (LWP 15464)):

#0  0xb773abe8 in __kernel_vsyscall ()

#1  0xb7711302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb770b5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb77091aa in start_thread (arg=0xb7545b40) at pthread_create.c:333

#5  0xb7631fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 4 (Thread 0xb6d44b40 (LWP 15465)):

#0  0xb773abe8 in __kernel_vsyscall ()

#1  0xb7711302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb770b5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb77091aa in start_thread (arg=0xb6d44b40) at pthread_create.c:333

#5  0xb7631fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 3 (Thread 0xb6543b40 (LWP 15466)):

#0  0xb773abe8 in __kernel_vsyscall ()

#1  0xb7711302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb770b5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb77091aa in start_thread (arg=0xb6543b40) at pthread_create.c:333

#5  0xb7631fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 2 (Thread 0xb5d42b40 (LWP 15467)):

#0  0xb773abe8 in __kernel_vsyscall ()

#1  0xb7711302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb770b5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb77091aa in start_thread (arg=0xb5d42b40) at pthread_create.c:333

#5  0xb7631fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

Thread 1 (Thread 0xb7546700 (LWP 15463)):

#0  0xb773abe8 in __kernel_vsyscall ()

---Type <return> to continue, or q <return> to quit---

#1  0xb75fb3e6 in nanosleep () at ../sysdeps/unix/syscall-template.S:81

#2  0xb75fb1a9 in __sleep (seconds=0) at ../sysdeps/unix/sysv/linux/sleep.c:138

#3  0x08048679 in main () at New0001.c:46

 

 

 

4.选有lock_wait的来查看以下

比如线程 gdb 的id 为4的线程

(gdb) thread 4

[Switching to thread 4 (Thread 0xb6d44b40 (LWP 15465))]

#0  0xb773abe8 in __kernel_vsyscall ()

(gdb) bt

#0  0xb773abe8 in __kernel_vsyscall ()

#1  0xb7711302 in __lll_lock_wait () at ../sysdeps/unix/sysv/linux/i386/i686/../i486/lowlevellock.S:144

#2  0xb770b5fe in __GI___pthread_mutex_lock (mutex=0x804a030 <g_smutex>) at ../nptl/pthread_mutex_lock.c:80

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

#4  0xb77091aa in start_thread (arg=0xb6d44b40) at pthread_create.c:333

#5  0xb7631fde in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:122

 

查看栈上的第三帧

(gdb) frame 3

#3  0x080485b5 in func (arg=0x0) at New0001.c:16

16 pthread_mutex_lock( &g_smutex);

调用锁阻塞了

 

(gdb) p  g_smutex

$1 = {__data = {__lock = 2, __count = 0, __owner = 15468, __kind = 0, __nusers = 1, {__elision_data = {__espins = 0,

        __elision = 0}, __list = {__next = 0x0}}},

  __size = "\002\000\000\000\000\000\000\000l<\000\000\000\000\000\000\001\000\000\000\000\000\000", __align = 2}

```



### core文件中是什么，gdb调试core文件

```
1.core文件
当程序运行过程中出现Segmentation fault (core dumped)错误时，程序停止运行，并产生core文件。core文件是程序运行状态的内存映象。使用gdb调试core文件，可以帮助我们快速定位程序出现段错误的位置。当然，可执行程序编译时应加上-g编译选项，生成调试信息。

当程序访问的内存超出了系统给定的内存空间，就会产生Segmentation fault (core dumped)，因此，段错误产生的情况主要有： 
 （1）访问不存在的内存地址； 
 （2）访问系统保护的内存地址； 
 （3）数组访问越界等。

core dumped又叫核心转储, 当程序运行过程中发生异常, 程序异常退出时, 由操作系统把程序当前的内存状况存储在一个core文件中, 叫core dumped。

2.具体步骤一： 
 （-1）先编译 + -g
 （0）使用ulimit -c unlimited，则表示core文件的大小不受限制。
 （1）启动gdb，进入core文件，命令格式：gdb [exec file] [core file]。 
  用法示例：gdb ./test test.core。
 （2）在进入gdb后，查找段错误位置：where或者bt 
 可以定位到源程序中具体文件的具体位置，出现了段错误。
```



### 如何读取一个10G文件，cat一个10g文件会发生什么

```
在生产环境中有时候可能会遇到大文件的读取问题，但是大文件读取如果按照一般的手法。如cat这种都是对io的一个挑战，如果io扛得住还好，如果扛不住造成的后果，如服务器内存奔溃，日志损坏

方法一：sed
例子:
    按照你自己的日志格式
     sed -n '/14\/Mar\/2015:21/,/14\/Mar\/2015:22/p' access.log >/home/test/test.log
　　　sed -n "1,1000p" access.log >/home/test/test.log     
新生成的test.log就是那个时间段的

方法二：linux split命令

split -l 1000000 access.log -d -a 10 acclog_
　　

方法三：类似python的第三方工具

word='abc'
with open('test.txt','r',encoding='utf-8') as f:   #test.txt为你的源文件
    with open('test2.txt','w',encoding='utf-8') as f2:   #test2.txt为你新生成的包含关键字的文件
        for line in f:
            if word in line:
                f2.write(line)
这里地word就是对关键字的过滤，你可以改成时间段
```

