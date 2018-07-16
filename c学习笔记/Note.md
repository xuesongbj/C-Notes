# Enum

枚举是一个被命名的整形常数的集合,枚举在日常生活中很常见。例如,表示星期的Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday就是一个枚举。

```c
enum week {
  Sunday,
  Monday,
  Tuesday,
  Wednesday,
  Thursday,
  Friday,
  Saturday
};
```

输出:

```c
Sunday = 0
Monday = 1
Tuesday = 2
Wednesday = 3
Thursday = 4
Friday = 5
Saturday = 6
```

### Enum成员的值可以相同

```c
enum clor {
  black = 1,
  yellow = 2,
  red = 1
}
 
int main(int argc, char **argv)
{
  enum color x = black;
  enum color y = red;
   
  printf("%d, %d\n", x, y);        // stdout: 1, 1
   
  return 0;
}
```



### Enum 和 define 区别

enum和define两种方式都可以定义一个常量,使用Enum和#define 有什么区别吗？使用#define关键字定义的常量，会在预编译阶段就将其展开,没有内存地址;而enum 关键字创建的常量会被编译器解释成整数型常量。它存在内存地址。



#### define 定义常量

```c
#define Num 10

int main(int argc, char **argv)
{
  printf("%d\n", Num);
  
  return 0;
}
```

#### define预处理结果

```c
$ gcc -E define.c

#define Num 10


int main(int argc, char **argv)
{

	printf("%d\n", Num);

	return 0;
}
```

通过以上实例可以看出,预处理之后的Num直接在main函数被展开。



### enum 定义常量

```c
#include <stdio.h>
#include <stdlib.h>

enum color {
	red,
	blue,
	black = 9,
	yellow = 9
};

int main(int argc, char **argv)
{
	enum color x = yellow;
	enum color y = black;

	printf("%d, %d\n", x, y);

	return 0;
}
```

#### enum预处理结果

```c
#include <stdio.h>
#include <stdlib.h>

enum color {
	red,
	blue,
	black = 9,
	yellow = 9
};

int main(int argc, char **argv)
{
	enum color x = yellow;
	enum color y = black;
  
	printf("%d, %d\n", x, y);

	return 0;
}
```

可以看到enum预处理之后x和y常量并没有被展开,这样x和y肯定存在内存地址。



#### 查看enum变量内存位置

```c
(gdb) info locals
x = black
y = black             // 虽然源码中x=yellow, y=black.但编译之后.发现x,y都指向了black.
 
(gdb) info proc mappings
process 4519
Mapped address spaces:

          Start Addr           End Addr       Size     Offset objfile
            0x400000           0x401000     0x1000        0x0 /home/zhangshaozhi/c/enum
            0x600000           0x601000     0x1000        0x0 /home/zhangshaozhi/c/enum
            0x601000           0x602000     0x1000     0x1000 /home/zhangshaozhi/c/enum
      0x7ffff7a0e000     0x7ffff7bd1000   0x1c3000        0x0 /usr/lib64/libc-2.17.so
      0x7ffff7bd1000     0x7ffff7dd0000   0x1ff000   0x1c3000 /usr/lib64/libc-2.17.so
      0x7ffff7dd0000     0x7ffff7dd4000     0x4000   0x1c2000 /usr/lib64/libc-2.17.so
      0x7ffff7dd4000     0x7ffff7dd6000     0x2000   0x1c6000 /usr/lib64/libc-2.17.so
      0x7ffff7dd6000     0x7ffff7ddb000     0x5000        0x0
      0x7ffff7ddb000     0x7ffff7dfd000    0x22000        0x0 /usr/lib64/ld-2.17.so
      0x7ffff7fef000     0x7ffff7ff2000     0x3000        0x0
      0x7ffff7ff8000     0x7ffff7ffa000     0x2000        0x0
      0x7ffff7ffa000     0x7ffff7ffc000     0x2000        0x0 [vdso]
      0x7ffff7ffc000     0x7ffff7ffd000     0x1000    0x21000 /usr/lib64/ld-2.17.so
      0x7ffff7ffd000     0x7ffff7ffe000     0x1000    0x22000 /usr/lib64/ld-2.17.so
      0x7ffff7ffe000     0x7ffff7fff000     0x1000        0x0
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0 [stack]
  0xffffffffff600000 0xffffffffff601000     0x1000        0x0 [vsyscall]
   
            
(gdb) p/x &x
$4 = 0x7fffffffe27c
(gdb) p/x &y
$5 = 0x7fffffffe278
(gdb) x/2w 0x7fffffffe278
0x7fffffffe278:	0x00000009	0x00000009            
```

通过gdb调试,发现x和y变量地址被映射在.stack栈帧内,它们大为4byte(和int类型相同)。
