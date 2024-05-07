---
title: 编译与调试 - GCC and GBD
published: 2022-08-15
description: 《深入理解计算机系统》(CS:APP)的笔记, GCC与GDB的基础概念.
tags: [计算机系统]
category: 计算机系统
draft: false
---

# GCC  
编译程序的过程：  
**file.c ---> (预处理器cpp处理,将宏定义和声明中的文件等直接嵌入到file中去) file.i ---> (编译器ccl处理，生成机器代码，一种汇编语言) file.s 汇编程序(文本) --->(汇编器as处理，生成可重定位目标程序) file.o (二进制) --->（链接器ld处理，加入特殊函数如i/o等，生成可执行目标程序) file**  

## 利用gcc的命令可以在其中任何一步将其打断。  
|参数|效果|  
|:-:|:-:|  
|-Og -O1 -O2|-Og不优化，按源代码生成机器代码，后者优化|  
|-E|只进行预处理|  
|-S|只生成汇编程序文本.s，此时可以查看汇编代码|  
|-c|只生成.o二进制文件，无法查看|  
|-o|生成可执行目标程序,一定要有main()|  
|-std=|选择使用不同标准|  
|filename(末)|可同时编译一组文件，有且仅有一个main()|  

## 生成静态库，链接可重定位目标模块与静态库  
```  
1、形成静态库：AR工具。将1.o 2.o 3.o...组织为静态库name.a:  
ar rcs name.a 1.o 2.o 3.o  
生成的name.a在同一个目录下。  
2、进行手动链接：gcc  
gcc -static -o name_executable 1.o 2.o ./my_lib.a  
可见最后的静态库文件必须以路径的形式给出。  
```  
## 将源文件生成共享库，编译时动态链接  
```  
生成共享库：  
gcc -shared -o -fpic libXXX.so src1.c src2.c src3.c ...  
告诉GCC要生成要适应动态解析引用的可执行目标文件：  
gcc -o prog main.c ./libXXX.so  
只要包含这个动态库的文件路径就行  
```  
## 应用中(running time)有动态链接，需这样编译  
```  
gcc -rdynamic -o prog main.c -ldl  
```  
## 编译时打桩  
```  
gcc -DCOMPILETIME -c mymalloc.c  //包装函数所在  
gcc -I. -o main main.c mymalloc.o  
```  
## 链接时打桩  
```  
gcc -DLINKTIME -c mymalloc.c  
gcc -c main.c  
gcc -W1 --wrap,malloc -W1 --wrap,free -o main main.o mymalloc.o  
```  
## 运行时打桩  
```  
gcc -DRUNTIME -shared -fpic -o -mymalloc.so mymalloc.c -ldl  
gcc -o main main.c  
运行打桩后的main:  
LD_PRELOAD="./mymalloc.so" ./main  
```  
  
# GBD  
## 硬查看.o文件  
```  
(gbd)x/<num>xb function_name  
```  
其中，第一个x是gbd的简写。num表示表示一个数字，表示从函数function_name或者其它内存组件的地址开始，往后输出*num*个十六进制格式(*x*)的字节(*b*)  
## GBD语法  
和普通调试类似，也是断点、执行。。。当时GBD可以以任何形式读任何内存地址的值，读寄存器信息、栈帧信息，反汇编等等。  
是对可执行目标文件的进行调试，对应的是机器代码，所以很难追踪变量，使用它调试会有难度。  
启动GBD:  
```  
>> gbd prog  
```  
1）开始和停止  
```  
>> quit 退出  
>> run  运行程序，并在后面给出命令行参数  
>> kill 停止程序  
```  
2）断点  
```  
>> break function_name  在函数...入口处设置断点  
>> break * addr      在地址addr处设置断点（提前反汇编获取机器代码）  
>> delete n        删除断点m  
>> delete         删除所有断点  
```  
3）执行  
```  
>> stepi  执行一条（机器）指令  
>> stepi n  执行n条指令  
>> nexti  以函数为单位，执行一个函数  
>> continue  继续执行（到下一个断点）  
>> finish  运行到当前函数返回  
```  
4）检查代码、反汇编  
```  
>> disas  反汇编当前函数  
>> disas function_name  反汇编函数...  
>> disas addr  反汇编addr附近的函数  
>> disas addr1 addr2 反汇编指定范围内的所有代码  
>> print /x $rip  以十六进制输出程序计数器值  
```  
5）检查数据  
```  
>> print <type> $XXX  以某个进制形式输出寄存器XXX的值。<type>缺省为十进制，/x为十六进制，/t为二进制。  
>> print * (type *) addr  输出位于地址addr处的type类型的数  
```  
6）其它  
```  
>> info frame  当前栈帧信息  
>> info registers  所有寄存器的值  
>> help  
```  
*GBD的图形化扩展 ：DDD*  
  
  
## 反汇编  
使用  
```  
objdump -d filename  
```  
可以直接在命令行中输出反汇编结果。对二进制文件.o,和可执行目标文件都可以反汇编。生成的反汇编结果中机器指令的表示会和.s中的微妙地有些不同。  
当然也有专门的反汇编程序。  
