---
title: 百分号编码（Percent-Encoding）
date: 2019-04-22 15:18:00
tags:
- percent-encoding
---

百分号编码（Percent-Encoding）也被称为 URL 编码，是一种编码机制。该机制主要应用于 URI 编码中，URI 包含 URL 和 URN，所以它们也同样适用。除此之外，也用于 MIME 类型为"application/x-www-form-urlencoded"的内容。

百分号编码会对 URI 中不允许出现的字符或者其他特殊情况的允许的字符进行编码，对于被编码的字符，最终会转为以百分号"%"开头，后面跟着两位16进制数值的形式。举个例子，空格符（SP）是不允许的字符，在 ASCII 码对应的二进制值是"00100000"，最终转为"%20"。

<!-- more -->

## 保留字符

什么是保留字符？将 URI 分割为多个部分或子部分的那些字符，比如"/"和"?"，以及需保留以后使用的暂未定义其用处的字符，这些字符被称作保留字符。

根据 [RFC 3986](https://tools.ietf.org/html/rfc3986) 标准，保留字符为以下18个：

!	*	'	(	)	;	:	@	&	=	+	$	,	/	?	#	[	]

## 未保留字符

未保留字符则是在 URI 中允许的但没有保留目的的字符。这些字符包含大小写字母，数字，以及连字符"-”，句号“.”，下划线"_”，波浪号"~”。

以下为未保留字符：

A	B	C	D	E	F	G	H	I	J	K	L	M	N	O	P	Q	R	S	T	U	V	W	X	Y	Z	a	b	c	d	e	f	g	h	i	j	k	l	m	n	o	p	q	r	s	t	u	v	w	x	y	z	0	1	2	3	4	5	6	7	8	9	-	_	.	~

## 使用

未保留字符一般不需要百分号编码。

什么时候要编码？大概可分以下几种：

1. 保留字符在 URI 的组成部分中不作为分割符存在时，需进行编码；
2. 百分号字符"%"由于是作为百分号编码的标识符，因此需编码为"%25"；
3. 其他字符则需要先转换为 UTF-8 字节序列，然后再对其字节进行编码。

## application/x-www-form-urlencoded 类型

由于 [W3C HTML Form](<https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1>) 规范中，当 MIME 类型为“application/x-www-form-urlencoded”时，这里的表单的名（name）和值中的空格会被编码为"+"，而不是"%20"。因此对于 application/x-www-form-urlencoded 类型的内容进行百分比编码时，需把空格替换为"+"。

## 参考文献

1. [RFC 3986](https://tools.ietf.org/html/rfc3986)
2. [W3C HTML Form content types](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1)

