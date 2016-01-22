title: "PHP简单类型转换及类型函数判断下的区别"
date: 2015-04-01 22:58:02
tags: [PHP]
description: PHP简单类型转换及类型函数gettype(), empty(), is_null(), isset(), boolval()的区别

---
公司后端用Java重构后已经有几个月没写过PHP了，虽然PHP不是很出色的语言，但也是我进入工作后的第一语言，我也没打算放弃它。前那两天看到一个关于类型判断的帖子，觉得挺有趣的，虽然不复杂，却也挺考验PHP基础的功底的。

函数 *`gettype(), empty(), is_null(), isset(), boolval()`* 这几个在获取参数判断类型上常用的方法，感觉还是特别容易出错的，因此我在自己测试了之后总结如下：

|        | gettype()|empty()|is_null()|isset()|boolval()|
|--------|----------|-------|---------|-------|---------|
|$val="";|String|true|false|true|false|
|$val=null;|NULL|true|true|false|false|
|var $val|NULL|true|true|false|false|
|$val=array()|Array|true|false|true|false|
|$val=false;|boolean|true|false|true|false
|$var=1|int|false|false|true|true|
|$var=0|int|true|false|true|false|
|$var=-1|int|false|false|true|true|
|$var="9"|string|false|false|true|true|
|$var="0"|string|true|false|true|false|
|$var="hello"|string|false|false|true|true|
|$val="ture"|string|false|false|true|true|
|$val="false"|string|false|false|true|true|
以上是正确的格式对照表。
<!--more-->

##### 我自己在测试的过程中也出现了一些疑问：#####

1. bool值false在判断是否为空时，显示为空
```php
$val=false; 
var_dump(empty($val));
#print bool（true）
```
2. 数字0在判断为空是，显示为true
```php
$val=0;
var_dump(empty($val));
#print bool(true）
```
3. 字符串0在判断为空是，显示为true
```php
$val="0";
var_dump(empty($val));
#print bool(true）
```

更重要的是`$val="";`和`$val="0";`的判断的结果是一样的，这些我可以通过PHP的弱类型自动转换来解释，但之后我做了一个简单的实验，一时无法让我开窍了：
![php代码](http://7xi3xm.com1.z0.glb.clouddn.com/php.jpg)
按图上的推理逻辑，`''`那就应该是等于`'0'`了，这难道是我自己钻了牛角尖么？
答案是确实是自己走进了死胡同，PHP在不同类型做比较的时候会进行转换，但相同类型则没有转换成false的问题，因此就出现了这种让自己觉得可笑的事情。

