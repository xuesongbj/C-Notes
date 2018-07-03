# C标准库 - <stdio.h>



## <stdio.h> 作用

头文件<stdio.h> 声明了很多执行输入输出的函数,几乎所有的程序都要执行输出操作,所以这个头文件被广泛使用。它也是最早出现C标准库头文件之一。这个头文件比其他任何声明的函数都要多,同时实现这些函数机制很复杂，因此该库内容需要慢慢完善。



### 类型

<stdio.h> 声明了3种类型,其中包括size_t、FILE和fpos_t。



#### size_t

size_t 类型可以存储任何类型(包括数组)理论上可能的对象最大大小。size_t通常用于数组索引和循环计数。程序使用其它类型,如:unsigned int 用于数组索引可能失败。

 

#### FILE

它是一个对象类型,可以记录控制流需要的所有信息,包括它的文件定位符、指向相关缓冲的指针、记录是否发生了读/写的错误指示符和记录文件是否结束的文件结束符。



#### fpos_t

它是一个对象类型,可以唯一指定文件中的每一个未知位置所需的所有信息。

 



### 常用宏

| 宏                          | 描述                                                           |
| -------------------------- | ------------------------------------------------------------ |
| NULL                       | 宏是一个空指针常量的值                                                  |
| _IOFBF、_IOLBF和_IONBF       | 这些宏扩展了带有特定值的整型常量表达式，并适用于**setvbuf**函数的第三个参数                  |
| BUFSIZ                     | 这个宏是一个整数，该整数代表了**setbuf**函数使用的缓冲区大小                          |
| EOFM                       | 这个宏是一个表示已经到达文件结束的负整数                                         |
| FOPEN_MAX                  | 这个宏是一个整数，该整数代表了系统可以同时打开的文件数量                                 |
| FILENAME_MAX               | 这个宏是一个整数，该整数代表了字符数组可以存储的文件名的最大长度。如果实现没有任何限制，则该值应为推荐的最大值      |
| L_tmpnam                   | 这个宏是一个整数，该整数代表了字符数组可以存储的由 tmpnam 函数创建的临时文件名的最大长度             |
| SEEK_CUR、SEEK_END、SEEK_SET | 这些宏是在These macros are used in the fseek函数中使用，用于在一个文件中定位不同的位置 |
| TMP_MAX                    | 这个宏是 tmpnam 函数可生成的独特文件名的最大数量                                 |
| stderr、stdin和stdout        | 这些宏是指向 FILE 类型的指针，分别对应于标准错误、标准输入和标准输出流                       |



### 库函数

| 函数                                                                    | 描述                                                               |
| --------------------------------------------------------------------- | ---------------------------------------------------------------- |
| int fclose(FILE stream)                                               | 关闭流stream,刷新所有缓冲区                                                |
| FILE fopen(const char filename, const char mode)                      | 使用给定模式mode打开filename所指向文件                                        |
| size_t fread(void ptr, size_t filename, size_t nmemb, FILE stream)    | 从给定流stream读取数据到ptr所指向的数组                                         |
| int fseek(FILE stream, long int offset, int whence)                   | *设置流 stream 的文件位置为给定的偏移 offset，参数*offset*意味着从给定的*whence*位置查找的字节数 |
| size_t fwrite(const void ptr, size_t size, size_t nmemb, FILE stream) | 把 ptr 所指向的数组中的数据写入到给定流 stream 中                                  |

#### 文件读写实例

```c
#include <stdlib.h>
#include <stdio.h>

void write(char buffer[], int len)
{

	FILE *pf;
	pf = fopen("test.txt", "w+");
	fwrite(buffer, len, 1, pf);
	fclose(pf);
}

char* read(int len)
{
	char *buffer = malloc(sizeof(char) * len);
	FILE *pf = fopen("test.txt", "r");
	fread(buffer, len, 1, pf);

	fclose(pf);

	return buffer;
}

int main(int argc, char **argv)
{
  // Write content to test.txt file
	char buffer[] = {'a', 'b', 'c', '\n'};
	write(buffer, sizeof(buffer));

  // Read rows from test.txt file
	char *d = read(sizeof(buffer));
	for (int i = 0; i < sizeof(d); i++)
	{
		printf("%c", d[i]);
	}
  
  // Free sizeof(char) * len(sizeof(buffer)) byte memory
	free(d);

	return 0;
}
```



#### 内存泄漏检查

```c
==18512== Memcheck, a memory error detector
==18512== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==18512== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==18512== Command: ./test
==18512==
abc
==18512== Invalid read of size 1
==18512==    at 0x40080E: main (test.c:35)
==18512==  Address 0x52032c4 is 0 bytes after a block of size 4 alloc'd
==18512==    at 0x4C29C23: malloc (vg_replace_malloc.c:299)
==18512==    by 0x400772: read (test.c:16)
==18512==    by 0x4007F3: main (test.c:32)
==18512==
==18512==
==18512== HEAP SUMMARY:
==18512==     in use at exit: 0 bytes in 0 blocks
==18512==   total heap usage: 3 allocs, 3 frees, 1,140 bytes allocated
==18512==
==18512== All heap blocks were freed -- no leaks are possible
==18512==
==18512== For counts of detected and suppressed errors, rerun with: -v
==18512== ERROR SUMMARY: 4 errors from 1 contexts (suppressed: 0 from 0)
```



