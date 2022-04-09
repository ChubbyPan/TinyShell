# TinyShell
csapp tsh lab

TODO
- review tsh lab
- modify readme
   

## 这是一个什么项目?
用基本库实现了一个简易的shell程序,进行一系列读命令,解析命令,执行命令的过程.

## 实现了什么功能?
- 对命令进行解析
- 除了常用命令之外,支持一些built-in command
  - quit 退出当前进程
  - bg %PID 前台进程转换成后台进程
  - fg %PID 后台进程转换成前台进程
  - jobs 列出当前进程状态(running/suspend/foreground)
- 并发执行多个进程.
- catch signals, 更改进程状态.

## 具体内容
### **parseline():**
解析命令,形成参数列表, 返回是否为后台进程的标记位.
### **builtin_cmd():**
执行内置命令(quit,fg,bg,jobs).
### **eval():**
![eval_function](./eval_fuction)
### **waitfg():**
等待前台进程执行结束
### **handler()**
包括sigchld_handler():子进程结束之后向父进程发送sigchld信号,处理程序将回收已完成的子进程; sigint_handler():当键盘按下ctrl-c时,发送terminal信号,中止子进程; sigtstp_handler():当键盘按下ctrl-z时,发送suspend信号,挂起子进程.


## 遇到了什么问题,怎么解决?
### 信号同步问题
- 问题描述: 后台进程在执行完毕之后，工作列表里面也没有把已完成的进程删除。
- 解决方案: 当子进程结束之后会向父进程发送sigchld信号，但是由于block和unblock的设置不恰当，导致进程结束后发送的信号并没有捕捉到，没有执行对应的处理程序。从后台进程开始执行，每当阻塞和解阻塞时，都打印结果，先定位是哪部分出现问题，和预期结果是否一致。根据log进行修改。
### 进程卡死
- 问题描述: 子进程执行外部程序，外部进程包含进程终止的信号，此时前台进程卡死。
- 解决方案: 在sigchld_handler中，会一直调用一个waitpid函数，这个函数会等待子进程终止，并捕捉子进程发送的sigchld信号，并返回子进程因为什么终止的信息（通过status返回）；但是由于最开始没有对返回信息进行处理，导致只能处理子进程内部的终止或者暂停信号；所以继续对status进行处理即可。

## 还有哪些不足
不支持管道和重定向
