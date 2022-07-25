# Link

## 7.13.3运行时打桩

编译时打桩需要能够访问程序的源代码，链接时打桩需要能够访问程序的可重定位对象文件。不过，有一种机制能够在运行时打桩，它只需要能够访问可执行目标文件。这个很厉害的机制基于动态链接器的LD PRELOAD环境变量。如果LD PRELOAD环境变量被设置为一个共享库路径名的列表（以空格或分号分隔，那么当你加载和执行一个程序，需要解析未定义的引用时，动态链接器(LD-LINUX.S0)会先搜索LD PRELOAD库，然后才搜索任何其他的库。有了这个机制，当你加载和执行任意可执行文件时，可以对任何共享库中的任何函数打桩，包括1ibc.so。

下面是如何构建包含这些包装函数的共享库的方法：

```
linux>gcc -DRUNTIME -shared -fpic -o mymalloc.so mymalloc.c -ldl
```

这是如何编译主程序：

```
linux>gcc -o intr int.c
```

## 处理目标文件的工具

在Liux系统中有大量可用的工具可以帮助你理解和处理目标文件。特别地，GNU binutils包尤其有帮助，而且可以运行在每个Linux平台上。

- AR:创建静态库，插人、删除、列出和提取成员。
- STRINGS:列出一个目标文件中所有可打印的字符串。

```
在对象文件或二进制文件中查找可打印的字符串
补充说明
strings命令 在对象文件或二进制文件中查找可打印的字符串。字符串是4个或更多可打印字符的任意序列，以换行符或空字符结束。 strings命令对识别随机对象文件很有用。
语法
strings [ -a ] [ - ] [ -o ] [ -t Format ] [ -n Number ] [ -Number ]  [file ... ]
选项
-a --all：扫描整个文件而不是只扫描目标文件初始化和装载段
-f –print-file-name：在显示字符串前先显示文件名
-n –bytes=[number]：找到并且输出所有NUL终止符序列
：设置显示的最少的字符数，默认是4个字符
-t --radix={o,d,x} ：输出字符的位置，基于八进制，十进制或者十六进制
-o ：类似--radix=o
-T --target= ：指定二进制文件格式
-e --encoding={s,S,b,l,B,L} ：选择字符大小和排列顺序:s = 7-bit, S = 8-bit, {b,l} = 16-bit, {B,L} = 32-bit
@ ：读取中选项
实例
列出ls中所有的ASCII文本：
strings /bin/ls
列出ls中所有的ASCII文本：
cat /bin/ls strings
查找ls中包含libc的字符串，不区分大小写：
strings /bin/ls | grep -i libc
```

- STRIP:从目标文件中删除符号表信息。

```
NM:列出一个目标文件的符号表中定义的符号。
-a或--debug-syms：显示调试符号。
-B：等同于--format=bsd，用来兼容MIPS的nm。
-C或--demangle：将低级符号名解码(demangle)成用户级名字。这样可以使得C++函数名具有可读性。
-D或--dynamic：显示动态符号。该任选项仅对于动态目标(例如特定类型的共享库)有意义。
-f format：使用format格式输出。format可以选取bsd、sysv或posix，该选项在GNU的nm中有用。默认为bsd。
-g或--extern-only：仅显示外部符号。
-n、-v或--numeric-sort：按符号对应地址的顺序排序，而非按符号名的字符顺序。
-p或--no-sort：按目标文件中遇到的符号顺序显示，不排序。
-P或--portability：使用POSIX.2标准输出格式代替默认的输出格式。等同于使用任选项-f posix。
-s或--print-armap：当列出库中成员的符号时，包含索引。索引的内容包含：哪些模块包含哪些名字的映射。
-r或--reverse-sort：反转排序的顺序(例如，升序变为降序)。
--size-sort：按大小排列符号顺序。该大小是按照一个符号的值与它下一个符号的值进行计算的。
-t radix或--radix=radix：使用radix进制显示符号值。radix只能为“d”表示十进制、“o”表示八进制或“x”表示十六进制。
--target=bfdname：指定一个目标代码的格式，而非使用系统的默认格式。
-u或--undefined-only：仅显示没有定义的符号(那些外部符号)。
-l或--line-numbers：对每个符号，使用调试信息来试图找到文件名和行号。对于已定义的符号，查找符号地址的行号。对于未定义符号，查找指向符号重定位入口的行号。如果可以找到行号信息，显示在符号信息之后。
-V或--version：显示nm的版本号。
--help：显示nm的任选项。
输出符号类型说明（大写表示全局，小写表示局部）
A
该符号的值是绝对的，在以后的链接过程中，不允许进行改变。这样的符号值，常常出现在中断向量表中，例如用符号来表示各个中断向量函数在中断向量表中的位置。
B
该符号的值出现在非初始化数据段(bss)中。例如，在一个文件中定义全局static int test。则该符号test的类型为B，位于bss section中。其值表示该符号在bss段中的偏移。一般而言，bss段分配于RAM中
C
该符号为common。common symbol是未初始化数据段。该符号没有包含于一个普通section中。只有在链接过程中才进行分配。符号的值表示该符号需要的字节数。例如在一个c文件中，定义int test，并且该符号在别的地方会被引用，则该符号类型即为C。否则其类型为B。
补充：该符号所占的空间并不存在于执行文件中，而在初始化执行环境时分配此空间，但不会清零，可读写。
D
该符号位于初始话数据段中。一般来说，分配到data section中。例如定义全局int baud_table[5] = {9600, 19200, 38400, 57600, 115200}，则会分配于初始化数据段中。
补充：该符号所占用的空间存在于执行文件中，在初始化执行环境时分配，并复制数据到此空间，可读写。
G
该符号也位于初始化数据段中。主要用于small object提高访问small data object的一种方式。
I
该符号是对另一个符号的间接引用。
N
该符号是一个debugging符号。
R
该符号位于只读数据区。例如定义全局const int test[] = {123, 123};则test就是一个只读数据区的符号。注意在cygwin下如果使用gcc直接编译成MZ格式时，源文件中的test对应_test，并且其符号类型为D，即初始化数据段中。但是如果使用m6812-elf-gcc这样的交叉编译工具，源文件中的test对应目标文件的test,即没有添加下划线，并且其符号类型为R。一般而言，位于rodata section。值得注意的是，如果在一个函数中定义const char *test = “abc”, const char test_int = 3。使用nm都不会得到符号信息，但是字符串“abc”分配于只读存储器中，test在rodata section中，大小为4。
补充：此符号所占用的空间存在于执行文件中，是否使用副本空间并不确定。只读。
S
符号位于非初始化数据区，用于small object。
T
该符号位于代码区text section。
U
该符号在当前文件中是未定义的，即该符号的定义在别的文件中。例如，当前文件调用另一个文件中定义的函数，在这个被调用的函数在当前就是未定义的；但是在定义它的文件中类型是T。但是对于全局变量来说，在定义它的文件中，其符号类型为C，在使用它的文件中，其类型为U。
V
该符号是一个weak object。
W
The symbol is a weak symbol that has not been specifically tagged as a weak object symbol.
-
该符号是a.out格式文件中的stabs symbol。
?
该符号类型没有定义
```

