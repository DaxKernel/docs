# Docs for Memory Management

### How do we know where the kernel ends?

>- The best way to determine this is to get the boot loader to call the BIOS (on the PC) and determine it that way. 
>- It is possible to read the CMOS NVRAM for the amount of memory installed (whilst remaining in protected mode), but that will only tell us about the first 64MB. 
>- So we need to call BIOS interrupt 15h to do it properly; without running in VM86 mode, the best way to do this is in the boot loader.

### How do we know where the free regions of memory are

>- The (physical) addresses of free pages are pushed onto the stack; when a page is allocated,the next address is popped off the top of the stack and used. 
>- When a page is freed, its address is pushed back onto the top of the stack. With this, an allocation (or deallocation) becomes a matter of incrementing (or decrementing) some pointer.


### Why do we need to use grub mmap?

> - GRUB provides a convenient way of detecting the amount of RAM our machine has.
> - When GRUB runs, it loads this structure into memory and leaves the address of this structure in the EBX register.
>- The key for memory detection lies in the multiboot_info struct. To determine the contiguous memory size, you may simply check mbd->flags to verify bit 0 is set, and then you can safely refer to mbd->mem_lower for conventional memory (e.g. physical addresses ranging between 0 and 640KB) and mbd->mem_upper for high memory (e.g. from 1MB).
>- GRUB simply uses INT 15h, EAX=E820 to get the detailed memory map

### Difference between stack vs bitmap for keeping track of free pages.
 
>- If we want to perform many contiguous allocations or allocations from a known location, we choose a bitmap. 
>- While using virtual memory and it really doesn't matter where the physical page frames come from, we use a stack.
>- In bitmap we initilizae the bitmap array to 0 then as we get the unusable addresses we mark them to 1, but with stack we have to collect them first and then work backwards.
>- One advantage of the bitmap is its space efficiency, and its simplicity.


