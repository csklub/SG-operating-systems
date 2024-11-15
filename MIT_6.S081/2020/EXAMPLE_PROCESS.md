# XV6 Process Creation Tutorial

Assuming you have successfully installed xv6 and you when you run `make qemu`, you get to this console output:
```
xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$
```

This tutorial demonstrates how to create and run a simple process example from Chapter 1 of the MIT XV6 RISC-V [Book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf).

We'll create a parent-child process relationship and explore basic process management in XV6.

## Prerequisites

- XV6 operating system source code
- RISC-V toolchain installed
- Basic understanding of C programming

## Creating the Example

### 1. File Setup

Create a new file `example_process.c` in the user directory of your xv6-labs-2020 repo:

```bash
touch user/example_process.c
```

### 2. Source Code

Add the following code to `example_process.c`:

```c
// Copyright 2024 <your-name>
// SPDX-License-Identifier: MIT

#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

int
main(int argc, char *argv[])
{
    int pid;
    pid = fork();
    if (pid > 0) {
        printf("parent: child = %d\n", pid);
        pid = wait((int *) 0);
        printf("child %d is done\n", pid);
    } else if (pid == 0) {
        printf("child: exiting\n");
        exit(0);
    } else {
        printf("fork error\n");
    }
    exit(0);  // Ensure parent exits
}
```

### 3. Code Explanation

Let's break down the key components:

1. **Header Files**:
   - `kernel/types.h`: Basic type definitions
   - `kernel/stat.h`: File status structures
   - `user/user.h`: User-space system calls
   - `kernel/fs.h`: File system structures

2. **Process Creation**:
   - `fork()`: Creates a new process by duplicating the calling process
   - Returns:
     - To parent: Returns child's PID (positive number)
     - To child: Returns 0
     - Error: Returns -1

3. **Process Flow**:
   - Parent process:
     - Prints child's PID
     - Waits for child to finish using `wait()`
     - Prints completion message
   - Child process:
     - Prints exit message
     - Exits immediately

### 4. Adding to XV6 Build

Update `Makefile` to include your new program:

1. Find the `UPROGS` section
2. Append the following line at the end:
```makefile
$U/_example_process\
```

#### Detailed explanation of UPROGS
  1. UPROGS Definition:
  ```makefile
  UPROGS=\
      $U/_cat\
      $U/_echo\
      $U/_forktest\
      # ... more programs
  ```
  This is a list of user programs that will be compiled into the xv6 operating system. Each entry prefixed with $U/ (where U=user) represents a program in the user directory, and the underscore (_) prefix is a convention used in xv6.

  2. Compilation Rule:
  The key pattern rule for compiling these programs is:
  ```makefile
  _%: %.o $(ULIB)
      $(LD) $(LDFLAGS) -N -e main -Ttext 0 -o $@ $^
      $(OBJDUMP) -S $@ > $*.asm
      $(OBJDUMP) -t $@ | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $*.sym
  ```

  3. Compilation Process:
  - For each program listed in UPROGS (e.g., _cat):
    1. The corresponding .c file (e.g., cat.c) is first compiled into an object file (.o)
    2. The object file is linked with ULIB (user library files):
      ```makefile
      ULIB = $U/ulib.o $U/usys.o $U/printf.o $U/umalloc.o
      ```
    3. The linker creates the final executable
    4. Assembly (.asm) and symbol table (.sym) files are generated for debugging

  4. Integration:
  - These compiled programs are then included in the filesystem image (fs.img) through:
  ```makefile
  fs.img: mkfs/mkfs README $(UEXTRA) $(UPROGS)
      mkfs/mkfs fs.img README $(UEXTRA) $(UPROGS)
  ```

  For example, to compile _cat:
  1. cat.c → cat.o
  2. cat.o is linked with ULIB to create _cat
  3. _cat is included in fs.img
  4. This process repeats for each program in UPROGS

  The result is that all these user programs become available in the xv6 filesystem when the OS boots.

### 5. Building and Running

1. Build XV6:
```bash
make clean
make
```

2. Run XV6:
```bash
make qemu
```

3. In the XV6 shell, run:
```bash
example_process
```

### Expected Output

```
parent: child = 2
child: exiting
child 2 is done
```

## Understanding the Output

1. The parent process creates a child using `fork()`
2. Parent receives child's PID and prints it
3. Child process runs, prints its message, and exits
4. Parent's `wait()` returns with child's PID
5. Parent prints completion message and exits
