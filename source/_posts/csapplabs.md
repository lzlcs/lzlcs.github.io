---
title: CSAPP Labs 小结
date: 2023-09-04 12:00:23
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# `data`

**`bitXor`**

本质上是使用 非门 `~` 和 与门 `&` 实现 异或门 `^`

$$
\begin{align*}
a \ \oplus \ b &= (a \overline b) + (\overline a b)  \\
&= \overline{ \overline{ a \overline b} \cdot \overline { \overline a b}}


\end{align*}
$$

```c
int bitXor(int x, int y) {
    return ~(~(~x & y) & ~(~y & x));
}
```

**`tmin`**

使用二进制补码, 最高位为 `1` 时得到 `tmin`

```c
int tmin(void) {
    return 1 << 31;
}
```

# `malloc`

**隐式空闲链表**

根据 `head` 和 `foot` 中记录的长度可以找到左右块, 从而组成双向链表

使用策略: 首次适配, 立即合并

``` c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

team_t team = {
    /* Team name */
    "ateam",
    /* First member's full name */
    "lzl",
    /* First member's email address */
    "emmmm",
    /* Second member's full name (leave blank if none) */
    "",
    /* Second member's email address (leave blank if none) */
    ""
};

/* single word (4) or double word (8) alignment */
#define ALIGNMENT 8

/* rounds up to the nearest multiple of ALIGNMENT */
#define ALIGN(size) (((size) + (ALIGNMENT-1)) & ~0x7)


#define SIZE_T_SIZE (ALIGN(sizeof(size_t)))

#define MAX(a, b) ((a) > (b) ? a : b)

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1 << 12)

#define PACK(size, alloc) ((size) | (alloc))

#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE)))

static char *heap_listp;

static void *coalesce(void *bp) {
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));

    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && next_alloc) return bp;
    else if (prev_alloc && !next_alloc) {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

   } else if (!prev_alloc && next_alloc) {

        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);

    } else {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));

        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    return bp;
}

static void *extend_heap(size_t words) {
    char *bp;
    size_t size;


    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;
    
    bp = (char *)mem_sbrk(size);

    if ((long)bp == -1) return NULL;

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

    return coalesce(bp);
}


static void *find_fit(size_t asize) {

    void *cur_ptr = heap_listp;
    while (GET_SIZE(HDRP(cur_ptr)) != 0) {
        if (GET_SIZE(HDRP(cur_ptr)) >= asize &&
            !GET_ALLOC(HDRP(cur_ptr)))
                return cur_ptr;
        cur_ptr = NEXT_BLKP(cur_ptr);
    }
   
    return NULL;
}

static void place(void *bp, size_t asize) {

    size_t rest_size = GET_SIZE(HDRP(bp)) - asize;
    char *cur_bp = (char *)bp;

    if (rest_size >= 2 * DSIZE) {

        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        bp = NEXT_BLKP(bp);

        PUT(HDRP(bp), PACK(rest_size, 0));
        PUT(FTRP(bp), PACK(rest_size, 0));

    } else {
    
        asize = GET_SIZE(HDRP(bp));
        PUT(HDRP(cur_bp), PACK(asize, 1));
        PUT(FTRP(cur_bp), PACK(asize, 1));
    }
}
/* 
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
    mem_init();
    if ((heap_listp = (char *)mem_sbrk(4 * WSIZE)) == (void *)-1) return -1;

    PUT(heap_listp, 0);
    PUT(heap_listp + (1 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (2 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (3 * WSIZE), PACK(0, 1));
    heap_listp += (2 * WSIZE);

    return 0;
}

/* 
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size) {

    size_t asize;
    size_t extendsize;
    char *bp;

    if (size == 0) return NULL;

    if (size <= DSIZE) asize = 2 * DSIZE;
    else asize = DSIZE * ((size + (DSIZE) + (DSIZE - 1)) / DSIZE);

    bp = (char *)find_fit(asize);

    if (bp != NULL) {
        place(bp, asize);
        return bp;
    }

    extendsize = asize;

    bp = (char *)extend_heap(extendsize / WSIZE);

    if (bp == NULL) return NULL;

    place(bp, asize);

    return bp;
}

/*
 * mm_free - Freeing a block does nothing.
 */
void mm_free(void *ptr)
{
    if (ptr == NULL) return;

    size_t size = GET_SIZE(HDRP(ptr));

    PUT(HDRP(ptr), PACK(size, 0));
    PUT(FTRP(ptr), PACK(size, 0));

    coalesce(ptr);
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
    if (size == 0) {
        mm_free(ptr);
        return NULL;
    }

    if (ptr == NULL) {
        return mm_malloc(size);
    }

    void *newptr = mm_malloc(size);
    if (!newptr) return NULL;
    
    size_t oldsize = GET_SIZE(HDRP(ptr));

    if (size < oldsize) oldsize = size;

    memcpy(newptr, ptr, oldsize);
    mm_free(ptr);
    return newptr;

}
```
得分 `46 (util) + 36 (thru) = 83/100`

