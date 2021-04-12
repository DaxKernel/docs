# Why choose to port an existing malloc/free lib?

---

 If a memory allocator is created we will have to check for errors and debug if any are found .
 A much simpler way would be porting an existing malloc library to the kernel space in the os .
 We can focus on creating the os rather than spending a dedicated amount of time creating a memory allocator and as it is a well tested library we wouldn’t have to debug it. 
 Hence porting is fast, scalable, stable and requires less time to implement. 
 
# Describe the hooks that need to be implemented

---

 Hooks allow memory managers to request memory in terms of pages from the kernel.
 The page size is a constant of 4096 bytes which is obtained from f_size header file. 
 Number of pages required is the parameter accepted by hook. 
 Basically four types of hooks are implemented, two of which are used for page allocation and deallocation and two used for critial section.
 Critical section is a piece of code that is implemented here so that the completion of memory operations takes place without interruption of other process.

1. void *alloc_page(size_t pages); 
   Takes page size as input parameters , allocates consecutive pages  in virtual memory and returns a pointer pointing to the beginning of the group .

2. void free_page(void *start, size_t pages);
   no of pages to be free and the starting location in memory as input parameters and frees allocated pages in virtual memory.

3. void wait_and_lock_mutex() ; 
   locks the mutex before memory manager executes a memory process of a singe thread entering the critical section .
   We can apply mutex lock simply by disabling all the interrupts or by using spin lock.
   
4. void unlock_mutex();
   Mutex is unlocked after memory manager leaves the critical section .
   Unlocking can be done by re-enabling the interrupts or resetting the spin lock.

# How are the hooks actually implemented using the existing physical page allocator?

---

 The page size is obtained from the liballoc_init() function .

 1. For allocation of page we are using : void* liballoc_alloc(size_t n_pages) .
  The number of pages required for allocation is accepted as a input parameter and if the required number of pages is not available while considering all the number of unallocated pages then the function returns a NULL value. 
  If the required number of pages are available then memory allocation stats from pointer *start  .
  Consecutive pages is allocated till the number of required pages is satisfied .The pointer of the allocated memory is returned.

2.  For deallocation of pages we will be using : int liballoc_free(void* start, size_t n_pages) . 
  The values returned from void* liballoc_alloc(size_t n_pages)  is passed as input parameters into this hook. 
  Here we set a starting point to start deallocation of memory .Consecutive pages are being deallocated and added to the free page. 
  The number of pages to be deallocated should be given in the formal parameter .
  0 is returned if the deallocation of pages are successful.

 3. For locking of memory data structure we are using : int liballoc_lock() .
  Interrupts are disabled so that other important functions can only be done after memory operation.
  Interrupts are cleared using asm("cli"); and if the lock acquired is successful ,0 is returned .

 4. For unlocking of memory data structure we are using : int liballoc_unlock() . 
  Setting back the interrupts are done using asm("sti") command.
  Return 0 if the lock is released successfully.