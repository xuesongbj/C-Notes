# C标准库 - <stdarg.h>



## 简介

该库可用于在参数个数未知(即参数个数可变)时获取函数中的参数。



### C 函数调用栈

在说stdarg动态参数处理之前,先说一下正常情况下C的函数调用参数传递。正常情况下c函数调用参数传递采用__stdcall规则,它是从右到左进行参数入栈,即函数最右参数最先入栈。先看一个实例:

```c
#include <stdio.h>
#include <stdlib.h>
 
void test(int a, int b, int c)
{
  int d;
  d = a + b +c;
   
  printf("%d\n", d);
}
 
int main(int argc, char **argv)
{
  int a, b, c;
  a = 1;
  b = 2;
  c = 3;
  test(a, b, c);
   
  return 0;
}
```

编译和链接之后,可以对ELF文件反汇编,查看调用test函数时,是如何进行参数传递的。

```c
000000000040051d <test>:
  40051d:       55                      push   rbp
  40051e:       48 89 e5                mov    rbp,rsp
  400521:       48 83 ec 20             sub    rsp,0x20
  400525:       89 7d ec                mov    DWORD PTR [rbp-0x14],edi    ; a
  400528:       89 75 e8                mov    DWORD PTR [rbp-0x18],esi    ; b
  40052b:       89 55 e4                mov    DWORD PTR [rbp-0x1c],edx    ; c
  40052e:       8b 45 e8                mov    eax,DWORD PTR [rbp-0x18]
  400531:       8b 55 ec                mov    edx,DWORD PTR [rbp-0x14]
  400534:       01 c2                   add    edx,eax
  400536:       8b 45 e4                mov    eax,DWORD PTR [rbp-0x1c]
  400539:       01 d0                   add    eax,edx
  40053b:       89 45 fc                mov    DWORD PTR [rbp-0x4],eax
  40053e:       8b 45 fc                mov    eax,DWORD PTR [rbp-0x4]
  400541:       89 c6                   mov    esi,eax
  400543:       bf 30 06 40 00          mov    edi,0x400630
  400548:       b8 00 00 00 00          mov    eax,0x0
  40054d:       e8 ae fe ff ff          call   400400 <printf@plt>
  400552:       c9                      leave
  400553:       c3                      ret
```

C语言默认采用BP对栈帧进行寻址(Go采用SP对栈帧进行寻址)。通过以上汇编代码可以得知,调用test函数时参数使用寄存器进行传递。a参数处于栈帧空间rbp-0x14位置，b参数处于栈帧rbp-0x18，c参数处于栈帧rbp-0x1c。可以看出离SP最近的为c，其次为b，最后为a。此时，可以确定C函数参数传递采用__stdcall规则进行传递。

如果按这种规则,仅需要知道第一个参数地址,然后根据第二,第三...参数类型进行偏移量操作岂不是可以实现动态参数。虽然这种想法很好,C编译器具体是如何实现的,咱们可以通过一个实例来看看。

```c
#include <stdlib.h>
#include <stdio.h>
#include <stdarg.h>

void test(int a, ...) {
        printf("%p\n", &a);
}

int main(int argc, char **argv)
{
        int a, b, c;
        a = 1;
        b = 2;
        c = 3;
        test(a, b, c);

        return 0;
}
```

编译之后，通过gdb查看内存排序。

