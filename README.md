# LZAV - Fast Data Compression Algorithm (in C) #

## Introduction ##

LZAV is a fast general-purpose in-memory data compression algorithm based on
now-classic [LZ77](https://wikipedia.org/wiki/LZ77_and_LZ78) lossless data
compression method. LZAV holds a good position on the Pareto landscape of
factors, among many similar in-memory (non-streaming) compression algorithms.

LZAV algorithm's code is portable, cross-platform, scalar, header-only,
inlineable C (C++ compatible). It supports little- and big-endian platforms,
and any memory alignment models. The algorithm is efficient on both 32- and
64-bit platforms. Incompressible data expands by no more than 0.58%.

LZAV does not sacrifice internal out-of-bounds (OOB) checks for decompression
speed. This means that LZAV can be used in strict conditions where OOB memory
writes (and especially reads) that lead to a trap, are unacceptable (e.g.,
real-time, system, server software). LZAV can be safely used to decompress
malformed or damaged compressed data. Which means that LZAV does not require
calculation of a checksum (or hash) of the compressed data. Only a checksum
of uncompressed data may be required, depending on application's guarantees.

The internal functions available in the `lzav.h` file allow you to easily
implement, and experiment with, your own compression algorithms. LZAV stream
format and decompressor have a potential of very high decompression speeds,
which depends on the way data is compressed.

## Usage Information ##

To compress data:

```c
#include "lzav.h"

int max_len = lzav_compress_bound( src_len );
void* comp_buf = malloc( max_len ); // Or similar.
int comp_len = lzav_compress_default( src_buf, comp_buf, src_len, max_len );

if( comp_len == 0 && src_len != 0 )
{
    // Error handling.
}
```

To decompress data:

```c
#include "lzav.h"

void* decomp_buf = malloc( src_len ); // Or similar.
int l = lzav_decompress( comp_buf, decomp_buf, comp_len, src_len );

if( l < 0 )
{
    // Error handling.
}
```

To compress data with a higher ratio, for non-time-critical applications
(e.g., compression of application's static assets):

```c
int comp_len = lzav_compress_hi( src_buf, comp_buf, src_len, max_len );
```

LZAV algorithm and its source code (which is
[ISO C99](https://en.wikipedia.org/wiki/C99)) were quality-tested on:
Clang, GCC, MSVC, Intel C++ compilers; x86, x86-64 (Intel, AMD), AArch64
(Apple Silicon) architectures; Windows 10, CentOS 8 Linux, macOS 14.1.

## Comparisons ##

The tables below present performance ballpark numbers of LZAV algorithm
(based on Silesia dataset).

While LZ4 there seems to be compressing faster, LZAV comparably provides 13.2%
memory storage cost savings. This is a significant benefit in database and
file system use cases since compression is only about 30% slower while CPUs
rarely run at their maximum capacity anyway. In general, LZAV holds a very
strong position in this class of data compression algorithms, if one considers
all factors: compression and decompression speeds, compression ratio, and not
less important - code maintainability: LZAV is maximally portable and has a
rather small independent codebase.

Performance of LZAV is not limited to the presented ballpark numbers.
Depending on the data being compressed, LZAV can achieve 800 MB/s compression
and 4500 MB/s decompression speeds. Incompressible data decompresses at 10000
MB/s rate, which is not far from the "memcpy". There are cases like the
[enwik9 dataset](https://mattmahoney.net/dc/textdata.html) where LZAV
provides 20% higher memory storage savings compared to LZ4. However, on small
data (below 50 KB), compression ratio difference between LZAV and LZ4
diminishes, and LZ4 may have some advantage.

LZAV algorithm's geomean performance on a variety of datasets is 530 +/- 150
MB/s compression and 3500 +/- 1200 MB/s decompression speeds, on 4+ GHz 64-bit
processors released after 2019. Note that the algorithm exhibits adaptive
qualities, and its actual performance depends on the data being compressed.
LZAV may show an exceptional performance on your specific data.

It is also worth noting that compression methods like LZAV and LZ4 usually
have an advantage over dictionary- and entropy-based coding in that
hash-table-based compression has a very small overhead while classic LZ77
decompression has none at all - this is especially relevant for smaller data.

For a more comprehensive in-memory compression algorithms benchmark you may
visit [lzbench](https://github.com/inikep/lzbench).

### Apple clang 15.0.0 arm64, macOS 14.1, Apple M1, 3.5 GHz ###

Silesia compression corpus

|Compressor      |Compression    |Decompression  |Ratio          |
|----            |----           |----           |----           |
|**LZAV 3.4**    |565 MB/s       |3020 MB/s      |41.31          |
|LZ4 1.9.4       |700 MB/s       |4570 MB/s      |47.60          |
|Snappy 1.1.10   |495 MB/s       |3230 MB/s      |48.22          |
|LZF 3.6         |395 MB/s       |800 MB/s       |48.15          |
|**LZAV 3.4 HI** |107 MB/s       |2990 MB/s      |36.08          |
|LZ4HC 1.9.4 -9  |40 MB/s        |4360 MB/s      |36.75          |

### LLVM clang-cl 16.0.4 x86-64, Windows 10, Ryzen 3700X (Zen2), 4.2 GHz ###

Silesia compression corpus

|Compressor      |Compression    |Decompression  |Ratio          |
|----            |----           |----           |----           |
|**LZAV 3.4**    |505 MB/s       |2720 MB/s      |41.31          |
|LZ4 1.9.4       |680 MB/s       |4300 MB/s      |47.60          |
|Snappy 1.1.10   |425 MB/s       |2430 MB/s      |48.22          |
|LZF 3.6         |320 MB/s       |700 MB/s       |48.15          |
|**LZAV 3.4 HI** |86 MB/s        |2670 MB/s      |36.08          |
|LZ4HC 1.9.4 -9  |36 MB/s        |4100 MB/s      |36.75          |

### LLVM clang 12.0.1 x86-64, CentOS 8, Xeon E-2176G (CoffeeLake), 4.5 GHz ###

Silesia compression corpus

|Compressor      |Compression    |Decompression  |Ratio          |
|----            |----           |----           |----           |
|**LZAV 3.4**    |465 MB/s       |2390 MB/s      |41.31          |
|LZ4 1.9.4       |660 MB/s       |4200 MB/s      |47.60          |
|Snappy 1.1.10   |545 MB/s       |2150 MB/s      |48.22          |
|LZF 3.6         |370 MB/s       |880 MB/s       |48.15          |
|**LZAV 3.4 HI** |74 MB/s        |2370 MB/s      |36.08          |
|LZ4HC 1.9.4 -9  |32 MB/s        |4150 MB/s      |36.75          |

P.S. Popular Zstd's benchmark was not included here, because it is not a pure
LZ77, much harder to integrate, and has a much larger code size - a different
league, close to zlib. If you have a practical interest, performance of LZAV
can be compared to Zstd and other algorithms using precompiled
[TurboBench](https://github.com/powturbo/TurboBench/releases). Here are
author's measurements with TurboBench, on Ryzen 3700X, on Silesia dataset:

|Compressor      |Compression    |Decompression  |Ratio          |
|----            |----           |----           |----           |
|zstd 1.5.5 -1   |460 MB/s       |1870 MB/s      |41.0           |
|zstd 1.5.5 1    |436 MB/s       |1400 MB/s      |34.6           |

## Notes ##

1. LZAV API is not equivalent to LZ4 nor Snappy API. For example, the "dstl"
parameter in the decompressor should specify the original uncompressed length,
which should have been previously stored in some way, independent of LZAV.

2. Run-time memory sanitizers like Valgrind and Dr.Memory may generate the
"uninitialized read" warning in decompressor's block type 1 handler. This is
an expected behavior, and not a bug - this happens because of SIMD
optimizations that read bytes from the output buffer (within its valid range)
which were not yet initialized.

3. Compared to Clang, other compilers systematically produce 5% slower LZAV
code. Compiler architecture tuning (other than generic x86-64) may produce
varying, including counter-productive, results.

## Thanks ##

* [Paul Dreik](https://github.com/pauldreik), for finding memcpy UB in the
decompressor.
