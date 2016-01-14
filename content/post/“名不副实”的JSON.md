+++
author = ""
comments = true
date = "2013-05-15T09:21:14+08:00"
draft = false
image = ""
menu = "test"
share = true
slug = "名不副实的JSON"
tags = ["json", "javascript"]
title = "“名不副实”的JSON"

+++

   JSON (Javascript Object Notation):直接翻译就是javascript的对象表示方法。是一种轻量级的数据交换格式。 易于人阅读和编写。同时也易于机器解析和生成。 它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。 这些特性使JSON成为理想的数据交换语言。也就是说现在的JSON俨然已经“名不副实”了

JSON的特点:

1. JSON 是纯文本
2. JSON 具有“自我描述性”（人类可读）
3. JSON 具有层级结构（值中存在值）
4. JSON 可通过 JavaScript 进行解析
5. JSON 数据可使用 AJAX 进行传输 

与 XML相比:

1. 没有结束标签
2. 更短
3. 读写的速度更快
4. 能够使用内建的 JavaScript eval() 方法进行解析
5. 使用数组
6. 不使用保留字

正因为如此JSON大有取缔XML之势

JSON 的语法规则

JSON 语法是 JavaScript 对象表示法语法的子集。

1. 数据在名称/值对中
2. 数据由逗号分隔 
3. 花括号保存对象 
4. 方括号保存数组

Example 1.

    {
      "employees": [
        { "firstName":"Bill" , "lastName":"Gates" },
        { "firstName":"George" , "lastName":"Bush" },
        { "firstName":"Thomas" , "lastName":"Carter" }
         ]
    }

这个 employee 对象是包含 3 个员工记录（对象）的数组

JSON 值可以是：
    数字（整数或浮点数）
    字符串（在双引号中）
    逻辑值（true 或 false）
    数组（在方括号中）
    对象（在花括号中）
    null
因为 JSON 使用 JavaScript 语法，所以无需额外的软件就能处理 JavaScript 中的 JSON。
通过 JavaScript，您可以创建一个对象数组，并像这样进行赋值：

Example 2. 

	var employees = [
	{ "firstName":"Bill" , "lastName":"Gates" },
	{ "firstName":"George" , "lastName":"Bush" },
	{ "firstName":"Thomas" , "lastName": "Carter" }
	];
可以像这样访问 JavaScript 对象数组中的第一项：`employees[0].lastName;`
返回的内容是：Gates

 
