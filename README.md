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

##3 sed Programs

一段sed程序由一条或多条sed命令组成，这些命令可通过选项-e、-f、--expression或者--file指定，如果没指定上述任何选项，则通过第一个非选项参数指定。本文中，上述选项后面跟着的一条或多条命令称为“脚本”。

脚本中的命令以英文半角冒号(:)或者换行符(ASCII 10)分隔。某些命令由于自身语法原因后面不能跟冒号，需要改成换行符或者把该条命令放在整个脚本的最后。Commands can also be preceded with optional non-significant whitespace characters.

每一条sed命令包含一个可选的地址或地址段，后面是一个字符长度的地址名称及其他该命令指定的代码。

###3.1 How sed Works

sed维护两个数据缓存区：pattern space及辅助用的hold space，初始都为空。

对每行输入都按以下周期执行：首先，从输入流中读取一行，去掉行尾换行符，放入pattern space。然后执行命令；每一条命令都有一个关联地址：一种条件代码，只有当条件校验过命令才会被执行。

当执行完最后一条命令时，pattern space中的内容会被打印到输出流，如果之前去过换行符，此时会重新添加上换行符，如果指定了-n选项，则不会执行打印动作。之后是下一行的执行周期。

除非指定了特殊的命令（例如‘D’），pattern space在两个执行周期间会被清空，hold space则会保留数据。(参见命令‘h’，‘H’，‘x’，‘g’，‘G’)。

###3.2 Selecting lines with sed

sed中的地址可以是以下任意一种形式：

number  
指定输入中待匹配的行号。（需要注意的是，如果不指定-i或-s选项，sed会跨文件进行行号计数）。

first~step  
这是一个GNU扩展，匹配从first行开始的每step行。即存在一个非负数n使得行号=first+(n*step)的行被选中。例如：选择奇数行，1~2;选择从第二行开始的每三行，2~3;选择从第10行开始的每五行，10~5；选则第50行，50~0，一种更令人迷惑的写法。

$  
这个地址匹配最后一个输入文件的最后一行，如果指定了-i或者-s选项，则匹配每个文件的最后一行。

/regexp/  
选中匹配正则表达式regexp的所有行。如果regexp包含‘/’，则需要利用反斜线（\）进行转义。

空正则表达式‘//’重复上一个正则表达式匹配（指定s命令时效果相同）。 Note that modifiers to regular expressions are evaluated when the regular expression is compiled, thus it is invalid to specify them together with the empty regular expression. 

\%regexp%  
‘%’可替换为其他任意单个字符。

和上述正则表达式匹配作用相同，只是允许使用除了‘/’外不同的分隔符。在regexp包含大量‘/’时会非常有用， 避免了对每个‘/’繁琐的转义。如果regexp包含自定义的分隔符，也需要通过反斜线（\）转义。

/regexp/I  
\%regexp%I  
I修饰符是一个GNU扩展，使得正则匹配区分大小写。