**显式空闲链表**

空闲块中 `head` 之后的两个位置放指针, 指向上一个 / 下一个空闲块

```c
/*
 * mm-naive.c - The fastest, least memory-efficient malloc package.
 * 
 * In this naive approach, a block is allocated by simply incrementing
 * the brk pointer.  A block is pure payload. There are no headers or
 * footers.  Blocks are never coalesced or reused. Realloc is
 * implemented directly using mm_malloc and mm_free.
 *
 * NOTE TO STUDENTS: Replace this header comment with your own header
 * comment that gives a high level description of your solution.
 */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

/*********************************************************
 * NOTE TO STUDENTS: Before you do anything else, please
 * provide your team information in the following struct.
 ********************************************************/
team_t team = { /* Team name */
    "ateam",
    /* First member's full name */
    "lzl",
    /* First member's email address */
    "emmmm",
    /* Second member's full name (leave blank if none) */
    "",
    /* Second member's email address (leave blank if none) */
    ""
};

/* single word (4) or double word (8) alignment */
#define ALIGNMENT 8

/* rounds up to the nearest multiple of ALIGNMENT */
#define ALIGN(size) (((size) + (ALIGNMENT-1)) & ~0x7)


#define SIZE_T_SIZE (ALIGN(sizeof(size_t)))

#define MAX(a, b) ((a) > (b) ? a : b)

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1 << 6)

#define PACK(size, alloc) ((size) | (alloc))

#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE)))

#define GET_PREV(p) (*(unsigned int *)(p))
#define SET_PREV(p, prev) (*(unsigned int *)(p) = (prev))
#define GET_NEXT(p) (*((unsigned int *)(p)+1))
#define SET_NEXT(p, val) (*((unsigned int *)(p)+1) = (val))

static char *heap_listp;
static char *free_listp;

void remove_from_free_list(void *bp) {
    if (bp == NULL || GET_ALLOC(bp)) return;

    void *prev = GET_PREV(bp);
    void *next = GET_NEXT(bp);

    SET_PREV(bp, 0), SET_NEXT(bp, 0);

    if (prev == NULL && next == NULL) {
        free_listp = NULL;
    } else if (prev == NULL) {
        SET_PREV(next, 0);
        free_listp = next;
    } else if (next == NULL) {
        SET_NEXT(prev, 0);
    } else {
        SET_NEXT(prev, next);
        SET_PREV(next, prev);
    }

}

void insert_to_free_list(void *bp) {
    if (bp == NULL) return;

    if (free_listp == NULL) 
        return free_listp = bp, (void)0;

    SET_NEXT(bp, free_listp);
    SET_PREV(free_listp, bp);

    free_listp = bp;
}


static void *coalesce(void *bp) {
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));

    size_t size = GET_SIZE(HDRP(bp));

    void *prev_bp = PREV_BLKP(bp);
    void *next_bp = NEXT_BLKP(bp);

    if (prev_alloc && next_alloc) {

    }
    else if (prev_alloc && !next_alloc) {

        remove_from_free_list(next_bp);

        size += GET_SIZE(HDRP(next_bp));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

   } else if (!prev_alloc && next_alloc) {

        remove_from_free_list(prev_bp);

        size += GET_SIZE(HDRP(prev_bp));

        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(prev_bp), PACK(size, 0));

        bp = PREV_BLKP(bp);

    } else {

        remove_from_free_list(prev_bp);
        remove_from_free_list(next_bp);

        size += GET_SIZE(HDRP(next_bp));
        size += GET_SIZE(HDRP(prev_bp));

        PUT(FTRP(next_bp), PACK(size, 0));
        PUT(HDRP(prev_bp), PACK(size, 0));
        bp = PREV_BLKP(bp);

    }
    insert_to_free_list(bp);
    return bp;
}

static void *extend_heap(size_t words) {
    char *bp;
    size_t size;

    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;
    
    bp = (char *)mem_sbrk(size);

    if ((long)bp == -1) return NULL;

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

    SET_PREV(bp, 0), SET_NEXT(bp, 0);

    return coalesce(bp);
}

static void* find_fit(size_t asize) {

    for (void* bp = free_listp; bp != 0; bp = GET_NEXT(bp))
        if (GET_SIZE(HDRP(bp)) >= asize)
            return bp;

    return NULL;
}

static void place(void *bp, size_t asize) {

    size_t rest_size = GET_SIZE(HDRP(bp)) - asize;
    char *cur_bp = (char *)bp;

    remove_from_free_list(bp);

    if (rest_size >= 2 * DSIZE) {

        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        bp = NEXT_BLKP(bp);

        PUT(HDRP(bp), PACK(rest_size, 0));
        PUT(FTRP(bp), PACK(rest_size, 0));

        SET_PREV(bp, 0), SET_NEXT(bp, 0);
        coalesce(bp);

    } else {
    
        asize = GET_SIZE(HDRP(bp));
        PUT(HDRP(cur_bp), PACK(asize, 1));
        PUT(FTRP(cur_bp), PACK(asize, 1));
    }
}
/* 
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
    mem_init();
    if ((heap_listp = (char *)mem_sbrk(4 * WSIZE)) == (void *)-1) return -1;

    PUT(heap_listp, 0);
    PUT(heap_listp + (1 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (2 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (3 * WSIZE), PACK(0, 1));
    heap_listp += (2 * WSIZE);
    free_listp = NULL;

    return 0;
}

/* 
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size) {

    size_t asize;
    size_t extendsize;
    char *bp;

    if (size == 0) return NULL;

    if (size <= DSIZE) asize = 2 * DSIZE;
    else asize = DSIZE * ((size + (DSIZE) + (DSIZE - 1)) / DSIZE);

    bp = (char *)find_fit(asize);

    if (bp != NULL) {
        place(bp, asize);
        return bp;
    }

    extendsize = asize;

    bp = (char *)extend_heap(extendsize / WSIZE);

    if (bp == NULL) return NULL;

    place(bp, asize);


    return bp;
}

/*
 * mm_free - Freeing a block does nothing.
 */
void mm_free(void *ptr)
{
    if (ptr == NULL) return;

    size_t size = GET_SIZE(HDRP(ptr));

    PUT(HDRP(ptr), PACK(size, 0));
    PUT(FTRP(ptr), PACK(size, 0));

    SET_PREV(ptr, 0), SET_NEXT(ptr, 0);

    coalesce(ptr);
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
    if (size == 0) {
        mm_free(ptr);
        return NULL;
    }

    if (ptr == NULL) {
        return mm_malloc(size);
    }

    void *newptr = mm_malloc(size);
    if (!newptr) return NULL;
    
    size_t oldsize = GET_SIZE(HDRP(ptr));

    if (size < oldsize) oldsize = size;

    memcpy(newptr, ptr, oldsize);
    mm_free(ptr);
    return newptr;

}

void checkheap(int verbose);

static void printblock(void *bp)
{
  size_t hsize, halloc, fsize, falloc;

  checkheap(0);
  hsize = GET_SIZE(HDRP(bp));
  halloc = GET_ALLOC(HDRP(bp));
  fsize = GET_SIZE(FTRP(bp));
  falloc = GET_ALLOC(FTRP(bp));

  if (hsize == 0) {
    printf("%p: EOL\n", bp);
    return;
  }

  printf("%p: header: [%ld:%c] footer: [%ld:%c]\n", bp,
      hsize, (halloc ? 'a' : 'f'),
      fsize, (falloc ? 'a' : 'f'));
}

static void checkblock(void *bp)
{
  if ((size_t)bp % 8)
    printf("Error: %p is not doubleword aligned\n", bp);
  if (GET(HDRP(bp)) != GET(FTRP(bp)))
    printf("Error: header does not match footer\n");
}

/*
 * checkheap - Minimal check of the heap for consistency
 */
void checkheap(int verbose)
{
  char *bp = heap_listp;

  if (verbose)
    printf("Heap (%p):\n", heap_listp);

  if ((GET_SIZE(HDRP(heap_listp)) != DSIZE) || !GET_ALLOC(HDRP(heap_listp)))
    printf("Bad prologue header\n");
  checkblock(heap_listp);

  for (bp = heap_listp; GET_SIZE(HDRP(bp)) > 0; bp = NEXT_BLKP(bp)) {
    if (verbose)
      printblock(bp);
    checkblock(bp);
  }

  if (verbose)
    printblock(bp);
  if ((GET_SIZE(HDRP(bp)) != 0) || !(GET_ALLOC(HDRP(bp))))
    printf("Bad epilogue header\n");
}
```

`45 (util) + 40 (thru) = 85/100`

优化的点主要在速度上

