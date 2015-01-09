#sed, a stream editor
---
##sed, a stream editor

这份文档基于GNU sed 4.2.1版本。

版权&copy;1998,1999,2001,2003,2003,2004 Free Software Foundation, Inc.

##1 介绍

sed是一种流编辑器。流编辑器用于对输入流（文件或者管道输入）进行基本的文本转换。虽然在某些方面和允许进行脚本编辑的编辑器类似（例如ed）,但是sed每次只处理一行，效率更高。和其它类型编辑器不同的是：sed可以过滤管道中的文本。

---

##2 使用

通常sed都这么使用：

	sed SCRIPT INPUTFILE...

完整的使用方式：
	
	sed OPTIONS... [SCRIPT] [INPUTFILE...]

如果不指定INPUTFILE，或者INPUTFILE是-，sed会从标准输入过滤内容。第一个非可选参数会被认为是脚本而非待编辑文件当且仅当其他选项未指定脚本，即没有指定-e或者-f选项。

sed可以指定以下命令行选项：

--version  
打印sed版本和版权信息。

--help  
打印命令行参数的用法以及bug报告地址。

-n  
--quiet  
--silent
sed在每个脚本执行周期结束会默认打印模式空间中的内容（参见sed如何工作）。这些选项可以阻止上述默认打印，并且只有在明确指定p命令的时候才会输出内容。

-e script  
--expression=script  
把scripts中的命令添加到处理输入的命令集中。

-f script-file  
--file=script-file  
把script-file中的命令添加到处理输入的命令集中。

-i[SUFFIX]  
--in-place[=SUFFIX]  

这个选项是指文件会被改动。GNU sed会创建一个临时文件并把输出指定到临时文件而不是标准输出。

该选项和-s含义相同。

当到达文件结尾时，临时文件会被重命名为原始文件的名称。如果提供了扩展名，那么原始文件会在重命名临时文件之前先改名，以做备份。

上述过程遵循以下原则：如果扩展名不包含任何一个\*号，该扩展名会被加到当前文件名称后作为后缀；例如：
如果扩展名包含一个或多个\*号，每个\*号都会被替换为当前文件名。这样你可以为备份文件添加前缀，甚至可以把备份文件放到其他目录（前提是该目录已存在）。

如果没有提供扩展名，那么原始文件会被覆盖并且没有备份。

-l N  
--line-length=N  
指定一条命令自动换行的长度。0表示不进行换行。默认值是70。

--posix  
GNU sed包含一些对POSIX sed的扩展。为了简化脚本，指定该选项会禁用本文档描述的所有扩展及一些额外的命令。大部分扩展可以处理POSIX语法外的sed程序，其中部分还和POSIX标准冲突（例如报告Bug中的命令N）。如果只想禁用冲突的扩展，需要把变量POSIXLY_CORRECT设置为一个非空值。

-b  
-binary  
这个选项任何平台上都适用，但只有在区分文本文件和二进制文件的操作系统上才会生效。在MS-DOS、Windows、Cygwin-text这些以回车换行作为行结束符的平台上，sed找不到结束的回车符。当指定这个选项后，sed会以二进制方式打开文件，并且会认为行以换行符结束。

--follow-symlinks  
这个选项只存在于支持软链接的平台上，并且需要指定-i。如果命令行中的文件是软链接，sed会编辑软链接的目标文件。如果不指定--follow-symlinks，那么链接会中断，目标文件不会被修改。
例如文件example中内容为：
	
	this is an example

通过如下命令创建软链接example_link
	
	ln -s example example_link

	-rw-r--r-- 1 root root 19 1月   9 05:23 example
	lrwxrwxrwx 1 root root  7 1月   9 05:24 example_link -> example

执行如下命令：
	
	sed -i --follow-symlinks 'p' example_link

example文件内容变为：
	
	this is an example
	this is an example

如果不指定--follow-symlinks，执行如下命令：

	sed -i 'p' example_link

example文件内容不变，example_link成为文件

    -rw-r--r-- 1 root root 19 1月   9 05:30 example
    -rw-r--r-- 1 root root 38 1月   9 05:30 example_link

内容为：
	
	this is an example
	this is an example

-r  
--regexp-extended  
使用扩展正则表达式，即egrep所支持的。扩展正则表达式通常使用较少的反斜线，因此更清晰。但由于它是GNU扩展所以不够方便。参见扩展正则表达式。

-s  
--separate  
默认情况下，sed会把命令行里的多个文件当成一个单独的长数据流来处理。使用这个GNU扩展选项，sed会当成单独的文件来处理，需要注意的是：指定的地址范围不能跨文件（例如：‘/abc/,/def/’）；行号是相对于每个文件的首行；$是指每个文件的尾行；R命令指定的文件会倒回到行首。

-u  
--unbuffered  
输入和输出都会尽可能小的缓冲。（比如输入来自“tail -f”，你希望尽可能快的看到结果）。

如果命令行中没指定-e、-f、--expression或者--file，那么第一个非可选参数会被认为是待执行的script。

如果任何参数跟在上述几个选项之后，这个参数会被认为是输入文件。如果文件名为‘-’或者空，sed都会处理标准输入。
