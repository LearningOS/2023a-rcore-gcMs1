# Lab1 Report
### Author: gcMs1

## 题目总结
本题目标在于引入一个新的系统调用sys_task_info来获取当前运行项目信息。 

实现方法如下：  
1. TaskControl中添加任务使用的系统调用及调用次数(syscall_times)、系统调用时刻距离任务第一次被调度时刻的时长(time)。
2. 在TaskManager实例化中加入对syscall_times、time的初始化内容。
3. 在TaskManger中添加get_current_start_time()、get_current_status()、get_current_syscall_times()以及add_syscall_times()方法，并利用pub将其设置为外部接口，便于外部调用获取当前任务相关信息，其中add_syscall_times()函数在syscall函数开始调用，每次进入syscall则syscall_times加1。
4. 在process.rs中将获取task_info各种函数封装到一个接口中，外部文件调用该接口即可获得当前运行任务信息。  
   
## 简答题  
**1.请描述运行三个Rust bad测例报错行为：**    
  
所使用sbi版本为v0.3.2  
1).ch2b_bad_address.rs:  
报错信息：  
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.  
该错误是由于由于访问了一个无效的内存地址（0x0）而导致的页错误，表明应用程序不应该访问内核地址空间中的内存。  
  
2).ch2b_bad_instructions.rs:  
报错信息：  
[kernel] IllegalInstruction in application, kernel killed it.  
该错误是由于在用户模式下执行sret指令，sret指令通常用于RISC-V架构中的内核态切换。但用户模式下执行sret指令通常会导致非法指令异常。  
  
3).ch2b_bad_register.rs:  
报错信息：  
[kernel] IllegalInstruction in application, kernel killed it.  
该错误是由于在用户模式下执行csrr指令而导致非法指令异常，该指令用于从特权寄存器中读取特权模式状态，是一个特权指令，只能在特权模式下执行。 
 
  
**2.深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用：**  
  
1).刚进入__restore 时，a0代表了什么值。请指出__restore的两种使用情景。  
a0代表指向TrapContext结构的指针。__restore两种使用场景为：保存上下文和调用trap_handler、恢复上下文和返回用户空间。  
  
2).L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。  
(1).sstutas寄存器：这里将sstatus寄存器的值从内核栈中加载到t0，以便在恢复用户态时，能够还原之前的处理器状态。  
(2).sepc寄存器：这里将sepc寄存器的值从内核栈中加载到t1，以便在恢复用户态时，处理器能够返回到引发异常的指令，从异常发生的地方继续执行。  
(3).sscratch寄存器：这里将sscratch寄存器的值从内核栈中加载到t2，以便在恢复用户态时，将用户栈的指针正确设置为sscratch寄存器的值。  
  
3).L50-L56：为何跳过了 x2 和 x4？  
因为在上下文切换的过程中只保存和恢复那些在用户态和内核态之间需要保持一致的通用寄存器，通常包括x5到x31寄存器，因为它们用于存储用户程序的状态和临时数据。而x2和x4的值在上下文切换过程中不需要改变，因此可以跳过它们的保存和恢复。  
  
4).L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？  
sp与sscratch的值交换之后，sscratch指向了用户态栈，从而允许处理器在用户态中执行;sp则指向了内核栈的指针。  
  
5).__restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？  
__restore中状态切换发生在sret指令，当该指令执行时，处理器从内核态切换回用户态。原因是sret是用于从内核态返回到用户态的特权指令。它会将处理器状态设置为用户态，包括将特权级别设置为用户态，将程序计数器（sepc）设置为用户态代码的地址，以便处理器能够从用户态代码中继续执行。  
  
6).L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？  
第13行sp与sscratch的值交换之后，sp指向了用户态栈，从而允许处理器在用户态中执行;sscratch指向了内核栈的指针，以便在将来需要从用户态切换回内核态时能够正确恢复内核栈的指针。  
  
7).从 U 态进入 S 态是哪一条指令发生的？  
是L13的"csrrw sp, sscratch, sp"，作用是将sp寄存器的值设置为sscratch寄存器的当前值，从而将栈指针切换到内核栈，从用户态切换到内核态。  
  
## 荣誉准则  
1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：  
**与同组李宝润同学讨论了关于syscall_times的计数问题。**
  
2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：  
**1).rCore-Tutorial-Guide-2023A文档  
  2).OS API docs of ch3**  
    
3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。  
     
4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。