/regexp/M  
\%regexp%M  
The M modifier to regular-expression matching is a GNU sed extension which causes ^ and $ to match respectively (in addition to the normal behavior) the empty string after a newline, and the empty string before a newline. 特殊的字符序列（\`和\'）通常用来匹配缓冲区的开始和结束。M含义为多行（multi-line）。

如果没指定address，所有行都会被匹配；如果指定了address，只有符合指定address的行会被匹配。

可以通过以逗号（,）分隔的两个地址指定一个地址段。地址段从第一个地址匹配的行开始匹配，到第二个地址匹配的行结束（包含）。

如果第二个地址是正则表达式，那么会从第一个地址匹配到的下一行开始检测，结果至少会包含两行（除非输入流已结束）。

如果第二个地址是一个数字并且小于等于第一个地址匹配的行号，那么只有一行被匹配到。

GNU sed也支持一些特殊的two-address形式（都是GNU扩展）:

0,/regexp/  
第一个地址为0，sed就会从第一行就行正则匹配。换句话说，0,/regexp/和1,/regexp/相似，不同点是：0,/regexp/这种形式正则会匹配输入的第一行；而1,/regexp/这种形式正则会从第二行开始匹配。

记住这是地址唯一可以为0的地方，在其他任何地方使用会报错。

addr1,+N  
匹配addr1行及后面N行。

addr1,~N
匹配addr1行及后续的行直到行号为N的倍数为止。

在地址后添加！符号的含义是取反。即：如果一个地址范围后跟!，那么只有不在这个地址范围内的行会被选中。单行地址、空地址均有效。

---

###3.3 Overview of Regular Expression Syntax

为了更好的sed，对正则表达式应该有所了解。正则表达式是指对目标字符串进行从左到右匹配的一种模式。正则表达式中的大部分字符都是普通的：仅代表它自身，匹配目标字符串中的关联字符。例如模式

	The quick brown fox

仅匹配目标字符串中和它相同的部分。The power of regular expressions comes from the ability to include alternatives and repetitions in the pattern.模式中利用一些特殊字符对这些能力进行编码，这些特殊字符不再代表他们自身而会被以一种特殊的方式进行解释。下面对sed中的正则表达式语法进行简单说明。

char  
一个表示自身的普通字符。

*  
匹配之前的正则表达式实例0次或多次，可以是普通字符、‘\’转义过的特殊字符、‘.’、组合正则表达式（如下所示）或者括号表达式。作为GNU扩展，一个后缀正则表达式后也可以跟‘*’，例如：a**和a*是等价的。当*出现在一个正则表达式或子表达式的开头时，它只表示它自身（见POSIX 1003.1-2001），但许多非GNU实现不支持这个特性，需要进行转义‘\*’。

\+  
GNU扩展，匹配一次或多次。

\?  
GNU扩展，匹配零次或一次。

\{i\}  
匹配i次（i为整数；考虑兼容性，最好取值在0到255之间）。

\{i, j\}  
匹配i到j次。

\{i, \}
匹配i或更多次。

\(regexp\)  
对括号内的正则表达式进行分组，用来：

*	使用后缀操作符，例如“\(abcd\)*”：会匹配零次或多次整个“abcd”序列，而“abcd*”会匹配“abc”后面跟着零个或多个“d”。注意支持“\(abcd\)*”这种形式只是POSIX 1003.1-200的要求，许多非GNU实现并不支持，所以兼容性不是太好。
*	用于后向引用（下面会描述）。

.  
匹配任何字符，包含换行符。

^  
匹配pattern space的开始，例如： what appears after the circumflex must appear at the beginning of the pattern space.

在大部分脚本中，pattern space会被每行的内容初始化（见How sed works）。所以“^”匹配行的起始可以简化思考，“^#include”匹配那些以“#include”开始的行——如果之前包含空格，则匹配失败。这个简化一直有效直到pattern space中的原始内容被修改，例如指定了s命令。

^只有在正则表达式或子表达式的开头（即在\(或\|之后）才会被当做特殊字符。考虑可移植性，尽量避免^出现在子表达式的开头，尽管POSIX在这种情形下允许把^作为普通字符。

$  
和^类似，匹配pattern space的结束。只有在正则表达式或子表达式的结尾（即在\)或\|之前）才会被当做特殊字符，可虑可移植性，尽量避免出现在子表达式的结尾。

[list]  
[^list]  
匹配list中的任意单个字符，例如：[aeiou]匹配所有元音字符。list可以是char1-char2这种序列，匹配char1到char2之间的任意字符。

前置^表示取反，即匹配任意不在list中的字符。如果list中包含]，需要放到第一个字符的位置（^之后）；如果要包含-，需要放到第一个或最后一个字符的位置；包含^，需要放到第二个的位置。

在list中，$*.[\通常都是普通字符。例如：[\*]匹配“\”或者“*”，\在这就不是特殊字符。但是[.ch.]、[=a=]及[:space:]在list中有特殊含义，分别表示collating symbols, equivalence classes, and character classes。因此当list中[后跟着“.”、“=”或“:”时都有特殊含义。当不是POSIXLY_CORRECT模式时，特殊转义如\n和\t都会被识别。参见Escapes。

regexp1\|regexp2  
GNU扩展。匹配regexp1或者regexp2。可以使用括号来写一些复杂的正则表达式选项。匹配进程从左到右尝试每一个选项直到某个选项匹配成功。

regexp1regexp2  
匹配regexp1和regexp2的连接，连接的优先级高于“\|”、“^”和“$”，低于其他正则表达式运算符。

\digit  
匹配第digit个括号括起来的子表达式，这叫做后向引用（back reference）。子表达式按照括号出现的顺序从左到右进行编号。

\n  
匹配换行符。

\char  
匹配char，char可以是“$”，“*”，“.”，“[”，“\”或者“^”。注意：只有C风格的转义序列\n、\\可以被正确解释，在大部分sed实现中，\t会匹配到字符“t”而不是tab。

注意：正则表达式匹配是贪婪的，例如：匹配从左到右尝试，如果两个或更多的匹配都以同一个字符开始，它会选择最长的那个。

例子：

	“abcdef”

匹配“abcdef”

	“a*b”

匹配0个或多个a后跟着一个b。例如：“b”或者“aaaaab”。

	“a\?b”

匹配“b”或者“ab”。

	“a\+b\+”

一个或多个“a”后跟着一个或多个“b”：“ab”是最短的匹配，其他可能是“aaab”或“abbbb”或“aaaaaabbbbbbb”。

	“.*”
	“.\+”

这两个表达式都匹配字符串中的所有字符。但是第一个表达式匹配所有字符（包含空字符），第二个表达式至少匹配一个字符。

	“main.*(.*)”

匹配以“main”开头的字符串后面有一个左括号和一个右括号。“n”，“(”及“)”不需要相邻。

	“^#”

匹配以“#”开头的字符串。

	“\\$”

匹配以反斜线结束的字符串。第一个反斜线是为了转义。

	“\$”

匹配只包含一个美元符号的字符串，因为$被转义了。

	“[a-zA-Z0-9]”

在C语言环境中，匹配任何ASCII字母或者数字。

	“[^ tab]\+”

（这里tab指tab字符。）匹配包含一个或多个字符的字符串，并且所有字符都不是空格或者tab。通常表示一个英文单词。

	“\(.*\)\n\1$”

匹配包含两个以换行符分隔的相同子字符串的字符串。

	“.\{9\}A”

匹配9个字符后跟着一个“A”。

	“^.\{15\}A”

匹配一个包含16个字符的字符串，并且最后一个字符是“A”。

---

###3.4 Often-Used Commands

如果要深度使用sed，下面的命令不容错过。

#  
[No addresses allowed.]

#号后接着注释，直至换行。

如果你关心可移植性，需要注意sed的一些实现(非POSIX标准)或许只支持一个单行注释，并且只有在脚本的第一个字符是#的时候。

警告：如果sed脚本开始的两个字符是#n，那么-n（关闭自动打印）选项默认生效。所以假如上述行为不是你的本意，最好用大写N或者在#和n之间加一个空格。

q [exit-code]  
退出sed。注意如果没有指定-n选项，当前pattern space会被打印。这是一个GNU扩展并且可以返回退出码。

d  
清空pattern space;立即开始下一个周期。

p  
打印pattern space(到标准输出)。该命令通常和-n选项一起使用。

n  
如果没有禁用自动打印，打印pattern space，然后把下一行输入替换进pattern space。如果后续没有输入，则停止执行任何其他命令并退出sed。

{ commands }  
可以用{}包含一组命令。在需要一个地址（或者地址段）触发一组命令时会比较有用。

---

###3.6 Less Frequently-Used Commands

虽然不常用，但某些情况下非常有用。

y/source-chars/dest-chars/  
（‘/’可以用其他任意单个字符替换）。

把pattern space中匹配source_chars的字符转换为dest-chars中对应的字符。例如：

	sed  'y/ts/ab/' sed_example

输出：
	
	ahib ib a aeba1
	ahib ib a aeba2
	ahib ib a aeba3

输入文件中，每一行中的t都被替换成了a，s都被替换成了b。

‘/’（或其他分隔字符）、‘\’或换行可以出现在source-chars或dest-chars中，需要进行转义。source-chars和dest-chars必须包含相同数量的字符（转义后）。

a\  
text  
GNU扩展，接受两个地址。

在当前周期结束后或读取下一行输入前，输出命令后的文本（多行时除最后一行外每行以‘\’结束，‘\’不会输出）。，例如：

	sed -n '1,2a\line 1\
	line 2' sed_example

输出：

	line 1
	line 2
	line 1
	line 2

转义序列在text中生效，如果要打印‘\’，需要写成'\\'。

如果在a和换行符之间除了空字符-\序列以外，还有其他非空字符，那么该行会以a后的第一个非空字符起始，而且是行块的第一行。（简化了一行输入的编辑）这个扩展同样适用于i和c命令。例如：

	sed -n '1,2a        line 1\
	line 2' sed_example

输出：
	
	line 1
	line 2
	line 1
	line 2

输出中，line 1之前的空格都被忽略掉了。

i\  
text  
GNU扩展，接受两个地址。

对指定地址或地址段中的每一行输出一次命令后的文本（多行时除最后一行外每行以‘\’结束，‘\’不会输出）。例如：

	sed -n '1,2i\line 1\
	line 2' sed_example

输出：
	
	line 1
	line 2
	line 1
	line 2

执行后会在地址范围内的每一行之前输出i\后的多行文本。

c\  
text  
对指定地址或地址段中的所有行输出一次命令后的文本（多行时除最后一行外每行以‘\’结束，‘\’不会输出），如果不指定地址，则每行都会执行一次输出。该命令执行完后会开始一个新的周期，因为pattern space已被清空。例如：

指定地址范围：

	sed -n '1,2c\line 1\
	line 2' sed_example

输出：
	
	line 1
	line 2

不指定地址范围：

	sed -n 'c\line 1\
	line2' sed_example

输出：

	line 1
	line 2
	line 1
	line 2
	line 1
	line 2

=  
GNU扩展，接受两个地址。

打印当前输入行行号（后跟一个换行符）。例如执行：

	sed -n '1,2=' sed_example

输出：

	1
	2

l n  
按如下方式打印pattern space：不可打印字符（包含‘\’）以c风格转义的方式打印；长行会被分隔，分隔符是‘\’；行结束会加一个$标记。

n指定上述长行进行分隔的长度。如果n为0，则长行不会分隔；如果省略，则会使用默认值。n是GNU扩展。例如：

不指定n

	sed -n 'l' sed_example

输出
	
	this is a test1$
	this is a test2$
	this is a test3$

n为0
	
	sed -n 'l 0' sed_example

输出
	
	this is a test1$
	this is a test2$
	this is a test3$

n为指定长度
	
	sed -n 'l 5' sed_example

输出

	this\
	 is \
	a te\
	st1$
	this\
	 is \
	a te\
	st2$
	this\
	 is \
	a te\
	st3$

r filename  
GNU扩展，可配置两个地址。

在当前周期结束或者下一个输入行读取时，把filename中的内容插入到输出流中。注意如果filename不可读，会被当成空文件，不会报错。

作为一个GNU扩展，文件名支持标准输入/dev/stdin。

例如：sed_example1是一个新文件，内容为

	line 1
	line 2
	line 3

执行如下命令：

	sed -n 'r sed_example1' sed_example

输出：
	
	line 1
	line 2
	line 3
	line 1
	line 2
	line 3
	line 1
	line 2
	line 3

w filename  
把pattern space中的内容写到filename。作为GNU扩展，支持两个特殊的file-name：/dev/stderr，把结果写到标准错误，/dev/stdout，把结果写到标准输出。

在读取第一行输入前，会创建（或截取）文件。所有的写入命令（包含s命令执行成功后的w标识）都不会关闭或者重新打开该文件。

例如：执行如下命令

	sed -n 'w sed_example2' sed_example

会创建文件sed_example2，文件内容为：

	this is a test1
	this is a test2
	this is a test3

D  
如果pattern space中不包含换行，则开始一个新的周期。否则，删除pattern space中的文本直至第一个换行，并在当前场景下重新开始一个周期，不读取新的输入。

N  
添加一个新行到pattern space，然后再追加下一行输入，如果没有下一行则sed退出执行。

P  
打印pattern space中的行。

h  
把hold space中的内容替换为pattern space中的内容。

H  
追加一行到hold space，然后把pattern space中的内容追加到hold space。

g  
把pattern space中的内容替换为hold space中的内容。

G  
追加一行到pattern space，然后把hold space中的内容追加到pattern space。

x  
交换pattern space和hold space中的内容。

---
