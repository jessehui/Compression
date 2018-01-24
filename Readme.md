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

进入 `deflate` 函数后, `s->status = BUSY_STATE;` 此时进入真正进行 deflate 运算的步骤. 


```c
	/* Start a new block or continue the current one.
     */
    if (strm->avail_in != 0 || s->lookahead != 0 ||
        (flush != Z_NO_FLUSH && s->status != FINISH_STATE)) {
        block_state bstate;

        bstate = s->level == 0 ? deflate_stored(s, flush) :
                 s->strategy == Z_HUFFMAN_ONLY ? deflate_huff(s, flush) :
                 s->strategy == Z_RLE ? deflate_rle(s, flush) :
                 (*(configuration_table[s->level].func))(s, flush);

        if (bstate == finish_started || bstate == finish_done) {
            s->status = FINISH_STATE;
        }
        if (bstate == need_more || bstate == finish_started) {
            if (strm->avail_out == 0) {
                s->last_flush = -1; /* avoid BUF_ERROR next call, see above */
            }
            return Z_OK;
            /* If flush != Z_NO_FLUSH && avail_out == 0, the next call
             * of deflate should use the same flush parameter to make sure
             * that the flush is complete. So we don't have to output an
             * empty block here, this will be done at next call. This also
             * ensures that for a very small output buffer, we emit at most
             * one empty block.
             */
        }
        if (bstate == block_done) {
            if (flush == Z_PARTIAL_FLUSH) {
                _tr_align(s);
            } else if (flush != Z_BLOCK) { /* FULL_FLUSH or SYNC_FLUSH */
                _tr_stored_block(s, (char*)0, 0L, 0);
                /* For a full flush, this empty block will be recognized
                 * as a special marker by inflate_sync().
                 */
                if (flush == Z_FULL_FLUSH) {
                    CLEAR_HASH(s);             /* forget history */
                    if (s->lookahead == 0) {
                        s->strstart = 0;
                        s->block_start = 0L;
                        s->insert = 0;
                    }
                }
            }
            flush_pending(strm);
            if (strm->avail_out == 0) {
              s->last_flush = -1; /* avoid BUF_ERROR at next call, see above */
              return Z_OK;
            }
        }
    }
```

关于 `deflate_stored()` 函数:  
Copy without compression as much as possible from the input stream, return the current block state. 
`deflate_stored()` is written to minimize the number of times an input byte is copied. It is most efficient with large input and output buffers, which maximizes the opportunites to have a single copy from `next_in` to `next_out` .
对应的是 compression level 是 0. Means No Compression. 

`level` 不等于 0 的情况, 调用对应 `strategy` 的 deflate 函数, `deflate_huff` 或者 `deflate_rle`.   
最终调用 `FLUSH_BLOCK` 宏, 进行压缩处理. 

8. wrap

```c
typedef struct internal_state {
    z_streamp strm;
    int status;
    ...
    int wrap;    /* bit 0 true for zlib, bit 1 true for gzip */
    ...
} FAR deflate_state;
```

9. flush type

```c
#define Z_NO_FLUSH      0
#define Z_PARTIAL_FLUSH 1
#define Z_SYNC_FLUSH    2
#define Z_FULL_FLUSH    3
#define Z_FINISH        4
#define Z_BLOCK         5
#define Z_TREES         6
/* Allowed flush values; */
```
Normally the parameter flush is set to `Z_NO_FLUSH`, which allows deflate to decide how much data to accumulate before producing output, in order to maximize compression.
 
If the parameter flush is set to `Z_SYNC_FLUSH`, all pending output is flushed to the output buffer and the output is aligned on a byte boundary, so that the decompressor can get all input data available so far.
Flushing may degrade compression for some compression algorithms and so it should be used only when necessary.  This completes the current deflate block and follows it with an empty stored block that is three bits plus filler bits to the next byte, followed by four bytes (00 00 ff ff).

If flush is set to `Z_PARTIAL_FLUSH`, all pending output is flushed to the output buffer, but the output is not aligned to a byte boundary.  All of the input data so far will be available to the decompressor, as for `Z_SYNC_FLUSH`.  
This completes the current deflate block and follows it with an empty fixed codes block that is 10 bits long.  This assures that enough bytes are output in order for the decompressor to finish the block before the empty fixed codes block.

If flush is set to `Z_BLOCK`, a deflate block is completed and emitted, as for `Z_SYNC_FLUSH`, but the output is not aligned on a byte boundary, and up to seven bits of the current block are held to be written as the next byte after the next deflate block is completed.  In this case, the decompressor may not be provided enough bits at this point in order to complete decompression of the data provided so far to the compressor.  It may need to wait for the next block to be emitted.  This is for advanced applications that need to control the emission of deflate blocks.

If flush is set to `Z_FULL_FLUSH`, all output is flushed as with `Z_SYNC_FLUSH`, and the compression state is reset so that decompression can
restart from this point if previous compressed data has been damaged or if random access is desired.  Using `Z_FULL_FLUSH` too often can seriously degrade
compression.

If deflate returns with `avail_out == 0`, this function must be called again with the same value of the flush parameter and more output space (updated `avail_out`), until the flush is complete (deflate returns with non-zero `avail_out`).  In the case of a `Z_FULL_FLUSH` or `Z_SYNC_FLUSH`, make sure that avail_out is greater than six to avoid repeated flush markers due to `avail_out == 0` on return.
