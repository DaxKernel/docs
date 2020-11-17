The following steps illustrate the build process:

1. Initially we call ./build.sh
2. ./build.sh calls for ./header.sh
3. ./header.sh package calls ./config.sh
4. This package sets bash variables as specific values as follows
   - MAKE = make
   - HOST = i686-elf
   - AR = i686-elf-ar
   - AS =i686-elf-as
   - CC=i686-elf-gcc
   - BOOTDIR = /boot
   - LIBDIR = /usr/lib
   - eFLAGS = - 02 - g
   - CPPFLAGS = 1/
   - SYSROOT = dev/DaxOS/sysroot
   - CC = i686-elf-gcc "sysroot=SYSROOT"
   - SYSTEM_HEADER_PROJECTS ="libc kernel"
   - PROJECTS = "libc kernel"
5. The bash variables are returned to ./header.sh
6. In the ./header.sh
   - sysroot folder is being created
   - SYSTEM_HEADER_PROJECTS is repeatedly excecuted
     - PROJECT dir is opened
     - DESTDIR is set as ../../sysroot
     - make install-headers is called
7. system control is returned back to ./build.sh
8. In the ./build.sh
   - PROJECTS is being iterated
     - PROJECT dir is opened
     - DESTDIR is set as ../../sysroot
     - make install is being called
9. installing headers

   - We have two directories kernel and libc
   - We use make to install our headers into sysroot/usr/include
   - for both the directories ,the make install-header

     - Creates a directory sysroot/usr/include if it does not exist
     - And a copy everything is passed from include folder to sysroot/usr/include

   - The actual compilation is done by executing make install on each of the previously mentioned directories:
   - This is a little more complicated than installing the headers.
   - You must be familiar with both variables used and with make.

10. Make install on /kernel directory
11. The make install command further calls two sub commands:

    - make install-headers
      - explained in step 9
    - make install-kernel

      - install-kernel requires DaxOS.kernel to proceed -->
        - DaxOS.kernel requires OBJS and arch/i386/linker.ld to proceed -->
          - where OBJS is [crti.o crtbegin.o crtend.o ...kernel.o ...]
      - the .o files require .c or .S files with same name
      - These are available in the directory , so compile .c files and .S files to .o files
      - With these .o files, make will produce DaxOS.kernel by linking them together
      - then it is confirmed that multiboot is properly working

      - Now install-kernel performs the following by:
        - create sysroot/boot directory if it does not exist
        - copy DaxOS.kernel to sysroot/boot directory

12. Make install on /libc directory
13. This will call two sub-commands:
    - make install-headers
      - explained in step 9
    - make install-libs
      - install-libs require libk.a file as prerequisite.
      - .a files are archive files (like a zip or tar file). In fact you can open .a files to find a collection .c implementation files.
      - libk.a requires the items in LIBK_OBJS as prerequisite. LIBK_OBJS contains the items in FREEOBJS but with the ending .o replaced by .libk.o.
      - The .libk.o files require .c/.S files as prerequisite which is available. Therefore the .c/.S files are compiled by GCC to .libk.o files. Now libk.a can be produced by using GNU ar to combine all the .libk.o files into an archive.
      - Now install-libs can proceeds by:
        - create sysroot/usr/lib directory if it does not exist
        - copy the libk.a file to sysroot/usr/lib directory
