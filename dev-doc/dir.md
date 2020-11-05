# Directory/File structure explanation

A neat diagrammatic representation of the directory structure can be computed by executing the following command in the project root:

```properties
tree .
```

## Description of directories

- **Kernel** : Contains the kernel code
  - **arch** : Contains code for each architecture Dax Os supports. Currently this is the [i386](https://en.wikipedia.org/wiki/Intel_80386) platform.
    - boot.S : This is the bootstrap assembly
    - crti.s :
    - crtn.s : These are required for using C++ (initializing global constructors).
    - linked.ld : Linker Script
    - tty.c : Contains code for terminal
    - vga.h : VGA definitions
  - **include** : Contains .h files for the implementation found in arch directory.
  - **kernel** :
    - kernel.c : Contains kernel entry point. (kernel_main)
