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

如果不指定INPUTFILE，或者INPUTFILE是-，sed会从标准输入过滤内容。script是第一个非可选参数，
