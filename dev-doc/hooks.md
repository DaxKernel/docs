# Why choose to port an existing malloc/free lib?

* malloc/free code is an integral part of the OS.  
By using a trusted and tested library we ensure that we do not introduce any hard to detect bugs.
* Software reuse is important. If possible always prefer to reuse already written software.

 
# Describe the hooks that need to be implemented

 Four types of hooks are implemented; two of which are used for page frame allocation and two that are used to handle concurrency. 

1. ```c
   void *alloc_page(size_t pages);
   ```
   Allocates pages number of page frames and returns a pointer to the beginning of the allocated pages. 

2. ```c
   void free_page(void *start, size_t pages);
   ```
   Frees pages number of page frames starting from the pointer start. 

3. ```c
   void wait_and_lock_mutex(); 
   ```
   Mutex lock mechanism. 

4. ```c
   void unlock_mutex();
   ```
   Mutex unlock mechanism. 

# How are the hooks actually implemented using the existing physical page allocator?

 The page size is obtained from the liballoc_init() function .

 1. `void* liballoc_alloc(size_t n_pages)` simply needs to call get_page() n_pages number of times and return the result of the first call.


2. `void free_page(void *start, size_t pages)` only needs to call free_page() with start, start + page_size, start + 2*page_size, ...  
, start+ pages * page_size as parameters.

3. As far as concurrency is concerned, we only need to ensure that no interrupts occur when `wait_and_lock_mutex()` is called.  
   This is easily accomplished by disabling interrupts by clearing the interrupt flag.  
   The corresponding `unlock_mutex()` call can then enable the interrupts by setting the interrupt flag.