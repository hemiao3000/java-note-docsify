# <font colot="orange">IntelliJ IDEA 中有什么让你相见恨晚的技巧？</font>

本文总结了开发工具 idea 中提升开发效率的 10 个小技巧。

## 1. 快速生成 main 方法并打印

- 用 `psvm` 命令能快速生成 main 方法。
- 用 `sout` 命令能快速生成打印方法 System.out.println 。

两个命令相结合的效果如下：<img src="https://pic4.zhimg.com/50/v2-b5c1ec3908d15373a577e75c50695f31_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-b5c1ec3908d15373a577e75c50695f31_hd.jpg?source=1940ef5c" class="content_image"/>


## 2. 给 new 出来的对象快速赋值

在 new 出来的对象后面加上 `.var` ，就能实现快速赋值，效果如下：

<img src="https://pic4.zhimg.com/50/v2-f722ddb21b6cdbf4cbc22ac2bcab79a0_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-f722ddb21b6cdbf4cbc22ac2bcab79a0_hd.jpg?source=1940ef5c" class="content_image"/>

## 3. 快速 for 循环

1. 基本变量比如：int，long，byte 等，在需要进行 for 循环遍历的变量后加上 `.for` ，就能快速实现 for 循环功能。

效果如下：<img src="https://pic2.zhimg.com/50/v2-19f5f8611b1d80f453819739c5d7f3b5_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic4.zhimg.com/50/v2-19f5f8611b1d80f453819739c5d7f3b5_hd.jpg?source=1940ef5c" class="content_image"/>

2. 集合

在需要进行 forEach 循环遍历的集合后加上 `.for` ，就能快速实现 forEach 循环功能，效果如下：

<img src="https://pic1.zhimg.com/50/v2-46a4492a8dae31618e66fc4952290cbb_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-46a4492a8dae31618e66fc4952290cbb_hd.jpg?source=1940ef5c" class="content_image"/>



## 4. 快速判断

判断条件在开发过程中使用频率非常高，如何快速的写出判断条件呢？

- boolean.if 可以生成 if (boolean)
- boolean.else 可以生成 if (!boolean) 
- string.null 可以生成 if (string == null)
- string.nn 可以生成 if (string != null)

具体实现效果如下：

<img src="https://pic1.zhimg.com/50/v2-27b3c8e5df4b6dba694f9d1138bdb4ff_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-27b3c8e5df4b6dba694f9d1138bdb4ff_hd.jpg?source=1940ef5c" class="content_image"/>

此外，switch 也有类似的功能。

## 5. 快速 try ... catch

有时候我们有异常需要捕获，手动写 try...catch 比较麻烦，这时快速 try...catch 可以给我们节省不少时间，只需加 `.try` 即可，效果如下：

<img src="https://pic2.zhimg.com/50/v2-34ba5b349b7b5b9167beb7ee2d09a457_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-34ba5b349b7b5b9167beb7ee2d09a457_hd.jpg?source=1940ef5c" class="content_image"/>


## 6. 快速类型转换


有时候我们需要做类型转换，必须手写括号和赋值参数，同样有些麻烦，这时快速类型转换，可以帮我们搞定，只需加 `.castvar` 即可，效果如下：

<img src="https://pic4.zhimg.com/50/v2-8aaaab23060220d58200c9fb7a16edd5_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic4.zhimg.com/50/v2-8aaaab23060220d58200c9fb7a16edd5_hd.jpg?source=1940ef5c" class="content_image"/>

## 7.  快速抽取变量

有时候我们需要把方法中的局部变量，抽取成成员变量，或者全局变量，快速抽取变量可以帮你搞定，只需加 `.field` 即可，具体效果如下：

<img src="https://pic1.zhimg.com/50/v2-35ee97345cacbd286636d19ab9639537_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic2.zhimg.com/50/v2-35ee97345cacbd286636d19ab9639537_hd.jpg?source=1940ef5c" class="content_image"/>


## 8. 快速定义 Optional

有时候我们想把某个对象转换成 Optional ，避免出现空指针问题，只需加 `.opt` 即可，具体效果如下：

<img src="https://pic1.zhimg.com/50/v2-5d4659c3fdb49f9529890247af98c509_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic4.zhimg.com/50/v2-5d4659c3fdb49f9529890247af98c509_hd.jpg?source=1940ef5c" class="content_image"/>

## 9.  快速生成lambda语句

如果你在用 jdk1.8 以上的版本，那么 lambda 表达式必不可少，因为用它可以极大的提高开发效率，少写很多代码。使用 `.lambda` 就能快速生成 lambda 语句，具体效果如下：  

<img src="https://pic1.zhimg.com/50/v2-abf1936914396d92dd0e10b2b0d5ee66_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-abf1936914396d92dd0e10b2b0d5ee66_hd.jpg?source=1940ef5c" class="content_image"/>

## 10. 快速迁移代码到新方法

在代码重构时，经常需要把某段代码迁移到一个新方法中，这时使用快捷键 `ctrl + alt + m` ，具体效果如下：  

<img src="https://pic4.zhimg.com/50/v2-f66e4ee5df62bcc3d4ad1d3ccc2e83b9_hd.gif?source=1940ef5c" data-caption="" data-size="normal" data-thumbnail="https://pic1.zhimg.com/50/v2-f66e4ee5df62bcc3d4ad1d3ccc2e83b9_hd.jpg?source=1940ef5c" class="content_image"/>

