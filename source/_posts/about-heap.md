---
layout: post
title: About heap
tags:
  - pwn
categories:
  - exploit
date: 2017-04-20 23:48:31
---
本文主要介绍ptmalloc3中关键的数据结构以及 `malloc`、`free`函数的执行过程。
#### malloc_state
```
struct malloc_state {
    /* Serialize access.  */
    mutex_t mutex;

    /* Flags (formerly in max_fast). */
    int flags;

    #if THREAD_STATS
        /* Statistics for locking. Only used if THREAD_STATS is defined. */
        long stat_lock_direct, stat_lock_loop, stat_lock_wait;
    #endif

    /* Fastbins */
    mfastbinptr fastbinsY[NFASTBINS];

    /* Base of the topmost chunk -- not otherwise kept in a bin */
    mchunkptr        top;

    /* The remainder from the most recent split of a small request */
    mchunkptr last_remainder;

    /* Normal bins packed as described above */
    mchunkptr        bins[NBINS * 2 - 2];

    /* Bitmap of bins */
    unsigned int binmap[BINMAPSIZE];

    /* Linked list */
    struct malloc_state *next;

    #ifdef PER_THREAD
        /* Linked list for free arenas. */
        struct malloc_state *next_free;
    #endif

    /* Memory allocated from the system in this arena. */
    INTERNAL_SIZE_T system_mem;
    INTERNAL_SIZE_T max_system_mem;
};
```
<!-- more -->
#### heap_info
```
typedef struct _heap_info
{
  mstate ar_ptr; /* Arena for this heap. */
  struct _heap_info *prev; /* Previous heap. */
  size_t size;   /* Current size in bytes. */
  size_t mprotect_size; /* Size in bytes that has been mprotected
   PROT_READ|PROT_WRITE.  */
  /* Make sure the following data is properly aligned,
   particularly that sizeof (heap_info) + 2 * SIZE_SZ is
   a multiple of MALLOC_ALIGNMENT. */
  char pad[-6 * SIZE_SZ & MALLOC_ALIGN_MASK];
} heap_info;
```
#### fastbin

*   缓存small bin前7个
*   单链表，LIFO 栈
*   32bit:
    *   范围 0x10-0x40 8字节依次递增  共7个
*   64bit:
    *   范围 0x20-0x80 16字节依次递增 共7个
*   精确匹配
*   libc中的全局变量`global_max_fast`定义了fast bins中chunk的最大值，`get_max_fast()`函数用于获取该值

#### bins

*   共128，第0个和第127个不用，第1个为unsorted bin，接下来62个为small bin，后面63个为large bin
*   每个bin使用双向循环链表管理空闲chunk，bin的链表头的指针fb指向第一个可用的chunk，指针bk指向最后一个可用的chunk，分别对应宏first(b)和last(b)
*   small bins 共62个
    *   双向链表，FIFO
    *   精确匹配
    *   32bit
        *   范围 16-504B (<0x200B)
    *   64bit
        *   范围 32-1008B(< 0x400B)
*   large bins共63个
    *   分成6组，每组数量依次为32、16、8、4、2、1
    *   32bit
        *   从0x200B开始，公差依次为0x40B、0x200B、0x1000B、3276B、262144B
    *   64bit
        *   从0x400B开始，公差依次为0x40B、0x200B、0x1000B、0x8000B
    *   范围匹配
    *   每个bin中的chunk按照从大到小排列，同时一个chunk存在于两个双向链表中，一个链表包含了large bin中所有的chunk，另一个链表为chunk size链表，该链表从每个相同大小的chunk取出第一个chunk按照大小顺序链接在一起，便于一次跨域多个相同大小的chunk遍历下一个不同大小的chunk
*   unsorted bin，只有一个，位于bins表的第一个位置

#### top chunk
*   只有一个，位于malloc_state结构中

#### last_remainder

*   一个，分配区上次分配small chunk时，从一个chunk中分裂出一个small chunk返回给用户，分裂后的剩余部分形成一个chunk，last_remainder就指向这个chunk
*   每个bin使用双向循环链表管理空闲chunk，bin的链表头的指针fb指向第一个可用的chunk，指针bk指向最后一个可用的chunk，分别对应宏first(b)和last(b)

#### main_arena和 thread arena
![about_heap.png](https://i.loli.net/2018/05/12/5af6bda1e588a.png)

#### malloc

1.  首先检测请求大小是否小于 `global_max_fast`时，如果满足且对应的fastbin非空，采用LIFO，移除对应的fastbin指向的free chunk （存在大小检查），返回；不满足，继续下一步
2.  检查请求大小是否满足`in_smallbin_range`（小于MIN\_LARGE\_SIZE），如果对应的small bin非空，移除bk指向的free chunk，即双向链表中的最后一个free chunk，设置下一个chunk的inuse标志位，返回；不满足，继续下一步
3.  反向遍历unsorted bin表中的free chunk
    1.  如果需要分配small bin chunk，且unsorted bin中只有一个chunk，并且这个chunk为last\_remainder\_chunk且这个chunk的大小大于所需的chunk大小加上MINSIZE，如下所示：
    ```
        if (in_smallbin_range(nb) &&
            bck == unsorted_chunks(av) &&
            victim == av->last_remainder &&
            (unsigned long)(size) >
            (unsigned long)(nb + MINSIZE)) {
    ```
    2.  不满足以上条件，从unsorted bin中移除当前chunk，如果该chunk等于所需的chunk的大小，则返回
    3.  将当前chunk加到对应的small bin或者large bin表中
    4.  如果分配的chunk为large bin chunk，遍历large bin表，找到合适的chunk
    5.  如果通过上面的方式从最合适的 small bin 或 large bin 中都没有分配到需要的 chunk，则 查看比当前bin的index大的small bin或large bin是否有空闲chunk可利用来分配所需的 chunk
    6.  尝试从top chunk中分配所需chunk

#### free

1.  首先检查大小是否小于 `get_max_fast()`，如果小于且检查下一个相邻堆块通过，则将当前堆块插到对应的fast bin表表头，返回；否则进入下一步
2.  检查当前块的前一个堆块是否空闲，如果空闲，则将它从bin表中删除（unlink），计算合并后的大小
3.  检查与当前块相邻的下一个chunk是不是top chunk，如果不是，检测是否处于inuse状态，若空闲，unlink
4.  将合并后的chunk加入unsorted bin 双向链表中

#### 参考：

*   https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/
