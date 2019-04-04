---
layout:     post
title:      shell脚本中使用getopts处理多命令行选项
date:       2019-04-04
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
categories: Shell
tags:
    - Shell
---

在Linux系统中，许多命令都提供了选项，使用不同的选项就会得到不通的执行结果   

例如：ls命令，ls命令提供了多个选项:-l、-a、-A、-h、-i等等，每个选项具有不同的功能，我们自己写脚本时也可以定义选项，提示用户如何使用，本文介绍如何使用getopts命令来处理命令选项。   

# 1.getopts命令的如何使用   

用getopts命令获取到脚本选项后，getopts会将获取到的选项所对应的参数,自动保存到OPTARG这个变量中。   

getopts命令格式：getopts   OPTSTRING  VARNAME      

OPTSTRING：告诉getopts会有哪些选项和参数（用选项后面加“：”来表示选项后面需要加参数）   

VARNAME：保存getopts获取到的选项   

示例：getopts  ahf:   var   

告诉getopts查找-a、-h、-f选项，其中f选项后面需要跟一个参数，获取到的选项都保存到变量var中   

getopts命令支持两种错误报告模式，详细错误报告模式和抑制错误报告模式。

在详细错误报告模式下：如果getopts检测到一个无效的选项，var的值会被设置为(?)；如果getopts检测到一个后面需要跟参数的选项，后面没有参数，var的值也会被设置为(?)

在抑制错误报告模式下：如果getopts检测到一个无效的选项，var的值会被设置为(?)，变量OPTARG会被设置为这个无效的选项；如果getopts检测到一个后面需要跟参数的选项，后面没有参数，var的值会被设置为(：)，变量OPTARG会被设置为这个无效的选项

# 2.通过脚本来讲解getopts如何获取选项，如何赋值给变量VARNAME和OPTSTRING   

## 示例
```
#!/bin/bash
status=off                 #定义变量status，初始值设置为off
filename=""              #定义变量filename，用于保存选项参数（文件）
output=""                 #定义变量output，用于保存选项参数（目录）
Usage () {                  #定义函数Usage，输出脚本使用方法
    echo "Usage"
    echo "myscript [-h] [-v] [-f <filename>] [-o <filename>]"
    exit -1
}

while getopts :hvf:o: varname   #告诉getopts此脚本有-h、-v、-f、-o四个选项，-f和-o后面需要跟参数

（没有选项时，getopts会设置一个退出状态FALSE，退出循环）

do
   case $varname in
    h)
      echo "$varname"
      Usage
      exit
      ;;
    v)
      echo "$varname"
      status=on
      echo "$status"
      exit
      ;;
    f)
      echo "$varname"
      echo "$OPTARG"
      filename=$OPTARG                    #将选项的参数赋值给filename
      if [ ! -f $filename ];then               #判断选项所跟的参数是否存在且是文件
         echo "the source file $filename not exist!"
         exit
      fi
      ;;
    o)
      echo "$varname"
      echo "$OPTARG"
      output=$OPTARG                      #将选项参数赋值给output
      if [ ! -d  $output ];then               #判断选项参数是否存在且是目录
         echo "the output path $output not exist"
         exit
      fi
      ;;
    :)                                               #当选项后面没有参数时，varname的值被设置为（：），OPTARG的值被设置为选项本身
      echo "$varname"

      echo "the option -$OPTARG require an arguement"        #提示用户此选项后面需要一个参数
      exit 1
      ;;
    ?)                                   #当选项不匹配时，varname的值被设置为（？），OPTARG的值被设置为选项本身
      echo "$varname"
      echo "Invaild option: -$OPTARG"           #提示用户此选项无效
      Usage
      exit 2
      ;;
    esac
done

```
![getopts](https://raw.githubusercontent.com/handerfly/handerfly.github.io/master/img/getopt.png)  