## Final Goal
---
Our final objective is to implement memory management.  
* This means implementing the **malloc** function in the C standard library.
* The malloc function takes the number of bytes to be allocated as argument and returns a pointer to beginning of the allocated memory.
* If the request fails, it returns NULL.
* In addition to malloc, we also support other memory functions such as **calloc** and **realloc**.  
calloc is the same as malloc except that it zero-initializes the memory locations.  
realloc is used to change the size of an already allocated block of memory.

Now we will see what we need to do to accomplish this final goal.

## What does the memory look like?
---

The first thing that we need to do is to visualize how the memory is laid out.  
This diagram shows a detailed view of the physical memory in our OS.  
Since our OS is 32-bit and the hardware is byte-addressable, each memory address has 32 bits.   
Therefore the first memory address is composed of 32 zeros and the last memory address is composed of 32 ones.


When we boot our OS, we ask GRUB to place our kernel at the one megabyte mark. Below the one megabyte mark, we have:
* BIOS Data Area
* Extended BIOS Data Area
* Bootloader code

This section is shown in reddish-brown color.
These are important for the system, so we must take care to ensure that we **do not** write to memory below the 1 MB mark.

Our kernel is stored in memory from 1MB onwards. Next thing we want to know is where our kernel ends in physical memory.  
We can find this by placing a symbol at the end of our linker script. Then in C, we can take the address of that symbol to
determine where our kernel ends. Therefore, we can say that, the region between 1MB and the kernel_end symbol is completely 
occupied by our kernel.

Next we need a memory map that tells us which regions of the memory are free. With this information, we can decide on a strategy to
keep track of the free regions of memory.  
The free regions of memory are indicated in violet color. Some amount of this free memory
needs to be used to implement a data structure that keeps track of the free regions. This is indicated by yellow color in this diagram.

## How do we know which regions are free?
---

As mentioned earlier we use a memory map for this purpose.  
There are two primary methods we can use to get the memory map:  

**1. We can use a BIOS function**  
This involves storing the command 0xE820 into the EAX register and then raising a software interrupt using the assembly instruction
`INT 0x15`.  
There are two problems with using this BIOS function:
* There may be overlapping regions.
* Some of the entries may be unlisted.
* However the biggest problem with this approach is that we need to be in real mode to call bios functions. We would much rather
prefer to be in 32-bit protected mode.

**2. We can ask GRUB to give us the mmap**  
We mentioned in the previous presentation that we use GRUB with multiboot spec to write our OS.  
In addition to providing an easy way for a bootloader to detect and boot the kernel, multiboot also provides a rich 
variety of useful information that we can use in our OS.  
One of the more important information that it provides is the memory map.

## Using GRUB to detect memory
---

1. The first thing to do is to get GRUB to pass the multiboot info to our kernel.  
Once GRUB boots our OS, it leaves the multiboot information in the EAX and EBX registers. So we need to push both these registers into the stack.  
We can easily do this using the `pushl` assembly instruction.
Next we correspondingly modify our kernel_main function to accept these two arguments as a multiboot_info pointer and a constant called magic.  

2. Next we verify that multiboot is working by ensuring that the magic constant is equal to a predefined value.

3. We also need to verify that multiboot did actually load the memory map.  
We can do this by examining the flags field of the multiboot_info i.e  
if bit 6 of this field is set, we can be sure that 
multiboot did load the memory map.

4. Now we can simply iterate through the mmap using the code shown here. The memory address where the memory map is stored is given 
by multiboot_info->mmap_addr.  
We can get the next element of the memory map by performing simple pointer arithmetic i.e we simply 
add the size of the structure to the memory address.

## Parsing the memory map
---

Each element of the memory map is a struct as indicated by the code shown here.
* **size** field gives the size of this struct without accounting for itself  
i.e it is sizeof(this struct) - sizeof(this struct.size)  
* **base_addr_low** and **base_addr_high** gives the address to start of this memory region.  
* **length_low** and **length_high** gives the number of bytes in this memory region.
* **type** tells us the nature of this memory region. If type is 1, this memory region can be considered to be free.  
We don't want to use magic constant numbers in our code so we use a macro to define 1 as `MEMORY_AVAILABLE`.   
We also use another macro to indicate whether this region of memory overlaps with our kernel.