- SIZE:列出目标文件中节的名字和大小。

1. size 用于查看目标文件、库或可执行文件中各段及其总和的大小，是 GNU 二进制工具集 GNU Binutils 的一员。
2. 命令格式
   ```
   size [-A|-B|--format=compatibility]
       [--help]
       [-d|-o|-x|--radix=number]
       [--common]
       [-t|--totals]
       [--target=bfdname] [-V|--version]
       [OBJFILE...]
   ```

3. 选项说明

```
-A
-B
--format=compatibility
 控制输出格式。-A 或 --format=sysv 表示使用 System V size 风格，-B 或 --format=berkeley 表示使用 Berkeley size 风格。默认使用 Berkeley size 风格的输出。
 
 下面是 Berkeley 风格示例：
 $ size --format=Berkeley ranlib size
 text    data    bss     dec     hex     filename
 294880  81920   11592   388392  5ed28   ranlib
 294880  81920   11888   388688  5ee50   size
 
 下面是接近 System V 风格示例：
 $ size --format=SysV ranlib size
 ranlib  :
 section         size         addr
 .text         294880         8192
 .data          81920       303104
 .bss           11592       385024
 Total         388392
 
 size  :
 section         size         addr
 .text         294880         8192
 .data          81920       303104
 .bss           11888       385024
 Total         388688
 
--help
 显示帮助信息
 
-d
-o
-x
--radix=number
 控制大小输出的进制 -d 或 --radix=10 表示 10 进制，-o 或 --radix=8 表示八进制，-x 或 --radix=16 表示 16 进制
 
--common
 打印每个文件的 common symbols 大小
 
-t
--totals
 列出所有文件的总大小。注意，只能使用 Berkeley 风格输出
 
--target=bfdname
 指明目标文件的格式。该选项没有必要指定，因为 size 可自动推导
 
-V
--version
 显示版本
 
@file
 从指定的文件 file 读取命令行选项。文件中的选项由空白符（空格，TAB和回车）分隔。选项中可以包含空白字符，方法是将整个选项用单引号或双引号括起来。任何字符（包括反斜杠）可以通过添加前缀反斜杠来包含。文件本身可能包含额外的 @file 选项，该选项将以递归方式处理
```

4. 常用示例

（1）查看指定程序各个段的大小。以 size 为例。

```
[root@192 ~]# size /bin/size
   text	   data	    bss	    dec	    hex	filename
  23673	   1436	   1368	  26477	   676d	/bin/size
[root@192 ~]# 
```

（2）查看静态库中的各个目标文件的段大小。以 libc.a 为例。

```
[root@192 ~]# size /usr/lib64/libc.a
text    data     bss     dec     hex filename
233       4       0     237      ed init-first.o (ex /usr/lib64/libc.a)
1667       0       0    1667     683 libc-start.o (ex /usr/lib64/libc.a)
64       0       0      64      40 sysdep.o (ex /usr/lib64/libc.a)
953       0       0     953     3b9 version.o (ex /usr/lib64/libc.a)
395       0       0     395     18b check_fds.o (ex /usr/lib64/libc.a)
852       8    2192    3052     bec libc-tls.o (ex /usr/lib64/libc.a)
307       0       0     307     133 elf-init.o (ex /usr/lib64/libc.a)
8       0       0       8       8 dso_handle.o (ex /usr/lib64/libc.a)
0       0       4       4       4 errno.o (ex /usr/lib64/libc.a)
...
```

- READELF:显示一个目标文件的完整结构，包括ELF头中编码的所有信息。包含SIZE和NM的功能。
  
  [https://blog.csdn.net/mayue_web/article/details/115939103](https://)
- OBJDUMP:所有二进制工具之母。能够显示一个目标文件中所有的信息。它最大的作用是反汇编.text节中的二进制指令。
  
  [https://blog.csdn.net/tjcwt2011/article/details/117963146](https://)
  
  [https://linux265.com/course/linux-command-objdump.html](https://)
- Linux系统为操作共享库还提供了LDD程序 LDD:列出一个可执行文件在运行时所需要的共享库。
  
  [https://blog.csdn.net/feikudai8460/article/details/121834593](https://)