---
title: linux下vi编辑
date: 2018-04-13 21:46:55
tags: vi的编辑
typora-root-url: ..
---

​	简单介绍linux下vi的使用 <!--more-->

## 一、编辑模式和命令模式

​	刚打开文件时，处于命令模式，输入按键“i'进入输入模式，再按ESC按键退回到命令模式：

```
vi test.txt
```

##### 刚打开文件时处于命令模式或按ESC键也会处于命令模式:

![i_no_edi](/img/fangzy/vi/vi_no_edit.jpg)

##### 输入“i”编辑模式：

![i_edi](/img/fangzy/vi/vi_edit.png)

下面的几种方式也可以进入编辑模式:

1. 在命令模式输入“a":进入编辑模式，光标会移到下一个字符；
2. 在命令模式输入“o":进入编辑模式，光标会移到下一行；
3. 在命令模式输入“s":进入编辑模式，光标处于当前字符，并且会替换当前字符；

再按“ESC”键退回命令模式

## 二、代码的折叠与展开:

##### 在命令模式下，输入Shit+v 按键可进入行选中模式:

![i_row_selec](/img/fangzy/vi/vi_row_select.png)

##### 选中需要被折叠的代码，按下z+f按键，被选中的代码会被创建折叠:

![i_fold_cod](/img/fangzy/vi/vi_fold_code.png)

##### 展开被折叠的代码，按z+o按键:

![i_open_cod](/img/fangzy/vi/vi_open_code.png)

##### 已经创建折叠的代码已经被展开时，若想要重新合上，按z+c按键:

![i_close_cod](/img/fangzy/vi/vi_close_code.png)

## 三、在已经创建折叠的代码上快速移动方法:

1. 向下移到一行：按j键；
2. 向上移到一千:按k键；
3. 向左移动一个字符:按k键；
4. 向左移动一个字符:按l键；
5. 移动到下一个折叠处: 按z+j键；
6. 移动到上一个折叠处: 按z+k键

## 四、列选中方法

##### 输入ctrl+v进入列选中模式：

![i_cloumns_selec](/img/fangzy/vi/vi_cloumns_select.png)

##### 选中之后可以进行插入、删除操作：

1. 按shift+i进入列编辑模式，输入完后再按ESC按键退出列编辑模式:

   ![i_columns_edi](/img/fangzy/vi/vi_columns_edit.png)

## 五、退出vi

1. 输入”shift+:”，再输入"q!"回车,不保存并且退出编辑；
2. 输入”shift+:”，再输入"w!"回车,保存并且退出编辑；
3. 直接输入”shift+z+z”,保存并且退出编辑；

