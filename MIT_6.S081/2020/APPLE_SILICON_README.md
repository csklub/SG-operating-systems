# Setting up xv6-labs-2020 Development Environment on Apple M1, M2, M3, M4 chips

Before starting, ensure you have:
- Git installed
- At least 10GB of free disk space
- Internet connection

## Required Packages

Install `homebrew` as the package manager.
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install dependencies required to build repositories:
```bash
brew install python3 gawk gnu-sed make gmp mpfr libmpc isl zlib expat texinfo flock libslirp glib gcc pkg-config pixman libmpc-devel libmpdclient capstone gnutls libpng libjpeg jpeg-turbo libusb libssh
```

### Create a working directory
```bash
mkdir -p ~/Developer/tools
```

Switch to working directory
```bash
cd ~/Developer/tools
```

### Install riscv-gnu-toolchain

We'll be building the Newlib cross compiler as instructed [here](https://github.com/riscv-collab/riscv-gnu-toolchain?tab=readme-ov-file#installation-newlib)

1. Create a writeable directory `/opt/riscv`
```bash
sudo mkdir /opt/riscv && sudo chmod -R 767 /opt/riscv
```

2. Create bin folder that will be home to the risc-v binaries
```bash
mkdir /opt/riscv/bin
```

3. Clone riscv-gnu-toolchain repository
```bash
git clone git@github.com:riscv-collab/riscv-gnu-toolchain.git
```

4. Switch to riscv-gnu-toolchain repository directory
```bash
cd riscv-gnu-toolchain
```

5. Add riscv-gnu-toolchain to your shell configuration `PATH`

Open your `.bashrc` or `.zshrc` file
```bash
vim ~/.zshrc
```

Append this line
```bash
export PATH="/opt/riscv/bin:$PATH"
```

6. Install Newlib
```bash
./configure --prefix=/opt/riscv --with-languages="c,c++"
make
```

7. Check that all the riscv-gnu-toolchain binaries were created.
```bash
ls /opt/riscv/bin
```

Expected output:
```
riscv64-unknown-elf-addr2line     riscv64-unknown-elf-gcc-nm        riscv64-unknown-elf-nm
riscv64-unknown-elf-ar            riscv64-unknown-elf-gcc-ranlib    riscv64-unknown-elf-objcopy
riscv64-unknown-elf-as            riscv64-unknown-elf-gcov          riscv64-unknown-elf-objdump
riscv64-unknown-elf-c++           riscv64-unknown-elf-gcov-dump     riscv64-unknown-elf-ranlib
riscv64-unknown-elf-c++filt       riscv64-unknown-elf-gcov-tool     riscv64-unknown-elf-readelf
riscv64-unknown-elf-cpp           riscv64-unknown-elf-gdb           riscv64-unknown-elf-run
riscv64-unknown-elf-elfedit       riscv64-unknown-elf-gdb-add-index riscv64-unknown-elf-size
riscv64-unknown-elf-g++           riscv64-unknown-elf-gprof         riscv64-unknown-elf-strings
riscv64-unknown-elf-gcc           riscv64-unknown-elf-ld            riscv64-unknown-elf-strip
riscv64-unknown-elf-gcc-14.2.0    riscv64-unknown-elf-ld.bfd
riscv64-unknown-elf-gcc-ar        riscv64-unknown-elf-lto-dump
```


### Install Qemu

1. Switch to the working directory created earlier
```bash
cd ~/Developer/tools
```

2. Clone the qemu repo
```bash
git clone git@github.com:qemu/qemu.git
```

3. Switch to qemu repository directory
```bash
cd qemu
```

4. Build qemu
```bash
mkdir build
cd build
../configure --target-list=riscv64-softmmu
make
```

5. Add the qemu-system-riscv64 binary to your local binaries
```bash
make install
```

Confirm it was added
```bash
which qemu-system-riscv64
```

Output: `/usr/local/bin/qemu-system-riscv64`


## Setup Steps for Lab

1. **Clone the Repository**
   ```bash
   git clone git://g.csail.mit.edu/xv6-labs-2020
   cd xv6-labs-2020
   ```

2. **Switch to the Initial Lab Branch (util)**
   ```bash
   git checkout util
   ```

   **!Important**:
   Add the file changes in this [PR](https://github.com/mit-pdos/xv6-riscv/pull/62) to the respective files on your util branch

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
   riscv64-unknown-elf-gdb kernel/kernel
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
