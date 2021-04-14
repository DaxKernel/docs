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