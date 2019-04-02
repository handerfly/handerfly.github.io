---
layout:     post
title:      shell笔记-local、export用法 、declare、set
date:       2017-04-13
author:     BenderFly
header-img: img/post-bg-hacker.jpg
catalog: true
categories: Shell
tags:
    - Shell
    - Linux
---

# local一般用于局部变量声明，多在在函数内部使用。
```
    1.    Shell脚本中定义的变量是global的，其作用域从被定义的地方开始，到shell结束或被显示删除的地方为止。
    2.    Shell函数定义的变量默认是global的，其作用域从“函数被调用时执行变量定义的地方”开始，到shell结束或被显示删除处为止。函数定义的变量可以被显示定义成local的，其作用域局限于函数内。但请注意，函数的参数是local的。
    3.    如果同名，Shell函数定义的local变量会屏蔽脚本定义的global变量。
```

# export:将自定义变量设定为系统环境变量（仅限于该次登陆操作，当前shell中有效）
```
    语　　法：export [-fnp][变量名称]=[变量设置值]
    补充说明：在shell中执行程序时，shell会提供一组环境变量。export可新增，修改或删除环境变量，供后续执行的程序使用。
    参　　数：
        -f 　代表[变量名称]中为函数名称。
　    -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
　    -p 　列出所有的shell赋予程序的环境变量。
```

# Linux 指令篇：系统设置--declare  和set类似
```
功能说明：声明 shell 变量。

语　　法：declare [+/-][rxi][变量名称＝设置值] 或 declare -f

补充说明：declare为shell指令，在第一种语法中可用来声明变量并设置变量的属性([rix]即为变量的属性），在第二种语法中可用来显示shell函数。若不加上任何参数，则会显示全部的shell变量与函数(与执行set指令的效果相同)。

参　　数：
　+/- 　"-"可用来指定变量的属性，"+"则是取消变量所设的属性。 
　-f 　仅显示函数。 
　r 　将变量设置为只读。 
　x 　指定的变量会成为环境变量，可供shell以外的程序来使用。 
　i 　[设置值]可以是数值，字符串或运算式。

```