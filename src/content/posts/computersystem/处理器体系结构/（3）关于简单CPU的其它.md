---
title: 处理器原理(3) 关于简单CPU的其它
published: 2022-08-15
description: 《深入理解计算机系统》(CS:APP)的笔记, 关于CPU的简单内容. 处理器体系结构的内容都记录地非常简略...
tags: [计算机系统, 处理器体系结构]
category: 计算机系统
draft: false
---

:::warning
这是《深入理解计算机系统》(CS:APP)的笔记，彼时还未上CO,OS等计算机系统基础课，存在理解不到位与错误之处.  
处理器体系结构的内容都记录地非常简略...
:::

# 性能  
一般用指令没周期（IPC）或者平均周期每指令（CPI）。  
Y86中，正常情况下，指令吞吐量时一周期一指令。但流水线特殊情况使它有一定的截停。  
计算平均性能，一般忽略异常（很少），只考虑出现暂停、气泡的各种状态、组合的出现概率。  
一般提升性能的瓶颈是更高正确率的分支预测策略。  
# 多周期指令  
比如乘法、除法、浮点数加减乘除等。  
它们有特殊的硬件实现，一般处于*主流水线*之外的部分，而它们也被流水线化了。当主流水线遇到这些指令，就讲它们发射到它们专用的硬件。  
当然指令必须同步，所以涉及更多更复杂的各种转发与截停，等等。  
# 与存储系统的接口  
操作系统通过虚拟内存管理存储，而硬件需要将虚拟内存翻译为物理内存（有翻译后备缓冲器快速翻译）。  
处理器体系是一层一层的，程序先从高速缓存器（cache）中找数据、地址，如果有，一般能在一周期完成读写。如果高速缓存不命中，就要在内存中找，这一般要3-20周期，CPU中的流水一般会停下来等它。但若目标在磁盘中，则读写需要上百万个时钟周期，不可能等。处理器产生一个*缺页* 异常，将当前程序运行至今的状态“挂起”。触发操作系统的缺页异常处理例程，该例程发起一个从磁盘到主存的传送。一旦完成，操作系统就返回原本正在执行的程序（那个被“挂起”的），继续执行。  
# 当前的微处理器  
简单如CS:APP的Y86（当然这太简单了）的微处理器在各种嵌入式系统上仍大有用处。  
现在许多处理器实现了*超标量* 操作，可以并行地执行多条指令的取指、译码等等。最先进的处理器采用*乱序执行技术*，对指令的执行顺序可能完全不同于它们在程序中出现的顺序，但保留了ISA顺序模型的整体行为，很神奇。  
