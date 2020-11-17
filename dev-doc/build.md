## Understanding the build process

---

The following steps illustrate the build process:

1. Initially we call ./build.sh
2. ./build.sh calls ./header.sh
3. ./header.sh calls ./config.sh
4. config.sh shell script sets bash variables as follows:
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
5. Now ./header.sh proceeds
6. In the ./header.sh
   - sysroot folder is created if it does not exist
   - For each folder in SYSTEM_HEADER_PROJECTS as PROJECT, the following steps are executed:
     - PROJECT dir is opened
     - DESTDIR is set as ../../sysroot
     - make install-headers is called on the PROJECT directory
7. We return to ./build.sh, which proceeds as follows
8. In the ./build.sh
   - For each folder in PROJECTS as PROJECT, the following steps are executed:
     - PROJECT dir is opened
     - DESTDIR is set as ../../sysroot
     - make install is called on the PROJECT directory

## Understanding make install-headers command

---

1. installing headers

   - We have two directories kernel and libc
   - We use make to install our headers into sysroot/usr/include
   - for both the directories, the make install-header

     - Creates a directory sysroot/usr/include if it does not exist
     - And copies everything in include folder to sysroot/usr/include

## Understanding make install command

---

- The actual compilation is done by executing make install on each of the previously mentioned directories:
- This is a little more complicated than installing the headers.
- You must be familiar with both variables used and with make.

### Make install on /kernel directory

1. The make install command further calls two sub commands:

   - make install-headers
     - explained in step previous section
   - make install-kernel

     - install-kernel requires DaxOS.kernel as prerequisite
       - DaxOS.kernel requires OBJS and arch/i386/linker.ld as prerequisite
         - where OBJS is [crti.o crtbegin.o crtend.o ...kernel.o ...]
     - The .o files require .c or .S files with same name
     - These are available in the directory, so we compile .c files and .S files to corresponding .o files
     - With these .o files, make will produce DaxOS.kernel by linking them together with custom linker script
     - grub-mkrescue is used to confirm that multiboot is properly working

     - Now install-kernel performs the following:
       - create sysroot/boot directory if it does not exist
       - copy DaxOS.kernel to sysroot/boot directory

### Make install on /libc directory

1. Make install on /libc directory
2. This will call two sub-commands:
   - make install-headers
     - explained in previous sections
   - make install-libs
     - install-libs requires libk.a file as prerequisite.
     - .a files are archive files (like a zip or tar file). You can open .a files to find a collection .o output files.
     - libk.a requires the items in LIBK_OBJS as prerequisite. LIBK_OBJS contains the items in FREEOBJS but with the ending .o replaced by .libk.o.
     - The .libk.o files require .c/.S files as prerequisite which is available. Therefore the .c/.S files are compiled by GCC to .libk.o files.  
       Now libk.a can be produced by using GNU ar to combine all the .libk.o files into an archive.
     - Now install-libs can proceeds by:
       - create sysroot/usr/lib directory if it does not exist
       - copy the libk.a file to sysroot/usr/lib directory
