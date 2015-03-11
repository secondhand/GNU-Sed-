#sed, a stream editor
---
##sed, a stream editor

本文档基于GNU sed 4.2.1版本。

Copyright © 1998, 1999, 2001, 2002, 2003, 2004 Free Software Foundation, Inc.

本文档遵循自由软件基金会发布的GNU Free Documentation License，1.1版本或（你所知道的）任何最新版本。

你应该随着GNU sed收到一份GNU Free Documentation License的拷贝，即文件COPYING.DOC。如果没有，请写信到自由软件基金会，地址：59 Temple Place - Suite 330, Boston, MA 02110-1301, USA。

There are no Cover Texts and no Invariant Sections; this text, along with its equivalent in the printed manual, constitutes the Title Page.

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

\*  
匹配之前的正则表达式实例0次或多次，可以是普通字符、‘\’转义过的特殊字符、‘.’、组合正则表达式（如下所示）或者括号表达式。作为GNU扩展，一个后缀正则表达式后也可以跟‘\*’，例如：a\*\*和a\*是等价的。当\*出现在一个正则表达式或子表达式的开头时，它只表示它自身（见POSIX 1003.1-2001），但许多非GNU实现不支持这个特性，需要进行转义‘\*’。

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

在list中，$\*.[\通常都是普通字符。例如：[\*]匹配“\”或者“*”，\在这就不是特殊字符。但是[.ch.]、[=a=]及[:space:]在list中有特殊含义，分别表示collating symbols, equivalence classes, and character classes。因此当list中[后跟着“.”、“=”或“:”时都有特殊含义。当不是POSIXLY_CORRECT模式时，特殊转义如\n和\t都会被识别。参见Escapes。

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

###3.5 The s Command

s命令的语法是“s/regexp/replacement/flags”。/字符可以统一替换为其他任意单个字符，如果/（或其他替换字符）出现在regexp或者replacement中，需要转义。

s命令或许是sed中最重要的命令，包含许多选项。基本概念比较简单：尝试用regexp匹配pattern space中的内容；如果匹配成功，那么pattern space中匹配成功的部分会被replacement替换。

replacement可以包含\n（n是1-9中的一个数字）引用，which refer to the portion of the match which is contained between the nth \( and its matching \).replacement也可以包含非转义的&字符，代表pattern space中整个被匹配的部分。最后，作为GNU扩展，replacement可以包含由反斜线和L、l、U、u或E中的一个字符组成的特殊序列。含义如下：

\L  
把replacement转换为小写直至出现\U或\E，

\l
把下一个字符转换为小写，

\U  
把replacement转换为大写直至出现\L或\E，

\u  
把下一个字符转换为大写，

\E  
停止\L或\U引起的大小写转换。

如果最终的替换中要包含字符\、&或换行符，需要在这些字符前面加上\。

s命令可以包含以下flags：

g  
对所有匹配进行替换，不仅仅是第一个。

number  
只替换第number个匹配。

注意：POSIX标准并没有明确当同时使用g和number修饰符时的结果，当前sed的各种实现对此也没有达成广泛的一致。GNU sed对此的解释是：忽略number之前的所有匹配，然后匹配替换number之后的所有匹配项。

p  
如果进行了替换，打印新的pattern space。

注意：如果同时指定p和e选项，p和e的先后顺序会对结果产生影响。通常ep（评估然后打印）是你想要的，但是另外一种顺序在debug时非常有用。基于此，当前版本的GNU sed对于p在e之前和之后的不同表现（在评估之前或之后打印pattern space）进行了解释，而通常flags只会产生一次效果。这个行为虽然文档有描述，但未来的版本或许会有改动。

w file-name  
如果进行了替换，把结果写到指定文件中。作为GNU扩展，支持两个特殊的文件：/dev/stderr，把结果写到标准错误；/dev/stdout，把结果写到标准输出。

e  
GNU扩展，允许把shell命令通过管道指定到pattern space中。如果进行了替换，pattern space中的命令会被执行并且输出结果会覆盖到pattern space。A trailing newline is suppressed;如果待执行命令包含NUL字符，结果会是不确定的。

I  
i  
GNU扩展，以区分大小写方式进行正则匹配。

M  
m  
GNU扩展，在这种模式下，^和$分别匹配行的开始和行的结束。有其他特殊的字符序列（\`和\'）匹配buffer的起始和结束。M表示多行。

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

###3.8 Commands Specific to GNU sed

以下命令是GNU sed特有的，所以要小心使用或者在不考虑可移植性的情况下使用。这些命令允许你检测GNU sed扩展或者做一些需要经常做但标准sed不支持的任务。

e [command]  
该命令允许你把shell命令通过管道导进pattern space并执行。没有参数时，会把pattern space中的内容当做shell命令执行，并把shell命令执行的输出覆盖回pattern sapce。不支持换行符。例如，把文件sed_example中的内容改为如下：

	echo 1
	echo 2
	echo 3

执行：

sed "e" sed_example

输出：

	1
	2
	3

上述命令没指定-n选项，所以pattern space中的内容被打印出来，从输出可以看出sed_example中的每一行被当成命令执行了，并且结果重写到了pattern space中。

如果指定了一个参数，e命令会把参数当做命令执行并且把输出发送到输出流（和r类似）。该命令支持跨行，除最后一行外每行需要以反斜线结尾。例如，执行：

sed -n "e echo 1" sed_example

输出：

	1
	1
	1

上述两种情况下，如果待执行命令中包含NUL字符，那么结果是不可预期的。

F  
打印输入文件的文件名（另起一行）。

L n  
GNU扩展，填充以及连接pattern space中的行以产生包含n（最多）个字符的多行输出，和fmt效果类似。如果忽略n，会使用默认值。这个命令是一个失败的尝试，未来版本会被移除。

Q [exit-code]  
只接受一个地址。

和命令q类似，可以返回退出码，但不会打印pattern space中的内容。

这个命令有一些作用，因为除了它实现“不打印pattern space”这个小特性就只能使用-n选项（这会增加脚本的复杂度）或者借助于下面的片段，读取了整个文件但并没有其他显而易见的效果：

	:eat
	$d    Quit silently on the last line
	N     Read another line, silently
	g     Overwrite pattern space each time to save memory
	b eat

R filename  
顺序读取filename中的行并在每个周期结束或者读取下一行输入时插入到输出流。注意：如果filename不可读或者已达到filename文件结尾，不会往输出流中插入新行，并且没有任何错误提示。例如，执行：

	sed "R sed_example" sed_example1

输出：

	sed_example1 line 1
	sed_example line1
	sed_example1 line 2
	sed_example line2
	sed_example1 line 3
	sed_example line3
	sed_example1 line 4

上述命令没有指定-n选项，所以每个周期都会打印pattern space中的内容，在这之后会打印sed_example中的一行，在sed_exmple顺序输出完成后，sed继续执行新的周期，没有输出错误。

和r命令类似，filename也可以是/dev/stdin，从标准输入读取行。例如：

执行：

	sed "R /dev/stdin" sed_example1

输出：

	stdin line 1
	sed_example1 line 1
	stdin line 1
	stdin line 2
	sed_example1 line 2
	stdin line 2
	stdin line 3 
	sed_example1 line 3
	stdin line 3
	stdin line 4
	sed_example1 line 4
	stdin line 4

执行后会等待标准输入，输入一行后会先打印pattern space中的内容，然后打印标准输入，接着等待下一次标准输入，循环直至sed所有周期执行结束。

T label  
如果在读取上一行输入后没有发生替换或者没有执行条件分支跳转，则跳转到label。label可以忽略，在这种情况下会开始执行下一个周期。

v version  
该命令只是用来检测是否支持GNU sed扩展，如果不支持GNU sed扩展，命令执行失败。进一步，你可以指定你所需要的sed版本，例如4.0.5。默认是4.0，因为从这个版本开始支持这个命令。

该命令会启用所有GNU扩展，即使环境里设置了POSIXLY_CORRECT。

例如在GNU sed 4.2.1版本下，执行：

	sed -n "v" sed_example

没有任何输出，执行：

	echo $?

输出：
	
	0

退出码为0，表示命令执行成功。

执行：

	sed -n "v 5.0" sed_example

输出：
	
	sed：-e 表达式 #1，字符 5：需要更高版本的sed

执行：

	echo $?

输出：
	
	1

退出码为1，表示命令执行失败。

W filename  
输出pattern space中的内容到filename直至第一个换行。其他和w命令类似。

z  
该命令会清空pattern space中的内容。通常和“s/.*//”效果相同，但效率更高并且如果输入流中有异常的多字节序列时依然有效，并且POSIX规定上述多字节序列不会被“.”匹配，所以在大部分多字节语言环境下（包含UTF-8），在脚本中间利用正则清空sed缓冲区的兼容性并不好。

---

###3.9 GNU Extensions for Escapes in Regular Expressions

直到本章，我们遇到的转义都是“\^”形式的，sed不会把进行转义的字符当做特殊字符。例如：“\*”匹配单个星号，而不是零个或多个反斜线。

本章将介绍另外一种转义方式——that is, escapes that are applied to a character or sequence of characters that ordinarily are taken literally, and that sed replaces with a special character. 这为不可打印字符编码提供了一种可见的方式。sed对于脚本中使用不可打印字符没有限制，但当用shell或脚本编辑sed脚本时，使用下述的转义序列比使用二进制字符更容易。

转义序列列表如下：

\a  
产生或匹配一个BEL字符，即报警字符（ASCII 7）。

\f  
产生或匹配一个换页符（ASCII 12）。

\n  
产生或匹配一个换行符（ASCII 10）。

\r  
产生或匹配一个回车（ASCII 13）。

\t  
产生或匹配一个制表符tab（ASCII 9）。

\v  
产生或匹配一个竖向制表符（vertical tab，ASCII 11）。

\cx  
产生或匹配CONTROL-x，x是任意字符。“\cx”的实际效果如下：如果x是一个小写字母，会被转换为大写。然后该字符的第6bit位（二进制40）会取反。因此“\cz”会变为二进制的1A，“\c{”变为二进制3B，“\c:”变为二进制7B。

\dxxx  
产生或匹配一个十进制ASCII值为xxx的字符。

\oxxx  
产生或匹配一个八进制ASCII值为xxx的字符。

\xxx  
产生或匹配一个十六进制ASCII值为xx的字符。

“\b”（退格）会被忽略因为会和匹配“单词边界”的转义序列冲突。

下述转义序列匹配特定的字符类并只适用于正则表达式：

\w  
匹配任意单词（word）字符，包含字母数字下划线。

\W  
匹配任意非单词字符。

\b  
匹配单词边界；也就是匹配左边是一个单词字符并且右边是一个非单词字符的位置，反之亦然。

\B  
匹配不是单词边界的位置；也就是匹配左边和右边同时为单词字符或者非单词字符的位置。

\`  
匹配pattern space的开始。在多行模式下和^不同。

\’  
匹配pattern space的结束。在多行模式下和$不同。

---

##4 Some Sample Scripts

下面是一些sed脚本的例子帮助你掌握它。

一些有趣的例子：

*	行居中
*	数字自增
*	文件名重命名为小写
*	打印bash环境
*	逆置行字符串

模拟标准工具：

*	tac：	 逆置行字符串
*	cat -n： 行编号
*	cat -b： 非空行编号
*	wc -c：  字符计数
*	wc -w：  单词计数
*	wc -l：  行计数
*	head：   打印第一行
*	tail：   打印最后一行
*	uniq：   行去重
*	uniq -d：打印输入中重复的行
*	uniq -u：删除重复行
*	cat -s： 合并空行

---

Next: Increment a number, Up: Examples

###4.1 Centering Lines

把一个文件中的所有行居中，并且行宽为80个字符宽度。如果要改变行宽，需要替换“\{...\}”中的数字，添加的空格数也需要改变。

注意：如何利用缓冲区命令分离待匹配正则表达式的部分是一个常用技术。

    #!/usr/bin/sed -f
     
    # 把80个空格放入缓冲区
    1 {
        x
        s/^$/          /
        s/^.*$/&&&&&&&&/
        x
    }
     
    # 删除头尾空格
    y/\t/ / #原文档中此处为tab，替换tab字符，实际应该用\t
    s/^ *//
    s/ *$//

    # 行尾加上换行及80个空格
    G

    # 保持前81个字符(80个字符 + 1个换行)
    s/^\(.\{81\}\).*$/\1/

    # \2 匹配半数空格, 这些空格被移到开头
    s/^\(.*\)\n\(.*\)\2/\2\1/

---

Next: Rename files to lower case, Previous: Centering lines, Up: Examples

###4.2 Increment a Number

这个脚本是几个演示如何用sed做算术运算的脚本之一，确实是可行的，但必须手工完成。

实现一个数的自增只需要把最后一位数字加1，即把它替换为下一个数字。有一个特例：如果这个数字是9，它前面的数字必须也加1直到没有9。

由Bruno Haible提出的解决方法非常聪明、智能，因为只使用了一个缓冲区，如果没有这个限制，Numbering lines中的算法会更快。工作原理：用下划线替换末尾的所有9，然后利用多个s命令让最后一位数字加1，然后把所有下划线替换为0。

	#!/usr/bin/sed -f
     
	/[^0-9]/ d

	# replace all leading 9s by _ (any other character except digits, could
	# be used)
	:d
	s/9\(_*\)$/_\1/
	td

	# incr last digit only.  The first line adds a most-significant
	# digit of 1 if we have to add a digit.
	#
	# The tn commands are not necessary, but make the thing
	# faster

	s/^\(_*\)$/1\1/; tn
	s/8\(_*\)$/9\1/; tn
	s/7\(_*\)$/8\1/; tn
	s/6\(_*\)$/7\1/; tn
	s/5\(_*\)$/6\1/; tn
	s/4\(_*\)$/5\1/; tn
	s/3\(_*\)$/4\1/; tn
	s/2\(_*\)$/3\1/; tn
	s/1\(_*\)$/2\1/; tn
	s/0\(_*\)$/1\1/; tn

	:n
	y/_/0/

---

Next: Print bash environment, Previous: Increment a number, Up: Examples

###4.3 Rename Files to Lower Case

这是一个非常奇怪的使用方式。文本转换，并把它转换成shell命令，然后以shell方式执行。别担心，还有更糟糕的hack用法，我曾经看过一个脚本把date命令的输出转换为了bc 程序。

这段程序的主体是sed脚本，把名称从小写映射到大写（反之亦然）并且会检查映射后的名称和之前是否一致。注意：该脚本时如何利用shell脚本和适当的引号进行参数化的。

	#! /bin/sh
	# rename files to lower/upper case...
	#
	# usage:
	#    move-to-lower *
	#    move-to-upper *
	# or
	#    move-to-lower -R .
	#    move-to-upper -R .
	#

	help()
	{
	     cat << eof
	Usage: $0 [-n] [-r] [-h] files...

	-n      do nothing, only see what would be done
	-R      recursive (use find)
	-h      this message
	files   files to remap to lower case

	Examples:
	    $0 -n *        (see if everything is ok, then...)
	    $0 *

	    $0 -R .

	eof
	}

	apply_cmd='sh'
	finder='echo "$@" | tr " " "\n"'
	files_only=

	while :
	do
	 case "$1" in
	     -n) apply_cmd='cat' ;;
	     -R) finder='find "$@" -type f';;
	     -h) help ; exit 1 ;;
	     *) break ;;
	 esac
	 shift
	done

	if [ -z "$1" ]; then
	     echo Usage: $0 [-h] [-n] [-r] files...
	     exit 1
	fi

	LOWER='abcdefghijklmnopqrstuvwxyz'
	UPPER='ABCDEFGHIJKLMNOPQRSTUVWXYZ'

	case `basename $0` in
	     *upper*) TO=$UPPER; FROM=$LOWER ;;
	     *)       FROM=$UPPER; TO=$LOWER ;;
	esac

	eval $finder | sed -n '

	# remove all trailing slashes
	s/\/*$//

	# add ./ if there is no path, only a filename
	/\//! s/^/.\//

	# save path+filename
	h

	# remove path
	s/.*\///

	# do conversion only on filename
	y/'$FROM'/'$TO'/

	# now line contains original path+file, while
	# hold space contains the new filename
	x

	# add converted file name to line, which now contains
	# path/file-name\nconverted-file-name
	G

	# check if converted file name is equal to original file name,
	# if it is, do not print nothing
	/^.*\/\(.*\)\n\1/b

	# now, transform path/fromfile\n, into
	# mv path/fromfile path/tofile and print it
	s/^\(.*\/\)\(.*\)\n\(.*\)$/mv "\1\2" "\1\3"/p

	' | $apply_cmd
	
---

###4.4 Print bash Environment

这个脚本会从Bourne-shell命令set的输出中去掉shell函数的定义。

	#!/bin/sh

	set | sed -n '
	:x

	# if no occurrence of ‘=()’ print and load next line
	/=()/! { p; b; }
	/ () $/! { p; b; }

	# possible start of functions section
	# save the line in case this is a var like FOO="() "
	h

	# if the next line has a brace, we quit because
	# nothing comes after functions
	n
	/^{/ q

	# print the old line
	x; p

	# work on the new line now
	x; bx
	'
---

Next: tac, Previous: Print bash environment, Up: Examples

###4.5 Reverse Characters of Lines

该脚本可以进行行倒置。由于可以同时移动两个字符，所以该脚本的效率比其他简单的实现高一些。

注意label定义前的tx命令，通常需要重置t命令检测过的标识。

想象力丰富的读者可以想到该脚本的用法，比如：倒置banner的输出。

	#!/usr/bin/sed -f

    /../! b

    # Reverse a line.  Begin embedding the line between two newlines
    s/^.*$/\
    &\
    /

    # Move first character at the end.  The regexp matches until
    # there are zero or one characters between the markers
    tx
    :x
    s/\(\n.\)\(.*\)\(.\n\)/\3\2\1/
    tx

    # Remove the newline markers
    s/\n//g

---
Next: cat -n, Previous: Reverse chars of lines, Up: Examples

###4.6 Reverse Lines of Files

从这个脚本开始后续都是一些没有实际用处（但有意思）的脚本模拟各种Unix命令。这个脚本模拟的是tac命令。

需要注意的是在其他非GNU sed实现中执行该脚本，内部缓存比较容易溢出。

	#!/usr/bin/sed -nf

	# reverse all lines of input, i.e. first line became last, ...

	# from the second line, the buffer (which contains all previous lines)
	# is *appended* to current line, so, the order will be reversed
	1! G

	# on the last line we're done -- print everything
	$ p

	# store everything on the buffer again
	h

---

Next: cat -b, Previous: tac, Up: Examples

###4.7 Numbering Lines

该脚本可以替换“cat -n”，事实上它的输出格式和GNU cat相同。

但实际上该脚本基本没用，因为已经有C的实现了并且下述的Bourne-shell脚本效果相同但速度更快：
	
	#! /bin/sh
	sed -e "=" $@ | sed -e '
		s/^/      /
		N
		s/^ *\(......\)\n/\1  /
	'

上述脚本利用sed打印行号，然后使用N命令对行两两分组。但该脚本可学习的东西比下面介绍的少。

该自增算法使用了两个缓存区，所以行号打印完以后不需要完整保存。行号被分隔成两部分，需要改变的数字进入pattern space，另一部分进入hold space；pattern space中的数字通过一步y命令替换实现自增，然后和hold space中的数字合并形成下一行的行号，写回hold space，以便下一次迭代时使用。

	#!/usr/bin/sed -nf

    # Prime the pump on the first line
    x
    /^$/ s/^.*$/1/

    # Add the correct line number before the pattern
    G
    h

    # Format it and print it
    s/^/      /
    s/^ *\(......\)\n/\1  /p

    # Get the line number from hold space; add a zero
    # if we're going to add a digit on the next line
    g
    s/\n.*$//
    /^9*$/ s/^/0/

    # separate changing/unchanged digits with an x
    s/.9*$/x&/

    # keep changing digits in hold space
    h
    s/^.*x//
    y/0123456789/1234567890/
    x

    # keep unchanged digits in pattern space
    s/x.*$//

    # compose the new number, remove the newline implicitly added by G
    G
    s/\n//
    h

---

Next: wc -c, Previous: cat -n, Up: Examples

###4.8 Numbering Non-blank Lines

模拟“cat -b”和“cat -n”类似，只需要选出需要进行编号的行。

为了说明在sed脚本中适当注释的重要性，这一节中和上节中重复的部分没有注释。

	#!/usr/bin/sed -nf

	/^$/ {
		p
		b
	}

	# Same as cat -n from now
	x
	/^$/ s/^.*$/1/
	G
	h
	s/^/      /
	s/^ *\(......\)\n/\1  /p
	x
	s/\n.*$//
	/^9*$/ s/^/0/
	s/.9*$/x&/
	h
	s/^.*x//
	y/0123456789/1234567890/
	x
	s/x.*$//
	G
	s/\n//
	h

---

Next: wc -w, Previous: cat -b, Up: Examples

###4.9 Counting Characters

下述脚本展示了另一种利用sed做算术运算的方法，在这种情况下可能需要做大数加法，如果还通过连续的自增实现的话就不够灵活了（可能会更复杂）。

该方法是把数字映射成字母，就像利用sed实现的算盘。“a”是单位，“b”是单位的十倍，以此类推。我们只需要把当前行的字母数换算成单位数，然后进位成十、百等。

和之前一样，过程中的临时总数保存在hold space中。

最后把算盘形式转换为数字。For the sake of variety, this is done with a loop rather than with some 80 s commands：第一步转换单位，把“a”替换成数字；第二步进行轮转替换，“b”变成“a”，跳转到第一步，重复该流程直至没有字母。

	#!/usr/bin/sed -nf

	# Add n+1 a's to hold space (+1 is for the newline)
	s/./a/g
	H
	x
	s/\n/a/

	# Do the carry.  The t's and b's are not necessary,
	# but they do speed up the thing
	t a
	: a;  s/aaaaaaaaaa/b/g; t b; b done
	: b;  s/bbbbbbbbbb/c/g; t c; b done
	: c;  s/cccccccccc/d/g; t d; b done
	: d;  s/dddddddddd/e/g; t e; b done
	: e;  s/eeeeeeeeee/f/g; t f; b done
	: f;  s/ffffffffff/g/g; t g; b done
	: g;  s/gggggggggg/h/g; t h; b done
	: h;  s/hhhhhhhhhh//g

	: done
	$! {
	h
	b
	}

	# On the last line, convert back to decimal

	: loop
	/a/! s/[b-h]*/&0/
	s/aaaaaaaaa/9/
	s/aaaaaaaa/8/
	s/aaaaaaa/7/
	s/aaaaaa/6/
	s/aaaaa/5/
	s/aaaa/4/
	s/aaa/3/
	s/aa/2/
	s/a/1/

	: next
	y/bcdefgh/abcdefg/
	/[a-h]/ b loop
	p

---

Next: wc -l, Previous: wc -c, Up: Examples

###4.10 Counting Words

在每个单词都被替换为字母“a”以后（上一节中每一个字母都被替换成“a”），本节脚本的剩余部分和上一节相同。

有趣的是真实的wc程序为“wc -c”做过优化，所以单词计数比字母计数慢；但这个脚本的瓶颈是数学运算，因此单词计数要更快一些（它处理的数更小）。

为了说明注释的重要性，这一节中和上节中重复的部分也没有注释。

	#!/usr/bin/sed -nf
	 
	# Convert words to a's
	s/[ tab][ tab]*/ /g
	s/^/ /
	s/ [^ ][^ ]*/a /g
	s/ //g

	# Append them to hold space
	H
	x
	s/\n//

	# From here on it is the same as in wc -c.
	/aaaaaaaaaa/! bx;   s/aaaaaaaaaa/b/g
	/bbbbbbbbbb/! bx;   s/bbbbbbbbbb/c/g
	/cccccccccc/! bx;   s/cccccccccc/d/g
	/dddddddddd/! bx;   s/dddddddddd/e/g
	/eeeeeeeeee/! bx;   s/eeeeeeeeee/f/g
	/ffffffffff/! bx;   s/ffffffffff/g/g
	/gggggggggg/! bx;   s/gggggggggg/h/g
	s/hhhhhhhhhh//g
	:x
	$! { h; b; }
	:y
	/a/! s/[b-h]*/&0/
	s/aaaaaaaaa/9/
	s/aaaaaaaa/8/
	s/aaaaaaa/7/
	s/aaaaaa/6/
	s/aaaaa/5/
	s/aaaa/4/
	s/aaa/3/
	s/aa/2/
	s/a/1/
	y/bcdefgh/abcdefg/
	/[a-h]/ by
	p

---

Next: head, Previous: wc -w, Up: Examples

###4.11 Counting Lines

这次没有令人费解的实现了，因为sed内部提供了“wc -l”功能！
	#!/usr/bin/sed -nf
	$=

---

Next: tail, Previous: wc -l, Up: Examples

###4.12 Printing the First Lines

这或许是最没有用的sed脚本：显示输入的前10行。行数在q命令之前。

	#!/usr/bin/sed -f
	10q

---

Next: uniq, Previous: head, Up: Examples

###4.13 Printing the Last Lines

打印最后n行比打印最后一行更复杂，但确实是可实现的。n is encoded in the second line, before the bang character.

这个脚本和tac脚本类似，把最终的输出缓存到hold space并在最后打印：

	#!/usr/bin/sed -nf
     
	1! {; H; g; }
	1,10 !s/[^\n]*\n//
	$p
	h

维持一个10行的的窗口，通过添加一行及删除最旧的行来滑动它（第二行的替换命令和D命令类似但不重启循环）。

在写高效且复杂的sed脚本时，“滑动窗口”是一种非常强大的技术，如果要手工实现P命令会需要大量的工作。

为了介绍这种在本章剩余部分被充分证明的技术（基于N，P，D命令），下面是一个利用简单“滑动窗口”实现的tail。

看起来复杂但实际上和上一个脚本效果相同：在输入适当的行数后，不再需要用hold space缓存状态，而是用N和D来实现pattern space滑动:

	#!/usr/bin/sed -f
	     
	1h
	2,10 {; H; g; }
	$q
	1,9d
	N
	D

注意输入在超过十行后的第一、二及四行是怎么失效的，在那之后，脚本按如下执行：在最后一行输入时退出，把下一行追加到pattern space，删除第一行。

---

Next:uniq -d, Previous: tail, Up: Examples

###4.14 Make Duplicate Lines Unique

本例展示了使用N、P、D命令的艺术，比较难以掌握。

	#!/usr/bin/sed -f
	h

	:b
	# On the last line, print and exit
	$b
	N
	/^\(.*\)\n\1$/ {
		# The two lines are identical. Undo the effect of
		# the n command.
		g
		bb
	}

	# If the N command had added the last line, print and exit
	$b

	# The lines are different; print the first and go
	# back working on the second.
	P
	D
如你所见，通过P和D命令维护了一个两行的窗口，这个技术在高级sed脚本中经常使用。

---

Next: cat -s, Previous: uniq -d, Up: Examples

###4.15 Print Duplicated Lines of Input

该脚本只打印重复行，和“uniq -d”类似。

	#!/usr/bin/sed -nf

	$b
	N
	/^\(.*\)\n\1$/ {
		# 打印两个重复行中第一行
		s/.*\n//
		p

		# 循环直至读取到一个非重复行
		:b
		$b
		N
		/^\(.*\)\n\1$/ {
			s/.*\n//
			bb
		}
	}

	# 最后一行后面不可能有重复行
	$b

	# 找到一个非重复行，单独留在pattern space中
	# 开始下一个循环, 寻找和它重复的行
	D

---

Next: cat -s, Previous: uniq -d, Up: Examples

###4.16 Remove All Duplicated Lines

这个脚本只打印不重复的行，和“unique -u”类似。

	#!/usr/bin/sed -f

	# Search for a duplicate line --- until that, print what you find.
	$b
	N
	/^\(.*\)\n\1$/ ! {
		P
		D
	}

	:c
	# Got two equal lines in pattern space.  At the
	# end of the file we simply exit
	$d

	# Else, we keep reading lines with N until we
	# find a different one
	s/.*\n//
	N
	/^\(.*\)\n\1$/ {
		bc
	}

	# Remove the last instance of the duplicate line
	# and go back to the top
	D

---

Previous: uniq -u, Up: Examples

###4.17 Squeezing Blank Lines

作为最后一个示例，下面有三个脚本都实现了相同的功能“cat -s”：多行空行合并为一行输出，但复杂度和执行速度递增。

如果首尾已经存在空行，第一个脚本会在首尾保留一个空行。

	#!/usr/bin/sed -f

	# 把所有空行连接到一起
	# 注意正则表示式中有一个“*”
	:x
	/^\n*$/ {
	N
	bx
	}

	# 挤压所有'\n', 也可以使用下面的正则表达式:
	# s/^\(\n\)*/\1/

	s/\n*/\
	/
	#注：该正则有误，应该为
	#s/\n+/\
	#/

这个脚本更复杂一些，会删除开头的所有空行，但结尾的空行会保留一个。

	#!/usr/bin/sed -f

	# 删除所有前置空行
	1,/^./{
	/./!d
	}

	# 删除空行后的空行，只保留一个空行
	:x
	/./!{
	N
	s/^\n$//
	tx
	}

这个脚本会删除所有的头尾空行，也是执行最快的。注意：所有循环都是通过n和b命令完成的，不依赖sed在行尾自动重启脚本。

	#!/usr/bin/sed -nf

	# delete all (leading) blanks
	/./!d

	# get here: so there is a non empty
	:x
	# print it
	p
	# get next
	n
	# got chars? print it again, etc...
	/./bx

	# no, don't have chars: got an empty line
	:z
	# get next, if last line we finish here so no trailing
	# empty lines are written
	n
	# also empty? then ignore it, and get next... this will
	# remove ALL empty lines
	/./!bz

	# all empty lines were deleted/ignored, but we have a non empty.  As
	# what we want to do is to squeeze, insert a blank line artificially
	i\

	bx

---

Next: Extended Commands, Previous: Other Commands, Up: sed Programs

##5 GNU sed's Limitations and Non-limitations

对于关注可移植性的用户，需要知道的是一些sed实现把行长度（pattern spaces和hold spaces）限制为不到4000字节。POSIX标准规定sed实现的行长度至少为8192字节。GNU sed内部对于行长度没有限制；只要可以malloc()到内存，可以支持任意长度。

sed用递归来处理子模式及“无限”重复，这意味着可用的堆栈空间会减少缓冲区的大小。

---
