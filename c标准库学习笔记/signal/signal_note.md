# C 标准库 － <signal.h>



## 简介

signal.h文件定义了一个变量类型 sig_atomic_t、两个函数调用和一些宏来处理程序执行期间报告不同的信号。



## sig_atomic_t

这是**int**类型，在信号处理程序中作为变量使用。它是一个对象的整数类型，该对象可以作为一个原子实体访问，即使存在异步信号时，该对象可以作为一个原子实体访问。



## Fake signal functions

SIG_ 宏与 signal 函数一起使用来定义信号的功能。




## 常用Signal

| 宏       | int值 | 描述                  |
| ------- | ---- | ------------------- |
| SIGINT  | 2    | 中断信号,如Ctrl-c        |
| SIGSEGV | 11   | 非法访问存储器,如访问不存在的内存单元 |
| SIGTERM | 15   | 向本程序发送终止请求信号        |
| SIGILL  | 4    | 非法指令(非法函数映射)        |
| SIGFPE  | 8    | 算出运算出错,如除数为0或溢出     |
| SIGABRT | 6    | 程序异常终止              |
| SIGQUIT | 3    | 程序退出                |
| SIGKILL | 9    | 强制杀死进程              |

* 更多signal可以看以下源码

  ```c
  /* ISO C99 signals.  */
  #define        SIGINT                2        /* Interactive attention signal.  */
  #define        SIGILL                4        /* Illegal instruction.  */
  #define        SIGABRT                6        /* Abnormal termination.  */
  #define        SIGFPE                8        /* Erroneous arithmetic operation.  */
  #define        SIGSEGV                11        /* Invalid access to storage.  */
  #define        SIGTERM                15        /* Termination request.  */
  /* Historical signals specified by POSIX. */
  #define        SIGHUP                1        /* Hangup.  */
  #define        SIGQUIT                3        /* Quit.  */
  #define        SIGTRAP                5        /* Trace/breakpoint trap.  */
  #define        SIGKILL                9        /* Killed.  */
  #define SIGBUS                10        /* Bus error.  */
  #define        SIGSYS                12        /* Bad system call.  */
  #define        SIGPIPE                13        /* Broken pipe.  */
  #define        SIGALRM                14        /* Alarm clock.  */
  /* New(er) POSIX signals (1003.1-2008, 1003.1-2013).  */
  #define        SIGURG                16        /* Urgent data is available at a socket.  */
  #define        SIGSTOP                17        /* Stop, unblockable.  */
  #define        SIGTSTP                18        /* Keyboard stop.  */
  #define        SIGCONT                19        /* Continue.  */
  #define        SIGCHLD                20        /* Child terminated or stopped.  */
  #define        SIGTTIN                21        /* Background read from control terminal.  */
  #define        SIGTTOU                22        /* Background write to control terminal.  */
  #define        SIGPOLL                23        /* Pollable event occurred (System V).  */
  #define        SIGXCPU                24        /* CPU time limit exceeded.  */
  #define        SIGXFSZ                25        /* File size limit exceeded.  */
  #define        SIGVTALRM        26        /* Virtual timer expired.  */
  #define        SIGPROF                27        /* Profiling timer expired.  */
  #define        SIGUSR1                30        /* User-defined signal 1.  */
  #define        SIGUSR2                31        /* User-defined signal 2.  */
  /* Nonstandard signals found in all modern POSIX systems
     (including both BSD and Linux).  */
  #define        SIGWINCH        28        /* Window size change (4.3 BSD, Sun).  */
  /* Archaic names for compatibility.  */
  #define        SIGIO                SIGPOLL        /* I/O now possible (4.2 BSD).  */
  #define        SIGIOT                SIGABRT        /* IOT instruction, abort() on a PDP-11.  */
  #define        SIGCLD                SIGCHLD        /* Old System V name */
  /* Not all systems support real-time signals.  bits/signum.h indicates
     that they are supported by overriding __SIGRTMAX to a value greater
     than __SIGRTMIN.  These constants give the kernel-level hard limits,
     but some real-time signals may be used internally by glibc.  Do not
     use these constants in application code; use SIGRTMIN and SIGRTMAX
     (defined in signal.h) instead.  */
  #define __SIGRTMIN        32
  #define __SIGRTMAX        __SIGRTMIN
  ```



## Function

| 函数                                            | 描述                         |
| --------------------------------------------- | -------------------------- |
| void (signal(int sig, void (func)(int)))(int) | 该函数设置一个函数来处理信号,即信号处理程序     |
| int raise(int sig)                            | 该函数会促使生成信号sig。sig参数与SIG宏兼容 |



## 实例1

```c
# volatile 指出 i 是随时可能发生变化的，每次使用它的时候必须从 i的地址中读取，因而编译器生成的汇编代码会重新从i的地址读取数据放在 b 中.

volatile int i = 10;
         int b = i;
```



## 实例2

```c
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>

volatile sig_atomic_t gSignalStatus;

void signal_handler(int signal)
{
	gSignalStatus = signal;
}

int main(int argc, char **argv)
{
	signal(SIGINT, signal_handler);

	printf("SignalValue: %d\n", gSignalStatus);
	printf("Sending signal: %d\n", SIGINT);
	raise(SIGINT);
	printf("SignalValue: %d\n", gSignalStatus);

	return 0;
}
```

* 输出

  ```c
  $ gcc -g3 -std=c11 -Wall -Wpedantic -o test test.c && ./test 
  SignalValue: 0
  Sending signal: 2
  SignalValue: 2
  ```


