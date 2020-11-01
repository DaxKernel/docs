# Zeroth Presentation Speaker Notes

## Introduction (`Nihal`)

- We want to build a 32-bit operating system/ kernel
- We will use the C programming language and x86 assembly
  - The reasoning behind why we did not choose a modern programming language like Rust will be explained in the later slides.
- Our three basic goals are:

  ### 1. Implement Display

  - This involves printing text to the screen and drawing geometric shapes such as squares, triangles and so on ...
  - Drawing text to the screen
  - In our current design, we are using VGA text-mode and graphics-mode.  
    This involves writing 2 bytes to memory location _0xB8000_ to represent a character.

  ### 2. Input Drivers

  - This involves implementing both keyboard and mouse drivers from scratch.
  - Implementing keyboard driver involves reading scan codes from the PS/2 controller
  - Implementing mouse driver involves handling interrupts

  ### 3. C standard library

  - The functions that we take for granted - like strlen and printf, scanf, malloc are
    all parts of the C standard library.
  - They are not provided by the compiler.
  - It is upto the kernel to provide these libraries
  - We will implement some parts of the stdlib (like string.h and malloc function)

- Since it is not really practical, we will exclude things like networking.