```c
(gdb) disassemble
Dump of assembler code for function test:
   0x000000000040051d <+0>:	push   rbp
   0x000000000040051e <+1>:	mov    rbp,rsp
   0x0000000000400521 <+4>:	sub    rsp,0xc0
   0x0000000000400528 <+11>:	mov    QWORD PTR [rbp-0xa8],rsi
   0x000000000040052f <+18>:	mov    QWORD PTR [rbp-0xa0],rdx
   0x0000000000400536 <+25>:	mov    QWORD PTR [rbp-0x98],rcx
   0x000000000040053d <+32>:	mov    QWORD PTR [rbp-0x90],r8
   0x0000000000400544 <+39>:	mov    QWORD PTR [rbp-0x88],r9
   0x000000000040054b <+46>:	test   al,al
   0x000000000040054d <+48>:	je     0x40056f <test+82>
   0x000000000040054f <+50>:	movaps XMMWORD PTR [rbp-0x80],xmm0
   0x0000000000400553 <+54>:	movaps XMMWORD PTR [rbp-0x70],xmm1
   0x0000000000400557 <+58>:	movaps XMMWORD PTR [rbp-0x60],xmm2
   0x000000000040055b <+62>:	movaps XMMWORD PTR [rbp-0x50],xmm3
   0x000000000040055f <+66>:	movaps XMMWORD PTR [rbp-0x40],xmm4
   0x0000000000400563 <+70>:	movaps XMMWORD PTR [rbp-0x30],xmm5
   0x0000000000400567 <+74>:	movaps XMMWORD PTR [rbp-0x20],xmm6
   0x000000000040056b <+78>:	movaps XMMWORD PTR [rbp-0x10],xmm7
   0x000000000040056f <+82>:	mov    DWORD PTR [rbp-0xb4],edi
=> 0x0000000000400575 <+88>:	lea    rax,[rbp-0xb4]
   0x000000000040057c <+95>:	mov    rsi,rax
   0x000000000040057f <+98>:	mov    edi,0x400680
   0x0000000000400584 <+103>:	mov    eax,0x0
   0x0000000000400589 <+108>:	call   0x400400 <printf@plt>
   0x000000000040058e <+113>:	lea    rax,[rbp-0xb4]
   0x0000000000400595 <+120>:	add    rax,0x10
   0x0000000000400599 <+124>:	mov    rsi,rax
   0x000000000040059c <+127>:	mov    edi,0x400680
   0x00000000004005a1 <+132>:	mov    eax,0x0
   0x00000000004005a6 <+137>:	call   0x400400 <printf@plt>
   0x00000000004005ab <+142>:	leave
   0x00000000004005ac <+143>:	ret
End of assembler dump.
(gdb) p/x &a
$1 = 0x7fffffffe12c
(gdb) x/10xw &a
0x7fffffffe12c:	0x00000001	0xffffe160	0x00007fff	0x00000002
0x7fffffffe13c:	0x00000000	0x00000003	0x00000000	0x00000002
0x7fffffffe14c:	0x00000000	0xf7dd5e80
```

通过查看a地址后的内存排序,发现跟调用test(a, b, c)时内存的顺序并不是一致。所以通过上面的猜想使用a地址＋类型偏移量是拿不到第二个参数值。那如何在c语言下进行动态参数传递哪？



### stdarg

c标准库提供了stdarg库，它可以实现动态参数传递。stdarg库定义了个变量类型va_list和3个宏，这三个宏可用于在参数个数未知(即参数个数可变)时获取函数中的参数。



#### 库变量

| 变量      | 描述                                         |
| ------- | ------------------------------------------ |
| va_list | 适用于va_start()、va_arg()和va_end()这三个宏存储信息的类型 |

#### 宏

| 宏                                   | 描述                                                                  |
|:-----------------------------------:| ------------------------------------------------------------------- |
| void va_start(va_list ap, last_arg) | 初始化ap变量,它与va_arg和va_end宏一起使用的。last_arg是最后一个传递给函数的已知的固定参数,即省略号之前的参数。 |
| type va_arg(va_list ap, type)       | 这个宏检索函数列表中类型为type的下一个参数。                                            |
| void va_end(va_list ap)             | 这个宏允许使用了va_start宏带有可变参数的函数返回。如果在从函数返回之前没有调用va_end，则结果为为定义。          |



### stdarg 实例

```c
#include <stdlib.h>
#include <stdio.h>
#include <stdarg.h>

void test(int a, ...) {
	va_list ap;
	va_start(ap, a);

	printf("%d\n", va_arg(ap, int));
	printf("%d\n", va_arg(ap, int));

	va_end(ap);
}

int main(int argc, char **argv)
{
	int a, b, c;
	a = 1;
	b = 2;
	c = 3;
	test(a, b, c);

	return 0;
}
```

编译查看一下返回结果:

```c
$ gcc -g3 -std=c11 -Wall -Wpedantic -o test test.c && ./test
2
3
```

接下来,再通过反汇编看一下stdarg是如何实现的。

