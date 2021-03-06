---
title: PHP位运算
date: 2019-12-27 14:30:17
tags:
  - 位运算
  - PHP
---

## 基本运算规则

### &（与）
相同位为1,则为1,否则为0 （必须都为1,则位为1）

如：
```php
3 & 5
```
3 => 00000011
5 => 00000101
结果：00000001 转换为10进制为1

### | （或）
相同位为0则为0,否则为1 （有1个为1,则位为1）

如：
```php
3 | 5
```
3 => 00000011
5 => 00000101
结果：00000111 转换为10进制为7

### ^ （异或）
相同为0,相异为1
如：
```php
3 ^ 5
```
3 => 00000011
5 => 00000101
结果：00000110 转换为10进制为6

### ~ （取反）
位全部取反，要注意高位为1是负数
如：
```php
~3
```
3 => 00000011
取反，所以负数的原码为：
11111100 
高位为1,则表示是负数，
正整数的原码，反码和补码计算。【符号位为0，原码=反码=补码】
负整数的原码，反码和补码计算，先求原码，再求反码，最后求补码，（补码=反码的最低位+1）
反码为 ：
10000011
补码为：
10000011 + 1 = 10000100 转换为10进制为4
高为1,是负数，所以为 -4

### << （左位移）
向左移动N位，低位以0填充，高位可能会变化，左移时右侧以零填充，符号位被移走意味着正负号不被保留。
如:
3 << 10 = 3072
3 << 6 = 96 (移到最高位为1,高位不被保留)


### >> （右位移）
左侧以符号位填充，意味着正负号被保留。
-3 << 5 = -96

> 左移或右移都是以遇到1开始，向左或向右，向左移动到最高位为1，高位不被保留

## 优先级
1. ~ 
2. <<
3. \>>
4.  &
5.  ^
6.  |


## 进制转换

### 二进制转十进制
将正的十进制数除以二，得到的商再除以二，依次类推知道商为零或一时为止，然后在旁边标出各步的余数，最后倒着写出来，高位补零就OK咧。哎呀，还是举例说明吧，比如42转换为二进制，如图1所示操作。

https://www.cnblogs.com/web-record/p/11132861.html

### 十进制转二进制

https://www.cnblogs.com/web-record/p/11132861.html
要注意的是，如果是奇数，则最后是算出来的结果+1
