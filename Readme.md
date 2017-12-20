# Compression

### 1. 关于 gzip, zlib 和 zip
1. gzip是UNIX下的一种数据格式. gzip是在zlib之上，包了一层，在头和尾添加了一些额外的信息. 
gzip是一种文件压缩工具（或该压缩工具产生的压缩文件格式），它的设计目标是处理单个的文件。gzip在压缩文件中的数据时使用的就是zlib。为了保存与文件属性有关的信息，gzip需要在压缩文件（.gz）中保存更多的头信息内容，而zlib不用考虑这一点。但gzip只适用于单个文件，所以我们在UNIX/Linux上经常看到的压缩包后缀都是.tar.gz或.tgz，也就是先用tar把多个文件打包成单个文件，再用gzip压缩的结果。  
2. zlib 是一个开源库, 提供了在内存中压缩和解压的函数. zlib 的设计目标是处理单纯的数据. 
3. zip 只是一种数据结构，跟rar同级别的。
zip是适用于压缩多个文件的格式（相应的工具有PkZip和WinZip等），因此，zip文件还要
进一步包含文件目录结构的信息，比gzip的头信息更多。但需要注意，zip格式可采用多种
压缩算法，我们常见的zip文件大多不是用zlib的算法压缩的，其压缩数据的格式与gzip大
不一样。

gzip, zlib以及图形格式png，使用的压缩算法都是deflate算法。

gzip 对于要压缩的文件，首先使用LZ77算法的一个变种进行压缩，对得到的结果再使用Huffman编码的方法进行压缩.

### 2. zstd
Zstandard was designed to give compression comparable to that of DEFLATE (ZIP, gzip) with higher compression / decompression speeds. Zstandard combines use of a dictionary-type algorithm (LZ77) and Finite State Entropy (tANS) stage of entropy coding.

zstd 中使用的不是 Huffman Coding. 而是 FSE. FSE 之前, 要提到算术编码 (Arithmetic Coding). 

There was still a problem with Huffman encoding (and all previous ones) : an hidden assumption is that a symbol must be encoded using an integer number of bits. To say it simply, you can't go lower than 1 bit.
It seems reasonable, but that's not even close to Shannon's limit. An event which has 90% probability to happen for example should be encoded using 0.15 bits. You can't do that using Huffman trees.

A solution to this problem was found almost 30 years later, by Jorma Rissanen, under the name of Arithmetic coder.

Zstd is developed for is configurable memory requirement, with the objective to fit into low-memory configurations, or servers handling many connections in parallel.


### 3. fflush
fflush has no effect on an unbuffered stream.
Buffers are normally maintained by the operating system, which determines the optimal time to write the data automatically to disk: when a buffer is full, when a stream is closed, or when a program terminates normally without closing the stream. 

### 4. assert
assert 宏的原型定义在 assert.h 中，其作用是如果它的条件返回错误，则终止程序执行. 

```c
#include "assert.h" 
void assert( int expression );
```
assert的作用是先计算表达式 expression ，如果其值为假（即为0），那么它先向stderr打印一条出错信息,然后通过调用 abort 来终止程序运行。  
使用assert的缺点是，频繁的调用会极大的影响程序的性能，增加额外的开销。    
在调试结束后，可以通过在包含#include 的语句之前插入 #define NDEBUG 来禁用assert调用，

### 5. zlib 中 OF
```c
#ifndef OF /* function prototypes */
#  ifdef STDC
#    define OF(args)  args
#  else
#    define OF(args)  ()
#  endif
#endif
```

### 6. zlib z_stream_s struct
```c
typedef struct z_stream_s {
    z_const Bytef *next_in;     /* next input byte */

    uInt     avail_in;  /* number of bytes available at next_in */
    uLong    total_in;  /* total number of input bytes read so far */

    Bytef    *next_out; /* next output byte will go here */
    uInt     avail_out; /* remaining free space at next_out */
    uLong    total_out; /* total number of bytes output so far */

    z_const char *msg;  /* last error message, NULL if no error */
    struct internal_state FAR *state; /* not visible by applications */

    alloc_func zalloc;  /* used to allocate the internal state */
    free_func  zfree;   /* used to free the internal state */
    voidpf     opaque;  /* private data object passed to zalloc and zfree */

    int     data_type;  /* best guess about the data type: binary or text
                           for deflate, or the decoding state for inflate */
    uLong   adler;      /* Adler-32 or CRC-32 value of the uncompressed data */
    uLong   reserved;   /* reserved for future use */
} z_stream;
```


### 7. zlib deflate function
```c
ZEXTERN int ZEXPORT deflate OF((z_streamp strm, int flush));
```

deflate compresses as much data as possible, and stops when the input buffer becomes empty or the output buffer becomes full. It may introduce some output latency (reading input without producing any output) except when forced to flush.

deflate performs one or both of the following actions:

- Compress more input starting at `next_in` and update `next_in` and `avail_in` accordingly. If not all input can be processed (because there is not enough room in the output buffer), `next_in` and `avail_in` are updated and processing will resume at this point for the next call of deflate().
- Generate more output starting at `next_out` and update `next_out` and `avail_out` accordingly. This action is forced if the parameter flush is non zero. Forcing flush frequently degrades the compression ratio, so this parameter should be set only when necessary. Some output may be provided even if flush is zero.


