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

## Motivation (`Antony`)

- Writing low level code is hard and we wanted to do something **hard**.

  - _Illustrate the difficulty by mentioning the steps needed to setup a compiler_:
    1. First install 8 different dependencies.
    2. Download and **compile** binutils (for GNU linker and assembler).
    3. Download and **compile** GCC as a cross-compiler.
    4. Get a bootloader (eg: GRUB).
    5. Write custom linker script.
    6. Build a bootable image of your kernel.
    7. Now you have a clean slate. No printing, no keyboard, no mouse functionality.
  - Makes you appreciate the built-in abstractions.  
    Example:
    - We call a function and text magically appears on the screen
    - When the user presses a key, an event is triggered

- We want this project to be used in conjunction with Operating System classes (CS204).

  - Students end up having a good theoretical understanding but have no idea how to implement the concepts in code.
  - Students can examine the codebase to understand how the concepts are implemented.
  - This can encourage to students to contribute code to linux kernel and other open source projects.

- Additionally we want students to understand how to write clean code and design good system architectures.

- Students can also be introduced to basic concepts of devOps such as:
  - How to use source control and proper ettiquettes to follow when commiting and so on ...
  - How to do unit testing
  - How to write testable code
  - How to design workflows that shorten development cycles etc ...
