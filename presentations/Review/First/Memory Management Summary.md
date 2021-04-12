# Memory Management Summary

---

- From 0x0000 0000(starting postion) to 1MB, it stores the data of the boot loader, so the memory is reserved for that purpose and we cant't use that space.

- We use memory map, to find which all portions of the memory is free or used. In our kernal, Grub provides the memory map. Grub only knows where our OS starts and it doesnt know where it ends. So to find the end of our OS, we add kernel_end symbol to our linker script

- 1MB to kernal_end -  we cant use this region as it is reserved for our kernal data.

- To find and track the free regions we use stack and the regions are split into page frames like a block of memory i.e 4KB. For this purpose we use stack. 

- Here, after kernal_ends each memory is coverted into the page frames(like 4kb each), the  starting address of the 4kb is pushed to the stack. So if we need to access or use a memory we can pop it from the stack. Hence stack acts as a collection of free blocks of memory.  

-We need a granular memory allocator,hence we use liballoc. Liballoc- is a standard library used for memory allocation in our initial memory architecture. Liballoc uses page frame allocator(less than 4kb memory blocks). It has various functions like malloc, realloc, calloc, etc.


