---
title: C/C++中的lvalues和rvalues
tags:
  - c
  - lvalues
  - rvalues
id: 446
categories:
  - 学习
date: 2014-11-03 21:50:51
---

在C/C++中，lvalues(左值)和rvalues(右值)可能是一个一直被忽视的概念

<!--more-->

先以一个简单的例子为例进行说明，如一个简单的复数类重载"+"操作符，可能会使用的重载原型是

1.  <span class="typ">Complex<span style="color: #666600;">&</span></span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">::</span><span class="kwd">operator</span><span class="pln"> </span><span class="pun">+(</span><span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex&</span><span class="pln"> c</span><span class="pun">);</span>
 

其不修改"+"两边操作数的实现可能如下所示

1.  <span class="typ">Complex<span style="color: #666600;">&</span></span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">::</span><span class="kwd">operator</span><span class="pln"> </span><span class="pun">+(</span><span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex&</span><span class="pln"> c</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span>
2.  <span class="pln">    </span><span class="kwd">return</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">(</span><span class="pln">real </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">real</span><span class="pun">,</span><span class="pln"> image </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">image</span><span class="pun">);</span>
3.  <span class="pun">}</span>
 

但编译会有错误提示："invalid initialization of non-const reference of type 'Complex&' from an rvalue of type 'Complex'"
如果不考虑程序的正确性，为了编译通过可以使用如下形式

1.  <span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex&</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">::</span><span class="kwd">operator</span><span class="pln"> </span><span class="pun">+(</span><span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex&</span><span class="pln"> c</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span>
2.  <span class="kwd">    return</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">(</span><span class="pln">real </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">real</span><span class="pun">,</span><span class="pln"> image </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">image</span><span class="pun">);</span>
3.  <span class="pun">}</span>
 

其实正确的重载方法如下,程序返回时会调用copy constructor

1.  <span class="typ">Complex</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">::</span><span class="kwd">operator</span><span class="pln"> </span><span class="pun">+(</span><span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex&</span><span class="pln"> c</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span>
2.  <span class="pln">    </span><span class="kwd">return</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">(</span><span class="pln">real </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">real</span><span class="pun">,</span><span class="pln"> image </span><span class="pun">+</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">image</span><span class="pun">);</span>
3.  <span class="pun">}</span>
 

如果允许修改"+"左边的操作数，也可使用如下

1.  <span class="typ">Complex&</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">::</span><span class="kwd">operator</span><span class="pln"> </span><span class="pun">+(</span><span class="kwd">const</span><span class="pln"> </span><span class="typ">Complex</span><span class="pun">&</span><span class="pln"> c</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span>
2.  <span class="pln">    real </span><span class="pun">+=</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">real</span><span class="pun">;</span>
3.  <span class="pln">    image </span><span class="pun">+=</span><span class="pln"> c</span><span class="pun">.</span><span class="pln">image</span><span class="pun">;</span>
4.  <span class="pln">    </span><span class="kwd">return</span><span class="pln"> </span><span class="pun">*</span><span class="kwd">this</span><span class="pun">;</span>
5.  <span class="pun">}</span>
 

这里要解释为什么"const Complex& Complex::operator +(const Complex& c)"这种函数原型的返回值可以接受临时对象"Complex(real + c.real, image + c.image)"就需要理解lvalues、rvalues

1\. lvalues是在内存中有实际存储单元的标识符(变量名)
2\. rvalues真好相反，是没有存储单元的表达式
3\. lvalues可以单独或组合成表达式作为rvalues
4\. lvalues如果有const修饰则是不可修改的lvalues
如

1.  <span class="com">// ----1-----</span>
2.  <span class="typ">int</span><span class="pln"> var</span><span class="pun">;</span>
3.  <span class="pln">var </span><span class="pun">=</span><span class="pln"> </span><span class="lit">4</span><span class="pun">;</span>
4.  <span class="pln"> </span>
5.  <span class="com">// ----2-----</span>
6.  <span class="lit">4</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> var</span><span class="pun">;</span><span class="pln"> </span><span class="com">// ERROR!</span>
7.  <span class="pun">(</span><span class="pln">var </span><span class="pun">+</span><span class="pln"> </span><span class="lit">1</span><span class="pun">)</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="lit">4</span><span class="pun">;</span><span class="pln"> </span><span class="com">// ERROR!</span>
8.  <span class="pln"> </span>
9.  <span class="com">// ----3-----</span>
10.  <span class="com">// lvalues as rvalues</span>
11.  <span class="typ">int</span><span class="pln"> var2 </span><span class="pun">=</span><span class="pln"> var</span><span class="pun">;</span>
12.  <span class="pln">var </span><span class="pun">=</span><span class="pln"> var </span><span class="pun">+</span><span class="pln"> </span><span class="lit">1</span><span class="pun">;</span>
13.  <span class="pln"> </span>
14.  <span class="com">// ----4-----</span>
15.  <span class="kwd">const</span><span class="pln"> </span><span class="typ">int</span><span class="pln"> a </span><span class="pun">=</span><span class="pln"> </span><span class="lit">10</span><span class="pun">;</span><span class="pln"> </span><span class="com">// 'a' is an lvalue</span>
16.  <span class="pln">a </span><span class="pun">=</span><span class="pln"> </span><span class="lit">10</span><span class="pun">;</span><span class="pln"> </span><span class="com">// but it can't be assigned!</span>
 