```c
000000000040051d <test>:
  40051d:       55                      push   rbp
  40051e:       48 89 e5                mov    rbp,rsp
  400521:       48 81 ec e0 00 00 00    sub    rsp,0xe0
  400528:       48 89 b5 58 ff ff ff    mov    QWORD PTR [rbp-0xa8],rsi
  40052f:       48 89 95 60 ff ff ff    mov    QWORD PTR [rbp-0xa0],rdx
  400536:       48 89 8d 68 ff ff ff    mov    QWORD PTR [rbp-0x98],rcx
  40053d:       4c 89 85 70 ff ff ff    mov    QWORD PTR [rbp-0x90],r8
  400544:       4c 89 8d 78 ff ff ff    mov    QWORD PTR [rbp-0x88],r9
  40054b:       84 c0                   test   al,al
  40054d:       74 20                   je     40056f <test+0x52>
  40054f:       0f 29 45 80             movaps XMMWORD PTR [rbp-0x80],xmm0
  400553:       0f 29 4d 90             movaps XMMWORD PTR [rbp-0x70],xmm1
  400557:       0f 29 55 a0             movaps XMMWORD PTR [rbp-0x60],xmm2
  40055b:       0f 29 5d b0             movaps XMMWORD PTR [rbp-0x50],xmm3
  40055f:       0f 29 65 c0             movaps XMMWORD PTR [rbp-0x40],xmm4
  400563:       0f 29 6d d0             movaps XMMWORD PTR [rbp-0x30],xmm5
  400567:       0f 29 75 e0             movaps XMMWORD PTR [rbp-0x20],xmm6
  40056b:       0f 29 7d f0             movaps XMMWORD PTR [rbp-0x10],xmm7
  40056f:       89 bd 2c ff ff ff       mov    DWORD PTR [rbp-0xd4],edi
  400575:       c7 85 38 ff ff ff 08    mov    DWORD PTR [rbp-0xc8],0x8
  40057c:       00 00 00
  40057f:       c7 85 3c ff ff ff 30    mov    DWORD PTR [rbp-0xc4],0x30
  400586:       00 00 00
  400589:       48 8d 45 10             lea    rax,[rbp+0x10]
  40058d:       48 89 85 40 ff ff ff    mov    QWORD PTR [rbp-0xc0],rax
  400594:       48 8d 85 50 ff ff ff    lea    rax,[rbp-0xb0]
  40059b:       48 89 85 48 ff ff ff    mov    QWORD PTR [rbp-0xb8],rax
  4005a2:       8b 85 38 ff ff ff       mov    eax,DWORD PTR [rbp-0xc8]
  4005a8:       83 f8 30                cmp    eax,0x30
  4005ab:       73 23                   jae    4005d0 <test+0xb3>
  4005ad:       48 8b 95 48 ff ff ff    mov    rdx,QWORD PTR [rbp-0xb8]
  4005b4:       8b 85 38 ff ff ff       mov    eax,DWORD PTR [rbp-0xc8]
  4005ba:       89 c0                   mov    eax,eax
  4005bc:       48 01 d0                add    rax,rdx
  4005bf:       8b 95 38 ff ff ff       mov    edx,DWORD PTR [rbp-0xc8]
  4005c5:       83 c2 08                add    edx,0x8
  4005c8:       89 95 38 ff ff ff       mov    DWORD PTR [rbp-0xc8],edx
  4005ce:       eb 15                   jmp    4005e5 <test+0xc8>
  4005d0:       48 8b 95 40 ff ff ff    mov    rdx,QWORD PTR [rbp-0xc0]
  4005d7:       48 89 d0                mov    rax,rdx
  4005da:       48 83 c2 08             add    rdx,0x8
  4005de:       48 89 95 40 ff ff ff    mov    QWORD PTR [rbp-0xc0],rdx
  4005e5:       8b 00                   mov    eax,DWORD PTR [rax]
  4005e7:       89 c6                   mov    esi,eax
  4005e9:       bf 30 07 40 00          mov    edi,0x400730
  4005ee:       b8 00 00 00 00          mov    eax,0x0
  4005f3:       e8 08 fe ff ff          call   400400 <printf@plt>
  4005f8:       8b 85 38 ff ff ff       mov    eax,DWORD PTR [rbp-0xc8]
  4005fe:       83 f8 30                cmp    eax,0x30
  400601:       73 23                   jae    400626 <test+0x109>
  400603:       48 8b 95 48 ff ff ff    mov    rdx,QWORD PTR [rbp-0xb8]
  40060a:       8b 85 38 ff ff ff       mov    eax,DWORD PTR [rbp-0xc8]
  400610:       89 c0                   mov    eax,eax
  400612:       48 01 d0                add    rax,rdx
  400615:       8b 95 38 ff ff ff       mov    edx,DWORD PTR [rbp-0xc8]
  40061b:       83 c2 08                add    edx,0x8
  40061e:       89 95 38 ff ff ff       mov    DWORD PTR [rbp-0xc8],edx
  400624:       eb 15                   jmp    40063b <test+0x11e>
  400626:       48 8b 95 40 ff ff ff    mov    rdx,QWORD PTR [rbp-0xc0]
  40062d:       48 89 d0                mov    rax,rdx
  400630:       48 83 c2 08             add    rdx,0x8
  400634:       48 89 95 40 ff ff ff    mov    QWORD PTR [rbp-0xc0],rdx
  40063b:       8b 00                   mov    eax,DWORD PTR [rax]
  40063d:       89 c6                   mov    esi,eax
  40063f:       bf 30 07 40 00          mov    edi,0x400730
  400644:       b8 00 00 00 00          mov    eax,0x0
  400649:       e8 b2 fd ff ff          call   400400 <printf@plt>
  40064e:       c9                      leave
  40064f:       c3                      ret
```



stdarg库具体实现说明有待补充...
