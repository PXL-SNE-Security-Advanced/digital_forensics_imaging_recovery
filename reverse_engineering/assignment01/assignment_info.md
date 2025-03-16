# Assignment 1: Reverse Engineering - Decompiling a C Program

In this assignment, you will decompile simple C programs to understand how it works.

## references

- [Ghidra](https://ghidra-sre.org/)
- [Task 11 - A Christmas DOScovery: Tapes of Yule-tide Past](https://tryhackme.com/room/adventofcyber2023)
- [Task 12 - Memory corruption](https://tryhackme.com/room/adventofcyber2023)

## Instructions

**Decompile the C Program**:
    - Use the provided C program and decompile it to understand the source code.
    - The C programs are available in the exercise files.
    - Use Ghidra to decompile the C program.
    - Describe the functionality of the C programs in your own words.
    - Copy the decompiled source code, describe the functionality.

## Exercise Files

The exercise files are available in the exercise_files folder.

### Oef1.exe

### Oef2.exe

### Oef3.exe

### Oef4.exe

### Oefening - age.c

Compile the following C code and decompile it to better understand how the decompilation works.

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    char name[50];
    int birth_year, birth_month, birth_day;
    time_t current_time = time(NULL);
    struct tm *tm = localtime(&current_time);
    int current_year = tm->tm_year + 1900;
    int current_month = tm->tm_mon + 1;
    int current_day = tm->tm_mday;

    printf("What is your name?\n");
    scanf("%s", name);

    printf("What year were you born?\n");
    scanf("%d", &birth_year);

    printf("What month were you born (1-12)?\n");
    scanf("%d", &birth_month);

    printf("What day of the month were you born?\n");
    scanf("%d", &birth_day);

    int age = current_year - birth_year;
    if (current_month < birth_month || (current_month == birth_month && current_day < birth_day)) {
        age--;
    }
```