1\. "var = 4"中var是一个占有实际存储单元的变量，是lvalues
2\. "4","(var + 1)"则不是一个实际有存储单元的rvalues
3\. "var"作为左值可以单独或组合成复合表达式变成右值
4\. 左值"a"有const修饰则"a"是一个不可修改的左值

左值和右值之间的转换

1\. 所有非数组类型、函数类型、不完全类型的左值都可以隐式转换成右值，如

1.  <span class="typ">int</span><span class="pln"> a </span><span class="pun">=</span><span class="pln"> </span><span class="lit">1</span><span class="pun">;</span><span class="pln">     </span><span class="com">// a is an lvalue</span>
2.  <span class="typ">int</span><span class="pln"> b </span><span class="pun">=</span><span class="pln"> </span><span class="lit">2</span><span class="pun">;</span><span class="pln">     </span><span class="com">// b is an lvalue</span>
3.  <span class="typ">int</span><span class="pln"> c </span><span class="pun">=</span><span class="pln"> a </span><span class="pun">+</span><span class="pln"> b</span><span class="pun">;</span><span class="pln">    </span><span class="com">// + needs rvalues, so a and b are converted to rvalues</span>
4.  <span class="pln">                          </span><span class="com">// and an rvalue is returned</span>
 

2\. 一般右值是不能转换成左值得，只在少数情况下，如一元"*"(dereference)运算符

1.  <span class="typ">int</span><span class="pln"> arr</span><span class="pun">[]</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="pun">{</span><span class="lit">1</span><span class="pun">,</span><span class="pln"> </span><span class="lit">2</span><span class="pun">};</span>
2.  <span class="typ">int</span><span class="pun">*</span><span class="pln"> p </span><span class="pun">=</span><span class="pln"> </span><span class="pun">&</span><span class="pln">arr</span><span class="pun">[</span><span class="lit">0</span><span class="pun">];</span>
3.  <span class="pun">*(</span><span class="pln">p </span><span class="pun">+</span><span class="pln"> </span><span class="lit">1</span><span class="pun">)</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="lit">10</span><span class="pun">;</span><span class="pln">   </span><span class="com">// OK: p + 1 is an rvalue, but *(p + 1) is an lvalue</span>
 

3\. 相反"&"运算符则把右值转换为左值

1.  <span class="typ">int</span><span class="pln"> var </span><span class="pun">=</span><span class="pln"> </span><span class="lit">10</span><span class="pun">;</span>
2.  <span class="typ">int</span><span class="pun">*</span><span class="pln"> bad_addr </span><span class="pun">=</span><span class="pln"> &</span><span class="pun">(</span><span class="pln">var </span><span class="pun">+</span><span class="pln"> </span><span class="lit">1</span><span class="pun">);</span><span class="pln"> </span><span class="com">// ERROR: lvalue required as unary '&' operand</span>
3.  <span class="typ">int</span><span class="pun">*</span><span class="pln"> addr </span><span class="pun">=</span><span class="pln"> <span style="color: #666600;">&</span></span><span class="pln">var</span><span class="pun">;</span><span class="pln">                   </span><span class="com">// OK: var is an lvalue</span>
4.  <span class="pln">&var </span><span class="pun">=</span><span class="pln"> </span><span class="lit">40</span><span class="pun">;</span><span class="pln">                               </span><span class="com">// ERROR: lvalue required as left operand</span>
5.  <span class="pln">                                                </span><span class="com">// of assignment</span>
 

4\. "="可以定义reference types，称作lvalue references，但是Non-const lvalues references不能被右值赋值

1.  <span class="pln">std</span><span class="pun">::</span><span class="pln">string</span><span class="pun">&</span><span class="pln"> sref </span><span class="pun">=</span><span class="pln"> std</span><span class="pun">::</span><span class="pln">string</span><span class="pun">();</span><span class="pln">  </span><span class="com">// ERROR: invalid initialization of</span>
2.  <span class="pln">                                                   </span><span class="com">// non-const reference of type</span>
3.  <span class="pln">                                                   </span><span class="com">// 'std::string&' from an rvalue of</span>
4.  <span class="pln">                                                   </span><span class="com">// type 'std::string'</span>
 

参考资料
1\. [Understanding lvalues and rvalues in C and C++](http://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c)