---
layout: post
title: 【转】用 Python 中的 encode 与 decode 解决字符串乱码问题
tags:
  - program
  - python
  - encode
---

在 Python 代码中涉及到中文时，常常会报这样一个错误「UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)」。本文就来研究一下这个问题。

字符串在 Python 内部的表示是 Unicode 编码，因此，在做编码转换时，通常需要以 Unicode 作为中间编码，即先将其他编码的字符串解码（decode）成 Unicode，再从 Unicode 编码（encode）成另一种编码。 

decode 的作用是将其他编码的字符串转换成 Unicode 编码，如`str1.decode('gb2312')`，表示将 GB2312 编码的字符串 str1 转换成 Unicode 编码。 

encode 的作用是将 Unicode 编码转换成其他编码的字符串，如`str2.encode('gb2312')`，表示将 Unicode 编码的字符串 str2 转换成 GB2312 编码。 

因此，转码的时候一定要先搞明白，字符串 str 是什么编码，然后 decode 成 Unicode，然后再 encode 成其他编码。

代码中字符串的默认编码与代码文件本身的编码一致。 

如：`s='中文'`，

如果是在 UTF-8 的文件中，该字符串就是 UTF-8 编码，如果是在 GB2312 的文件中，则其编码为 GB2312。这种情况下，要进行编码转换，都需要先用 decode 方法将其转换成 Unicode 编码，再使用 encode 方法将其转换成其他编码。通常，在没有指定特定的编码方式时，都是使用的系统默认编码创建的代码文件。 

如果字符串是这样定义：`s=u'中文'`，

则该字符串的编码就被指定为 Unicode 了，即 Python 的内部编码，而与代码文件本身的编码无关。因此，对于这种情况做编码转换，只需要直接使用 encode 方法将其转换成指定编码即可。

如果一个字符串已经是 Unicode 了，再进行解码则将出错，因此通常要对其编码方式是否为 Unicode 进行判断：

```python
isinstance(s, unicode)  #用来判断是否为Unicode
```

用非 Unicode 编码形式的 str 来 encode 会报错。

 如何获得系统的默认编码？ 

```python
#!/usr/bin/env python
#coding=utf-8
import sys
print sys.getdefaultencoding()
```

该段程序在英文 Windows XP 上输出为：ASCII。

在某些 IDE 中，字符串的输出总是出现乱码，甚至错误，其实是由于 IDE 的结果输出控制台自身不能显示字符串的编码，而不是程序本身的问题。 

如在 UliPad 中运行如下代码：

```python
s=u"中文"
print s
```

会提示「UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)」。这是因为 UliPad 在英文 Windows XP 上的控制台信息输出窗口是按照 ASCII 编码输出的（英文系统的默认编码是 ASCII），而上面代码中的字符串是 Unicode 编码的，所以输出时产生了错误。

将最后一句改为：`print s.encode('gb2312')`

则能正确输出“中文”两个字。

若最后一句改为：`print s.encode('utf8')`

则输出：\xe4\xb8\xad\xe6\x96\x87，这是控制台信息输出窗口按照 ASCII编码输出 UTF-8 编码的字符串的结果。

`unicode(str,'gb2312')` 与 `str.decode('gb2312')` 是一样的，都是将 GB2312 编码的 str 转为 Unicode 编码 

使用 `str.__class__` 可以查看 str 的编码形式。

原理说了半天，最后来个包治百病的吧，

代码如下:

```python
#!/usr/bin/env python 
#coding=utf-8 
s="中文" 
if isinstance(s, unicode): 
#s=u"中文" 
print s.encode('gb2312') 
else: 
#s="中文" 
print s.decode('utf-8').encode('gb2312')
```

---

原文链接：[http://www.pythontab.com/html/2016/pythonjichu_0713/1043.html](http://www.pythontab.com/html/2016/pythonjichu_0713/1043.html)

