# Running a C Program on Kali Linux from the Command Line

## 1. Install GCC (If Not Installed)

First, check if GCC (GNU Compiler Collection) is installed by running:

```bash
gcc --version
```

If GCC is not installed, install it with:

```bash
sudo apt update && sudo apt install gcc -y
```

---

## 2. Create a C File

Use any text editor to create a C file. For example, using **nano**:

```bash
nano hello.c
```

Then, add the following simple C program:

```c
#include <stdio.h>

int main() {
    printf("Hello, Kali Linux!\n");
    return 0;
}
```

Save the file with **CTRL + X**, then **Y**, and press **Enter**.

---

## 3. Compile the C Program

Run the following command to compile the program:

```bash
gcc hello.c -o hello
```

- `gcc` → Calls the compiler.
- `hello.c` → The C source file.
- `-o hello` → Output file named `hello` (the compiled binary).

---

## 4. Run the Compiled Program

Execute the compiled file with:

```bash
./hello
```

Expected output:

``` output
Hello, Kali Linux!
```

---

## 5. Debugging (Optional)

If you need to debug, use **gdb**:

```bash
gdb ./hello
```

Or compile with debugging symbols:

```bash
gcc -g hello.c -o hello
```

---

## 6. Running Without `./` (Optional)

To run the compiled file without `./`, move it to `/usr/local/bin`:

```bash
sudo mv hello /usr/local/bin/
```

Now you can run it with:

```bash
hello
```

## Cross-Compiling (Linux to Windows)

To compile a Windows PE executable on Linux, install mingw-w64:

```bash
sudo apt install mingw-w64 -y  # Debian/Ubuntu
```

```bash
sudo apt install gcc-multilib libc6-dev-i386 -y  # For 32-bit
```

- gcc-multilib → Enables 32-bit compilation support.
- libc6-dev-i386 → Installs 32-bit development libraries.

Then, compile the program with:

```bash
x86_64-w64-mingw32-gcc hello.c -o hello_x64.exe
```

Compile for 32-bit Windows:

```bash
i686-w64-mingw32-gcc hello.c -o hello_x86.exe
```

## check the file type of the compiled binary

```bash
file hello_x64.exe
```

Expected output:

``` output
hello_x64.exe: PE32+ executable (console) x86-64, for MS Windows
```

```bash
file hello_x86.exe
```

Expected output:

``` output
hello_x86.exe: PE32 executable (console) Intel 80386, for MS Windows
```



## Summary

| Target                  | Command                                         |
| :---------------------- | :---------------------------------------------- |
| Linux x86_64 (64-bit)   | gcc hello.c -o hello_x64                        |
| Linux x86 (32-bit)      | gcc -m32 hello.c -o hello_x86                   |
| Windows x86_64 (64-bit) | x86_64-w64-mingw32-gcc hello.c -o hello_x64.exe |
| Windows x86 (32-bit)    | i686-w64-mingw32-gcc hello.c -o hello_x86.exe   |
