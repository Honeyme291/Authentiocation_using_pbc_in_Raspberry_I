# Pairing-Based Cryptography (PBC) Installation and Cross-Compilation Guide

This guide explains how to install the Pairing-Based Cryptography (PBC) library on a Linux system and how to cross-compile it for embedded ARM devices such as microcontrollers.

---

##  1. Environment Setup

### 1.1 Update System and Install Dependencies
```bash
sudo apt-get update
sudo apt-get install build-essential libgmp-dev m4
```
> 💡 `libgmp-dev` is a required dependency for PBC, providing support for large integer arithmetic.

---

## 2. Installing PBC (x86 Environment)

### 2.1 Download and Extract PBC
```bash
wget http://crypto.stanford.edu/pbc/files/pbc-0.5.14.tar.gz
tar -xvf pbc-0.5.14.tar.gz
cd pbc-0.5.14
```

### 2.2 Configure, Build, and Install
```bash
./configure
make
sudo make install
```

### 2.3 Verify Installation
```bash
gcc -o test test.c -I /usr/local/include/pbc -L /usr/local/lib -lpbc -lgmp
./test ./param/a.param
```
If the program runs successfully, PBC is correctly installed.

---

##  3. Configure ARM Cross-Compilation Environment

> Default ARM toolchain:  
> `gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf`

### 3.1 Download and Extract ARM GCC
```bash
cd /usr/local
sudo wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz
sudo tar -xf gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz
```

### 3.2 Add to Environment Variables
```bash
sudo vi /etc/profile
```
Add the following line at the end of the file:
```bash
export PATH=$PATH:/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/bin
```
Apply changes:
```bash
source /etc/profile
reboot
```

### 3.3 Verify Installation
```bash
arm-none-linux-gnueabihf-gcc -v
```

---

## 4. Cross-Compile GMP Library (for ARM)

### 4.1 Download and Extract
```bash
wget https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz
tar -xf gmp-6.2.1.tar.xz
cd gmp-6.2.1
```

### 4.2 Configure and Build
```bash
./configure --host=arm-none-linux-gnueabihf   --enable-cxx   --prefix=/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf
make
sudo make install
```

---

## 5. Cross-Compile PBC Library (for ARM)

### 5.1 Download and Extract
```bash
wget https://crypto.stanford.edu/pbc/files/pbc-0.5.14.tar.gz
tar -xvf pbc-0.5.14.tar.gz
cd pbc-0.5.14
```

### 5.2 Configure and Build
```bash
./configure --host=arm-none-linux-gnueabihf   --prefix=/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/pbc   CC=arm-none-linux-gnueabihf-gcc   LDFLAGS="-L/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/lib"   CPPFLAGS="-I/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/include"
make
sudo make install
```

---

## 6. Compile C Code for ARM Platform

Assume you have a source file named `standard.c`:
```bash
arm-none-linux-gnueabihf-gcc -o standard standard.c   -I/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/include   -L/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/lib -lgmp   -I/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/pbc/include   -L/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/pbc/lib -lpbc
```
This will generate an ARM-compatible executable file named `standard`.

---

## 7. Run on ARM Device (Microcontroller)

1. **Transfer Executable and Libraries**
   ```bash
   scp standard root@192.168.7.1:/home/root/
   scp -r /usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/pbc root@192.168.7.1:/home/root/
   ```

2. **Connect via SSH**
   ```bash
   ssh -o HostKeyAlgorithms=+ssh-rsa,ssh-dss root@192.168.7.1
   ```

3. **Set Library Path**
   ```bash
   export LD_LIBRARY_PATH=/home/root/pbc/lib:$LD_LIBRARY_PATH
   ```

4. **Run the Program**
   ```bash
   ./standard param/a.param
   ```

If the program runs successfully, PBC is working properly on the ARM device.

---

## 8. Troubleshooting (FAQ)

| Issue | Possible Cause | Solution |
|--------|----------------|-----------|
| `error while loading shared libraries: libpbc.so` | Missing runtime library | Set the `LD_LIBRARY_PATH` environment variable |
| `configure: error: GMP library not found` | GMP not properly installed or wrong path | Check `--prefix`, `CPPFLAGS`, and `LDFLAGS` settings |
| Compilation error: `pbc.h not found` | Missing include path | Add `-I` flag to point to the PBC include directory |

---

## 9. References
- [PBC Official Website](http://crypto.stanford.edu/pbc)
- [GMP Official Website](https://gmplib.org)
- [ARM GNU Toolchain](https://developer.arm.com)

---
