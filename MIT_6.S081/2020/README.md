# Setting up xv6-labs-2020 Development Environment

## Prerequisites

Before starting, ensure you have:
- Git installed
- A Unix-like environment (Linux, macOS, or WSL for Windows)
- At least 1GB of free disk space
- Internet connection

## Required Packages

### For Ubuntu/Debian:
```bash
sudo apt-get update
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

### For macOS (using Homebrew):
```bash
brew tap riscv/riscv
brew install riscv-tools
brew install qemu
```

## Setup Steps

1. **Clone the Repository**
   ```bash
   git clone git://g.csail.mit.edu/xv6-labs-2020
   cd xv6-labs-2020
   ```

2. **Switch to the Initial Lab Branch (util)**
   ```bash
   git checkout util
   ```

3. **Test the Installation**
   ```bash
   make qemu
   ```

   This command will:
   - Compile the xv6 operating system
   - Start QEMU emulator
   - Boot xv6

   If successful, you should see the xv6 boot sequence and a prompt that looks like this:
   ```
   xv6 kernel is booting

   hart 1 starting
   hart 2 starting
   init: starting sh
   $
   ```

4. **Exit QEMU**
   - Press `Ctrl-a` then `x` to exit QEMU

  ```bash
  console        3 21 0
  $ QEMU: Terminated
  (base) ➜  xv6-labs-2020 git:(util) ✗
  ```

## Development Tips

1. **Using GDB for Debugging**
   In one terminal:
   ```bash
   make qemu-gdb
   ```
   In another terminal:
   ```bash
   gdb kernel/kernel
   ```

2. **Useful Make Commands**
   ```bash
   make clean     # Clean build files
   make           # Build xv6
   make qemu      # Run xv6 in QEMU
   make grade     # Test your implementation (when working on labs)
   ```

3. **File Organization**
   - `kernel/`: Kernel source files
   - `user/`: User programs
   - `Makefile`: Build configuration

## Working on Labs

1. Each lab has its own branch. To switch to a different lab:
   ```bash
   git fetch
   git checkout [lab-name]
   ```

2. Common lab branches:
   - util
   - syscall
   - pgtbl
   - traps
   - lazy
   - cow
   - thread
   - lock
   - fs
   - mmap

3. Before starting each lab, make sure to:
   ```bash
   git checkout [lab-name]
   make clean
   ```

## Submitting Changes

1. Make your changes in the appropriate files
2. Test your implementation:
   ```bash
   make grade
   ```
3. Commit your changes:
   ```bash
   git add [modified-files]
   git commit -m "Descriptive message about changes"
   ```

## Additional Resources

- [MIT 6.S081 Course Website](https://pdos.csail.mit.edu/6.828/2020/schedule.html)
- [xv6 Book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)
- [RISC-V Reference](https://github.com/riscv/riscv-isa-manual)
