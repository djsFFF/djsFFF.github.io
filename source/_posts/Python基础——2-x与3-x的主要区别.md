---
title: Python基础——Python 2.x与Python 3.x的主要区别
date: 2020-07-17 22:38:19
tags:
	- Python
categories:
	- Python
---

本文总结了Python 2.x与Python 3.x的主要区别。

<!--more-->

# print函数

print()函数取代print语句。

# Unicode

Python 2.x默认使用ASCII编码，不能直接输出中文，变量命名只能用英文。

Python 3.x默认使用UTF-8编码，可以直接输出中文，可以使用中文变量名。

# 除法运算`/`

Python 2.x，整数相除结果为整数，浮点数相除为浮点数。

Python 3.x，结果始终是浮点数。

# 打开文件

Python 2.x可以使用file(...)或者open(...)。

Python 3.x只能使用open(...)。

# xrange函数

Python 3.x取消了xrange函数，使用range函数完全代替。

# 八进制字面量表示

Python 2.x可以用01000或0o1000表示八进制512。

Python 3.x只可以使用0o1000。

# 不等运算符

Python 2.x可以用`!=`或`<>`。

Python 3.x只能使用`!=`。

# Python 3.x使用更加严格的缩进

Python 2.x中允许tab和space共存。

Python 3.x只能单独使用tab或者space。

# 去掉了repr表达式

Python 2.x中反引号``相当于repr函数的作用。

Python 3.x中去掉了``的写法，只允许使用repr函数。

# 数据类型

Python 3.x去掉了long类型，只有一种整形——int。新增了bytes类型。

# input和raw_input函数

Python 2.x中raw_input会将所有输入数据当做字符串，返回值为字符串；而input输入时必须是一个合法的Python表达式。

Python 3.x只能使用input。

# map、filter和reduce

Python 2.x中map，filter和reduce是内置函数，返回结果为列表。

Python 3.x中map和filter变成了类，返回结果为可迭代对象。reduce从全局挪到了functool模块中。

# 异常

捕获异常的语法由`except exc, var`改为`except exc as var`。

Python 2.x中所有类型的对象都可以被抛出；Python 3.x中只有继承自BaseException的对象才可以被抛出。