## Physical Frame Allocator
---
* Our objective here is to create a mechanism that allows us to allocate memory in large blocks - about 4KB here -  
and correspondingly deallocate it. This is what the malloc function will use internally.
* All we need to do really is simply keep track of the available memory and handle requests for memory allocation and deallocation.
* To keep track of free memory we have two main choices:

    * We can use a bitmap. In this approach every bit in the bitmap corresponds to a single 4KB chunk of free memory.  
    The primary advantage here is space efficiency i.e we only need one bit for each block.  
    However when allocating or deallocating memory we will need to search the entire bitmap to find a free block.  
    This search operation takes O(n) time.

    * Another possibility is to use a stack. This approach is very simple because all you need to do is push each memory address into the stack.  
    Now when you need a free page frame you simply do a pop operation on the stack.  
    In other words we can serve requests in O(1) time.

* We use the stack approach in our OS.

## Implementation (of PFA)
---
* First we need to calculate how much memory is needed for the stack that we use to keep track of memory.  
We can derive this mathematically. We'll look into that in the next slide.

* The next step is to allocate this stack at the end of the kernel.

* Next we need to mark any overlapping we have with kernel.  
When GRUB creates the memory map, it does not know that we have a kernel in memory.  
Therefore some of the regions that GRUB may mark as free may actually be used by our kernel.  
So if a particular region of the memory is overlapping with our kernel we need to
mark it with the macro MEMORY_OVERLAP so that we do not accidentally push those regions into the stack.

* Now all thats left to do is to push all the free memory addresses that do not overlap with our kernel into the stack.  
We'll see the code to do this.

## Deriving the amount of stack space needed
---

If T is the number of bytes of free memory and m is the number of page frames,  
the sum of the number of page frames multiplied by the size of each page frame and the space needed for stack should be equal to T.
From this, we can make m the subject.  
Clearly, the final equation becomes `m=4100/T`.  
Since we know T, we can easily find m.

## Pushing to the stack
---

What we're doing here is:

1. If a region of memory is overlapping with kernel+stack space, we reduce its length by the number of bytes that  
are overlapping and we set its base address to the end of the stack.

2. Then we push memory addresses within this memory region that are separated by 4096 bytes into the stack.

## Porting liballoc
---
So far we have implemented a physical page allocator i.e we now have a mechanism to allocate memory in blocks of 4kb each.
However, usually we only want to allocate a small amount of memory i.e we still need to implement a mechanism that  
will allow us to allocate memory in sizes smaller that 4kb. In other words, we want a granular memory allocator.

What we'll do is port an existing granular memory allocator to our OS. 

There are many existing open source memory allocators but we have chosen to go with liballoc here.
This is because:
* liballoc is quite popular i.e it has a good reputation and has stood the test of time. 
* liballoc is quite small - just two files - liballoc.h and liballoc.c

To port liballoc to our OS, we will need to implement some hooks. These hooks are functions that the liballoc library expects  
us to provide. Primarily, the hooks require an existing and working physical memory allocator.
So in short, 
1. we simply add the files to our project
2. modify the makefiles correspondingly 
3. implement the hooks
4. compile

## liballoc - Hooks
---

* The first two hooks are simply used to allocate and free physical pages.  
**Describe the functions**
* The next two hooks are used for mutual exclusion i.e there are some operations that liballoc does internally that  
must be executed in an atomic manner. So we need to provide a lock/unlock mechanism.
Since our OS is single threaded, the only source of concurrency is interrupts.  
So on the lock procedure, we disable interrupts by clearing the interrupt flag and on the unlock procedure,  
we enable interrupts by setting the interrupt flag.

## DEMO
---
Create a dynamic array, initialize and print to the screen. 
Then free the memory.

## THANK YOU