#### 源码剖析

##### FILE 数据结构

```c
/*
	include/stdio.h
*/

__BEGIN_NAMESPACE_STD
/* The opaque type of streams.  This is the definition used elsewhere.  */
typedef struct _IO_FILE FILE;
__END_NAMESPACE_STD
```

FILE 数据结构真实

```c

/*
	include/libio.h
*/

struct _IO_FILE {
  int _flags;           /* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;   /* Current read pointer */
  char* _IO_read_end;   /* End of get area. */
  char* _IO_read_base;  /* Start of putback+get area. */
  char* _IO_write_base; /* Start of put area. */
  char* _IO_write_ptr;  /* Current put pointer. */
  char* _IO_write_end;  /* End of put area. */
  char* _IO_buf_base;   /* Start of reserve area. */
  char* _IO_buf_end;    /* End of reserve area. */
  
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
#if 0
  int _blksize;
#else
  int _flags2;
#endif
  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

#define __HAVE_COLUMN /* temporary */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  /*  char* _save_gptr;  char* _save_egptr; */

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```



##### fopen 源码

```c
/*
include/stdio.h
*/
#  if !defined NOT_IN_libc
extern _IO_FILE *_IO_new_fopen (const char*, const char*);
#   define fopen(fname, mode) _IO_new_fopen (fname, mode)
```

fopen并不是一个函数，它是一个宏。后端具体实现的是_IO_new_fopen函数。

```c
/*
	glibc/libio/iofopen.c
*/
_IO_FILE *
_IO_new_fopen (filename, mode)
     const char *filename;                        // 文件名称
     const char *mode;                            // 打开文件模式。r,w,a,w+...
{
  return __fopen_internal (filename, mode, 1);    // 真正打开文件函数
}


_IO_FILE *
__fopen_internal (filename, mode, is32)
     const char *filename;
     const char *mode;
     int is32;
{
  // heap申请sizeof(struct locked_FILE) byte大小内存空间
  struct locked_FILE
  {
    struct _IO_FILE_plus fp;
    struct _IO_wide_data wd;
  } *new_f = (struct locked_FILE *) malloc (sizeof (struct locked_FILE));

  // 是否内存申请成功,否则退出
  if (new_f == NULL)
    return NULL;
	
  // 如果文件打开成功,则进行映射
  if (_IO_file_fopen ((_IO_FILE *) new_f, filename, mode, is32) != NULL)
    return __fopen_maybe_mmap (&new_f->fp.file);

  // 当打开文件失败
  // 将fp指向的FILE结构体从_IO_list_all的单向链表取下,并调用_IO_file_close_it(fp)关闭fp。
  // 将heap申请内存释放
  _IO_un_link (&new_f->fp);
  free (new_f);
  return NULL;
}
```

_IO_un_link 函数:

```c
/*
	glibc/libio/_IO_un_link
*/

void
_IO_un_link (fp)
     struct _IO_FILE_plus *fp;
{
  if (fp->file._flags & _IO_LINKED)
    {
      struct _IO_FILE **f;
#ifdef _IO_MTSAFE_IO
      _IO_cleanup_region_start_noarg (flush_cleanup);
      _IO_lock_lock (list_all_lock);
      run_fp = (_IO_FILE *) fp;
      _IO_flockfile ((_IO_FILE *) fp);
#endif
      if (_IO_list_all == NULL)
        ;
      else if (fp == _IO_list_all)
        {
          _IO_list_all = (struct _IO_FILE_plus *) _IO_list_all->file._chain;
          ++_IO_list_all_stamp;
        }
      else
        for (f = &_IO_list_all->file._chain; *f; f = &(*f)->_chain)
          if (*f == (_IO_FILE *) fp)
            {
              *f = fp->file._chain;
              ++_IO_list_all_stamp;
              break;
            }
      fp->file._flags &= ~_IO_LINKED;
#ifdef _IO_MTSAFE_IO
      _IO_funlockfile ((_IO_FILE *) fp);
      run_fp = NULL;
      _IO_lock_unlock (list_all_lock);
      _IO_cleanup_region_end (0);
#endif
    }
}
libc_hidden_def (_IO_un_link)
```

__fopen_maybe_mmap函数:

```c
_IO_FILE *
__fopen_maybe_mmap (fp)
     _IO_FILE *fp;
{
#ifdef _G_HAVE_MMAP
  if ((fp->_flags2 & _IO_FLAGS2_MMAP) && (fp->_flags & _IO_NO_WRITES))
    {
      /* Since this is read-only, we might be able to mmap the contents
         directly.  We delay the decision until the first read attempt by
         giving it a jump table containing functions that choose mmap or
         vanilla file operations and reset the jump table accordingly.  */

      if (fp->_mode <= 0)
        _IO_JUMPS ((struct _IO_FILE_plus *) fp) = &_IO_file_jumps_maybe_mmap;
      else
        _IO_JUMPS ((struct _IO_FILE_plus *) fp) = &_IO_wfile_jumps_maybe_mmap;
      fp->_wide_data->_wide_vtable = &_IO_wfile_jumps_maybe_mmap;
    }
#endif
  return fp;

```
