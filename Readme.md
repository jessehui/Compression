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