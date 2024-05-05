---
title: LearnOpenGL·零 vscode 配置 openGL
published: 2024-04-23
description: empty
tags: [CG, OpenGL]
category: CG
draft: false
---

用vscode配置一个简单的opengl开发环境——只用来做点简单的学习性质的小项目，不太能用来开发真的项目。vscode配置C/C++的内容确实相当折腾。  

opengl最底层的库函数、接口已经包含在标准库函数里了，在编译器的include文件夹下的GL/...里。这里配置的是 GLFW 和 glad 两个第三方库，是关于窗口什么的，它们的获取在网上直接搜索vscode配置opengl都能搞到。**只是尤其注意如果下载预编译的glfw版本，一定要符合编译器位数，64位MinGW一定要用64位的GLFW！**  
  
这篇自用的说明解释如何新建一个项目（不用生成器），在新项目中配置第三方库。全部在.json中完成。  
这也是vscode极简使用第三方库的通用方法。  
  
为了方便管理，把glad 中的 glad.c 源文件用gcc and ar 工具编译为了libglad.a静态库文件，方便管理，免得每次新建项目都要把glad.c放到项目源文件目录中。  
  
为了统一管理，我把第三方库放到了统一的extraLibs文件夹中，每一个子项都有 include 文件夹和 lib文件夹，前者放头文件，后者放库文件。  
  
openGL配置中有几个头文件：glad/glad.h，KHR文件夹下有glad相关的，我们不用直接调用；GLFW/glfw3.h GLFW/glfw3native.h 和GLFW有关，我们要调用前一个。  
lib文件夹中有编译好的libglad.a静态库（这是我们自己用glad.c手动生成的），还有几个glfw相关的。我们要在编译时链接两个：libglad.a 和 libglfw3dll.a。还有一个是运行时动态链接的：glfw3.dll。  
  
#### 处理头文件。  
要让vscode能够识别第三方头文件，一种方法是直接将它们复制到MinGW下的include文件夹中，将它们编程标准库函数，一劳永逸。但是不便管理，它们将淹没在真正的库函数中。我将他们放到了统一的另一个文件夹，这就要让vscode和gcc能够找到它们。  
首先在*c_cpp_properties.json* 的 *includePath* 中加上头文件所在的第三方include目录所在的路径。注意不要用任何/** 之类的符号。  
  
然后就能通过#include\<GLFW/glfw3.h>这样来调用，语法检查器能正常找到头文件，但是编译器gcc不知道！  
  
修改 *task.json* 中的参数 *args* ，修改传给编译器的命令行参数！紧跟着文件名后面加上：  
```  
"-I",			//让gcc预编译时读取下面这些文件  
"three-party include folder path",  //包含了所有直接间接调用的文件夹  
"header file1 complete path",	//也要绝对路径！  
"header fail2 complete path",  
...  
```  
然后处理链接问题，在 args 的最后加上：  
```  
"-L",      //链接选项  
"three-party lib folder path",		//指定库的路径  
"-lXXXX",		  // -l+库名，一定要删掉扩展名和前面的lib  
"-lXXXXX",		//比如链接 libglfw3dll.a 就用 -lglfw3dll  
...  
```  
**最后记得将要运行时动态链接的glfw3.dll放到生成的.exe文件所在的目录下！**  
成功运行的 *task.json* 的 args 如下：  
```  
"args": [  
                "-g",  
                "${file}",  
                "-I",  
                "C:\\Users\\13468\\.vscode\\extraLibs\\openGL\\include",  
                "C:\\Users\\13468\\.vscode\\extraLibs\\openGL\\include\\GLFW\\glfw3.h",  
                "C:\\Users\\13468\\.vscode\\extraLibs\\openGL\\include\\glad\\glad.h",  
                "-o",  
                "${fileDirname}\\exe\\${fileBasenameNoExtension}.exe",  
                "-L",  
                "C:\\Users\\13468\\.vscode\\extraLibs\\openGL\\lib",  
                "-lglad",  
                "-lglfw3dll",  
                "-lglfw3"  
            ],  
```  
